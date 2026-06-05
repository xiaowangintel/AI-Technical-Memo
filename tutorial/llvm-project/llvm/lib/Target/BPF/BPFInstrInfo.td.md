# BPFInstrInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFInstrInfo.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file describes the BPF instructions in TableGen format.
- 目的（中文）: 使用 TableGen 定义目标指令，描述操作码、操作数、匹配模式与调度信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- BPFInstrInfo.td - Target Description for BPF Target ---------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the BPF instructions in TableGen format.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: include "BPFInstrFormats.td"
  14: 
  15: // Instruction Operands and Patterns
  16: 
  17: // These are target-independent nodes, but have target-specific formats.
  18: def SDT_BPFCallSeqStart : SDCallSeqStart<[SDTCisVT<0, iPTR>,
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions. This range defines declarative TableGen records such as SDT_BPFCallSeqStart, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。 这一段定义了 SDT_BPFCallSeqStart 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 19-36

```tablegen
  19:                                           SDTCisVT<1, iPTR>]>;
  20: def SDT_BPFCallSeqEnd   : SDCallSeqEnd<[SDTCisVT<0, iPTR>, SDTCisVT<1, iPTR>]>;
  21: def SDT_BPFCall         : SDTypeProfile<0, -1, [SDTCisVT<0, iPTR>]>;
  22: def SDT_BPFSetFlag      : SDTypeProfile<0, 3, [SDTCisSameAs<0, 1>]>;
  23: def SDT_BPFSelectCC     : SDTypeProfile<1, 5, [SDTCisSameAs<1, 2>,
  24:                                                SDTCisSameAs<0, 4>,
  25:                                                SDTCisSameAs<4, 5>]>;
  26: def SDT_BPFBrCC         : SDTypeProfile<0, 4, [SDTCisSameAs<0, 1>,
  27:                                                SDTCisVT<3, OtherVT>]>;
  28: def SDT_BPFWrapper      : SDTypeProfile<1, 1, [SDTCisSameAs<0, 1>,
  29:                                                SDTCisPtrTy<0>]>;
  30: def SDT_BPFMEMCPY       : SDTypeProfile<0, 4, [SDTCisVT<0, i64>,
  31:                                                SDTCisVT<1, i64>,
  32:                                                SDTCisVT<2, i64>,
  33:                                                SDTCisVT<3, i64>]>;
  34: 
  35: def BPFcall         : SDNode<"BPFISD::CALL", SDT_BPFCall,
  36:                              [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
```

- EN: This range defines declarative TableGen records such as SDT_BPFCallSeqEnd, SDT_BPFCall, SDT_BPFSetFlag, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SDT_BPFCallSeqEnd, SDT_BPFCall, SDT_BPFSetFlag 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-54

```tablegen
  37:                               SDNPVariadic]>;
  38: def BPFretglue      : SDNode<"BPFISD::RET_GLUE", SDTNone,
  39:                              [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
  40: def BPFcallseq_start: SDNode<"ISD::CALLSEQ_START", SDT_BPFCallSeqStart,
  41:                              [SDNPHasChain, SDNPOutGlue]>;
  42: def BPFcallseq_end  : SDNode<"ISD::CALLSEQ_END",   SDT_BPFCallSeqEnd,
  43:                              [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
  44: def BPFbrcc         : SDNode<"BPFISD::BR_CC", SDT_BPFBrCC, [SDNPHasChain]>;
  45: 
  46: def SDT_BPFLoadStackArg : SDTypeProfile<1, 1, [
  47:   SDTCisVT<0, i64>,  // result value
  48:   SDTCisVT<1, i64>   // operand: offset
  49: ]>;
  50: def SDT_BPFStoreStackArg : SDTypeProfile<0, 2, [
  51:   SDTCisVT<0, i64>,  // operand 0: offset
  52:   SDTCisVT<1, i64>   // operand 1: stored value
  53: ]>;
  54: 
```

- EN: This range defines declarative TableGen records such as BPFretglue, BPFcallseq_start, BPFcallseq_end, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 BPFretglue, BPFcallseq_start, BPFcallseq_end 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55: def BPFload_stack_arg : SDNode<"BPFISD::LOAD_STACK_ARG",
  56:                                SDT_BPFLoadStackArg,
  57:                                [SDNPHasChain, SDNPMayLoad]>;
  58: def BPFstore_stack_arg : SDNode<"BPFISD::STORE_STACK_ARG",
  59:                                 SDT_BPFStoreStackArg,
  60:                                 [SDNPHasChain, SDNPMayStore]>;
  61: 
  62: def BPFselectcc     : SDNode<"BPFISD::SELECT_CC", SDT_BPFSelectCC>;
  63: def BPFWrapper      : SDNode<"BPFISD::Wrapper", SDT_BPFWrapper>;
  64: def BPFmemcpy       : SDNode<"BPFISD::MEMCPY", SDT_BPFMEMCPY,
  65:                              [SDNPHasChain, SDNPMayStore, SDNPMayLoad]>;
  66: def BPFIsLittleEndian : Predicate<"Subtarget->isLittleEndian()">;
  67: def BPFIsBigEndian    : Predicate<"!Subtarget->isLittleEndian()">;
  68: def BPFHasALU32 : Predicate<"Subtarget->getHasAlu32()">;
  69: def BPFNoALU32 : Predicate<"!Subtarget->getHasAlu32()">;
  70: def BPFHasLdsx : Predicate<"Subtarget->hasLdsx()">;
  71: def BPFHasMovsx : Predicate<"Subtarget->hasMovsx()">;
  72: def BPFHasBswap : Predicate<"Subtarget->hasBswap()">;
```

- EN: This range defines declarative TableGen records such as BPFload_stack_arg, BPFstore_stack_arg, BPFselectcc, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 BPFload_stack_arg, BPFstore_stack_arg, BPFselectcc 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-90

```tablegen
  73: def BPFHasSdivSmod : Predicate<"Subtarget->hasSdivSmod()">;
  74: def BPFNoMovsx : Predicate<"!Subtarget->hasMovsx()">;
  75: def BPFNoBswap : Predicate<"!Subtarget->hasBswap()">;
  76: def BPFHasStoreImm : Predicate<"Subtarget->hasStoreImm()">;
  77: def BPFHasLoadAcqStoreRel : Predicate<"Subtarget->hasLoadAcqStoreRel()">;
  78: def BPFHasGotox : Predicate<"Subtarget->hasGotox()">;
  79: 
  80: class ImmediateAsmOperand<string name> : AsmOperandClass {
  81:   let Name = name;
  82:   let RenderMethod = "addImmOperands";
  83:   let DiagnosticType = !strconcat("Invalid", name);
  84: }
  85: 
  86: def SImm16AsmOperand : ImmediateAsmOperand<"SImm16">;
  87: 
  88: def brtarget : Operand<OtherVT> {
  89:   let PrintMethod = "printBrTargetOperand";
  90:   let ParserMatchClass = ImmediateAsmOperand<"BrTarget">;
```

- EN: This range defines declarative TableGen records such as BPFHasSdivSmod, BPFNoMovsx, BPFNoBswap, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BPFHasSdivSmod, BPFNoMovsx, BPFNoBswap 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91: }
  92: def calltarget : Operand<i64>;
  93: 
  94: def u64imm   : Operand<i64> {
  95:   let PrintMethod = "printImm64Operand";
  96: }
  97: 
  98: def s16imm : Operand<i16> {
  99:   let ParserMatchClass = SImm16AsmOperand;
 100: }
 101: 
 102: def gpr_or_imm : Operand<i64>;
 103: 
 104: def i64immSExt32 : PatLeaf<(i64 imm),
 105:                 [{return isInt<32>(N->getSExtValue()); }]>;
 106: def i32immSExt32 : PatLeaf<(i32 imm),
 107:                 [{return isInt<32>(N->getSExtValue()); }]>;
 108: def i64immZExt32 : PatLeaf<(i64 imm),
```

- EN: This range defines declarative TableGen records such as calltarget, u64imm, s16imm, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 calltarget, u64imm, s16imm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109:                 [{return isUInt<32>(N->getZExtValue()); }]>;
 110: 
 111: def imm_to_i64 : SDNodeXForm<timm, [{
 112:   return CurDAG->getTargetConstant(N->getZExtValue(), SDLoc(N), MVT::i64);
 113: }]>;
 114: 
 115: // Addressing modes.
 116: def ADDRri : ComplexPattern<i64, 2, "SelectAddr", [], []>;
 117: def FIri : ComplexPattern<i64, 2, "SelectFIAddr", [add, or], []>;
 118: 
 119: // Address operands
 120: def MEMri : Operand<i64> {
 121:   let PrintMethod = "printMemOperand";
 122:   let EncoderMethod = "getMemoryOpValue";
 123:   let DecoderMethod = "decodeMemoryOpValue";
 124:   let MIOperandInfo = (ops GPR, s16imm);
 125: }
 126: 
```

- EN: This range defines declarative TableGen records such as imm_to_i64, ADDRri, FIri, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 imm_to_i64, ADDRri, FIri 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 127-144

```tablegen
 127: // Conditional code predicates - used for pattern matching for jump instructions
 128: def BPF_CC_EQ  : PatLeaf<(i64 imm),
 129:                          [{return (N->getZExtValue() == ISD::SETEQ);}]>;
 130: def BPF_CC_NE  : PatLeaf<(i64 imm),
 131:                          [{return (N->getZExtValue() == ISD::SETNE);}]>;
 132: def BPF_CC_GE  : PatLeaf<(i64 imm),
 133:                          [{return (N->getZExtValue() == ISD::SETGE);}]>;
 134: def BPF_CC_GT  : PatLeaf<(i64 imm),
 135:                          [{return (N->getZExtValue() == ISD::SETGT);}]>;
 136: def BPF_CC_GTU : PatLeaf<(i64 imm),
 137:                          [{return (N->getZExtValue() == ISD::SETUGT);}]>;
 138: def BPF_CC_GEU : PatLeaf<(i64 imm),
 139:                          [{return (N->getZExtValue() == ISD::SETUGE);}]>;
 140: def BPF_CC_LE  : PatLeaf<(i64 imm),
 141:                          [{return (N->getZExtValue() == ISD::SETLE);}]>;
 142: def BPF_CC_LT  : PatLeaf<(i64 imm),
 143:                          [{return (N->getZExtValue() == ISD::SETLT);}]>;
 144: def BPF_CC_LTU : PatLeaf<(i64 imm),
```

- EN: This range defines declarative TableGen records such as BPF_CC_EQ, BPF_CC_NE, BPF_CC_GE, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 BPF_CC_EQ, BPF_CC_NE, BPF_CC_GE 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 145-162

```tablegen
 145:                          [{return (N->getZExtValue() == ISD::SETULT);}]>;
 146: def BPF_CC_LEU : PatLeaf<(i64 imm),
 147:                          [{return (N->getZExtValue() == ISD::SETULE);}]>;
 148: def BPF_CC_EQ_32  : PatLeaf<(i32 imm),
 149:                          [{return (N->getZExtValue() == ISD::SETEQ);}]>;
 150: def BPF_CC_NE_32  : PatLeaf<(i32 imm),
 151:                          [{return (N->getZExtValue() == ISD::SETNE);}]>;
 152: def BPF_CC_GE_32  : PatLeaf<(i32 imm),
 153:                          [{return (N->getZExtValue() == ISD::SETGE);}]>;
 154: def BPF_CC_GT_32  : PatLeaf<(i32 imm),
 155:                          [{return (N->getZExtValue() == ISD::SETGT);}]>;
 156: def BPF_CC_GTU_32 : PatLeaf<(i32 imm),
 157:                          [{return (N->getZExtValue() == ISD::SETUGT);}]>;
 158: def BPF_CC_GEU_32 : PatLeaf<(i32 imm),
 159:                          [{return (N->getZExtValue() == ISD::SETUGE);}]>;
 160: def BPF_CC_LE_32  : PatLeaf<(i32 imm),
 161:                          [{return (N->getZExtValue() == ISD::SETLE);}]>;
 162: def BPF_CC_LT_32  : PatLeaf<(i32 imm),
```

- EN: This range defines declarative TableGen records such as BPF_CC_LEU, BPF_CC_EQ_32, BPF_CC_NE_32, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 BPF_CC_LEU, BPF_CC_EQ_32, BPF_CC_NE_32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 163-180

```tablegen
 163:                          [{return (N->getZExtValue() == ISD::SETLT);}]>;
 164: def BPF_CC_LTU_32 : PatLeaf<(i32 imm),
 165:                          [{return (N->getZExtValue() == ISD::SETULT);}]>;
 166: def BPF_CC_LEU_32 : PatLeaf<(i32 imm),
 167:                          [{return (N->getZExtValue() == ISD::SETULE);}]>;
 168: def NoCond : PatLeaf<(vt)> {}
 169: 
 170: // For arithmetic and jump instructions the 8-bit 'code'
 171: // field is divided into three parts:
 172: //
 173: //  +----------------+--------+--------------------+
 174: //  |   4 bits       |  1 bit |   3 bits           |
 175: //  | operation code | source | instruction class  |
 176: //  +----------------+--------+--------------------+
 177: //  (MSB)                                      (LSB)
 178: class TYPE_ALU_JMP<bits<4> op, bits<1> srctype,
 179:                    dag outs, dag ins, string asmstr, list<dag> pattern>
 180:   : InstBPF<outs, ins, asmstr, pattern> {
```

- EN: This range defines declarative TableGen records such as BPF_CC_LTU_32, BPF_CC_LEU_32, NoCond, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 BPF_CC_LTU_32, BPF_CC_LEU_32, NoCond 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 181-198

```tablegen
 181: 
 182:   let Inst{63-60} = op;
 183:   let Inst{59} = srctype;
 184: }
 185: 
 186: //For load and store instructions the 8-bit 'code' field is divided as:
 187: //
 188: //  +--------+--------+-------------------+
 189: //  | 3 bits | 2 bits |   3 bits          |
 190: //  |  mode  |  size  | instruction class |
 191: //  +--------+--------+-------------------+
 192: //  (MSB)                             (LSB)
 193: class TYPE_LD_ST<bits<3> mode, bits<2> size,
 194:                  dag outs, dag ins, string asmstr, list<dag> pattern>
 195:   : InstBPF<outs, ins, asmstr, pattern> {
 196: 
 197:   let Inst{63-61} = mode;
 198:   let Inst{60-59} = size;
```

- EN: This range defines declarative TableGen records such as TYPE_LD_ST, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 TYPE_LD_ST 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-216

```tablegen
 199: }
 200: 
 201: // jump instructions
 202: class JMP_RR<BPFJumpOp Opc, string OpcodeStr, PatLeaf Cond>
 203:     : TYPE_ALU_JMP<Opc.Value, BPF_X.Value,
 204:                    (outs),
 205:                    (ins GPR:$dst, GPR:$src, brtarget:$BrDst),
 206:                    "if $dst "#OpcodeStr#" $src goto $BrDst",
 207:                    [(BPFbrcc i64:$dst, i64:$src, Cond, bb:$BrDst)]> {
 208:   bits<4> dst;
 209:   bits<4> src;
 210:   bits<16> BrDst;
 211: 
 212:   let Inst{55-52} = src;
 213:   let Inst{51-48} = dst;
 214:   let Inst{47-32} = BrDst;
 215:   let BPFClass = BPF_JMP;
 216: }
```

- EN: This range defines declarative TableGen records such as JMP_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 JMP_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217: 
 218: class JMP_RI<BPFJumpOp Opc, string OpcodeStr, PatLeaf Cond>
 219:     : TYPE_ALU_JMP<Opc.Value, BPF_K.Value,
 220:                    (outs),
 221:                    (ins GPR:$dst, i64imm:$imm, brtarget:$BrDst),
 222:                    "if $dst "#OpcodeStr#" $imm goto $BrDst",
 223:                    [(BPFbrcc i64:$dst, i64immSExt32:$imm, Cond, bb:$BrDst)]> {
 224:   bits<4> dst;
 225:   bits<16> BrDst;
 226:   bits<32> imm;
 227: 
 228:   let Inst{51-48} = dst;
 229:   let Inst{47-32} = BrDst;
 230:   let Inst{31-0} = imm;
 231:   let BPFClass = BPF_JMP;
 232: }
 233: 
 234: class JMP_IND<BPFJumpOp Opc, string OpcodeStr, list<dag> Pattern>
```

- EN: This range defines declarative TableGen records such as JMP_RI, JMP_IND, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 JMP_RI, JMP_IND 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 235-252

```tablegen
 235:     : TYPE_ALU_JMP<Opc.Value, BPF_X.Value,
 236:                    (outs),
 237:                    (ins GPR:$dst),
 238:                    !strconcat(OpcodeStr, " $dst"),
 239:                    Pattern> {
 240:   bits<4> dst;
 241: 
 242:   let Inst{51-48} = dst;
 243:   let BPFClass = BPF_JMP;
 244: }
 245: 
 246: class JMP_JCOND<BPFJumpOp Opc, string OpcodeStr, list<dag> Pattern>
 247:     : TYPE_ALU_JMP<Opc.Value, BPF_K.Value,
 248:                    (outs),
 249:                    (ins brtarget:$BrDst),
 250:                    !strconcat(OpcodeStr, " $BrDst"),
 251:                    Pattern> {
 252:   bits<16> BrDst;
```

- EN: This range defines declarative TableGen records such as JMP_JCOND, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 JMP_JCOND 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 253-270

```tablegen
 253: 
 254:   let Inst{47-32} = BrDst;
 255:   let BPFClass = BPF_JMP;
 256: }
 257: 
 258: class JMP_RR_32<BPFJumpOp Opc, string OpcodeStr, PatLeaf Cond>
 259:     : TYPE_ALU_JMP<Opc.Value, BPF_X.Value,
 260:                    (outs),
 261:                    (ins GPR32:$dst, GPR32:$src, brtarget:$BrDst),
 262:                    "if $dst "#OpcodeStr#" $src goto $BrDst",
 263:                    [(BPFbrcc i32:$dst, i32:$src, Cond, bb:$BrDst)]> {
 264:   bits<4> dst;
 265:   bits<4> src;
 266:   bits<16> BrDst;
 267: 
 268:   let Inst{55-52} = src;
 269:   let Inst{51-48} = dst;
 270:   let Inst{47-32} = BrDst;
```

- EN: This range defines declarative TableGen records such as JMP_RR_32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 JMP_RR_32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-288

```tablegen
 271:   let BPFClass = BPF_JMP32;
 272: }
 273: 
 274: class JMP_RI_32<BPFJumpOp Opc, string OpcodeStr, PatLeaf Cond>
 275:     : TYPE_ALU_JMP<Opc.Value, BPF_K.Value,
 276:                    (outs),
 277:                    (ins GPR32:$dst, i32imm:$imm, brtarget:$BrDst),
 278:                    "if $dst "#OpcodeStr#" $imm goto $BrDst",
 279:                    [(BPFbrcc i32:$dst, i32immSExt32:$imm, Cond, bb:$BrDst)]> {
 280:   bits<4> dst;
 281:   bits<16> BrDst;
 282:   bits<32> imm;
 283: 
 284:   let Inst{51-48} = dst;
 285:   let Inst{47-32} = BrDst;
 286:   let Inst{31-0} = imm;
 287:   let BPFClass = BPF_JMP32;
 288: }
```

- EN: This range defines declarative TableGen records such as JMP_RI_32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 JMP_RI_32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 289-306

```tablegen
 289: 
 290: multiclass J<BPFJumpOp Opc, string OpcodeStr, PatLeaf Cond, PatLeaf Cond32> {
 291:   def _rr : JMP_RR<Opc, OpcodeStr, Cond>;
 292:   def _ri : JMP_RI<Opc, OpcodeStr, Cond>;
 293:   def _rr_32 : JMP_RR_32<Opc, OpcodeStr, Cond32>;
 294:   def _ri_32 : JMP_RI_32<Opc, OpcodeStr, Cond32>;
 295: }
 296: 
 297: let isBranch = 1, isTerminator = 1, hasDelaySlot=0 in {
 298: // cmp+goto instructions
 299: defm JEQ  : J<BPF_JEQ, "==",  BPF_CC_EQ, BPF_CC_EQ_32>;
 300: defm JUGT : J<BPF_JGT, ">", BPF_CC_GTU, BPF_CC_GTU_32>;
 301: defm JUGE : J<BPF_JGE, ">=", BPF_CC_GEU, BPF_CC_GEU_32>;
 302: defm JNE  : J<BPF_JNE, "!=",  BPF_CC_NE, BPF_CC_NE_32>;
 303: defm JSGT : J<BPF_JSGT, "s>", BPF_CC_GT, BPF_CC_GT_32>;
 304: defm JSGE : J<BPF_JSGE, "s>=", BPF_CC_GE, BPF_CC_GE_32>;
 305: defm JULT : J<BPF_JLT, "<", BPF_CC_LTU, BPF_CC_LTU_32>;
 306: defm JULE : J<BPF_JLE, "<=", BPF_CC_LEU, BPF_CC_LEU_32>;
```

- EN: This range defines declarative TableGen records such as J, _rr, _ri, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 J, _rr, _ri 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 307-324

```tablegen
 307: defm JSLT : J<BPF_JSLT, "s<", BPF_CC_LT, BPF_CC_LT_32>;
 308: defm JSLE : J<BPF_JSLE, "s<=", BPF_CC_LE, BPF_CC_LE_32>;
 309: defm JSET : J<BPF_JSET, "&", NoCond, NoCond>;
 310: def JCOND : JMP_JCOND<BPF_JCOND, "may_goto", []>;
 311: 
 312: let Predicates = [BPFHasGotox] in {
 313:   let isIndirectBranch = 1, isBarrier = 1 in {
 314:     def JX : JMP_IND<BPF_JA, "gotox", [(brind i64:$dst)]>;
 315:   }
 316: }
 317: }
 318: 
 319: let hasSideEffects = 1, mayLoad = 1 in {
 320:   def LOAD_STACK_ARG_PSEUDO
 321:     : Pseudo<(outs GPR:$dst), (ins s16imm:$off),
 322:              "load_stack_arg\t$dst, $off", []>;
 323: }
 324: 
```

- EN: This range defines declarative TableGen records such as JSLT, JSLE, JSET, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 JSLT, JSLE, JSET 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 325-342

```tablegen
 325: let hasSideEffects = 1, mayStore = 1 in {
 326:   def STORE_STACK_ARG_PSEUDO
 327:     : Pseudo<(outs), (ins s16imm:$off, GPR:$src),
 328:              "store_stack_arg\t$off, $src", []>;
 329: 
 330:   def STORE_STACK_ARG_IMM_PSEUDO
 331:     : Pseudo<(outs), (ins s16imm:$off, i32imm:$val),
 332:              "store_stack_arg_imm\t$off, $val", []>;
 333: }
 334: 
 335: // ALU instructions
 336: class ALU_RI<BPFOpClass Class, BPFArithOp Opc, int off,
 337:              dag outs, dag ins, string asmstr, list<dag> pattern>
 338:     : TYPE_ALU_JMP<Opc.Value, BPF_K.Value, outs, ins, asmstr, pattern> {
 339:   bits<4> dst;
 340:   bits<32> imm;
 341: 
 342:   let Inst{51-48} = dst;
```

- EN: This range defines declarative TableGen records such as STORE_STACK_ARG_PSEUDO, STORE_STACK_ARG_IMM_PSEUDO, ALU_RI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STORE_STACK_ARG_PSEUDO, STORE_STACK_ARG_IMM_PSEUDO, ALU_RI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 343-360

```tablegen
 343:   let Inst{47-32} = off;
 344:   let Inst{31-0} = imm;
 345:   let BPFClass = Class;
 346: }
 347: 
 348: class ALU_RR<BPFOpClass Class, BPFArithOp Opc, int off,
 349:              dag outs, dag ins, string asmstr, list<dag> pattern>
 350:     : TYPE_ALU_JMP<Opc.Value, BPF_X.Value, outs, ins, asmstr, pattern> {
 351:   bits<4> dst;
 352:   bits<4> src;
 353: 
 354:   let Inst{55-52} = src;
 355:   let Inst{51-48} = dst;
 356:   let Inst{47-32} = off;
 357:   let BPFClass = Class;
 358: }
 359: 
 360: multiclass ALU<BPFArithOp Opc, int off, string OpcodeStr, SDNode OpNode> {
```

- EN: This range defines declarative TableGen records such as ALU_RR, ALU, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ALU_RR, ALU 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 361-378

```tablegen
 361:   def _rr : ALU_RR<BPF_ALU64, Opc, off,
 362:                    (outs GPR:$dst),
 363:                    (ins GPR:$src2, GPR:$src),
 364:                    "$dst "#OpcodeStr#" $src",
 365:                    [(set GPR:$dst, (OpNode i64:$src2, i64:$src))]>;
 366:   def _ri : ALU_RI<BPF_ALU64, Opc, off,
 367:                    (outs GPR:$dst),
 368:                    (ins GPR:$src2, i64imm:$imm),
 369:                    "$dst "#OpcodeStr#" $imm",
 370:                    [(set GPR:$dst, (OpNode GPR:$src2, i64immSExt32:$imm))]>;
 371:   def _rr_32 : ALU_RR<BPF_ALU, Opc, off,
 372:                    (outs GPR32:$dst),
 373:                    (ins GPR32:$src2, GPR32:$src),
 374:                    "$dst "#OpcodeStr#" $src",
 375:                    [(set GPR32:$dst, (OpNode i32:$src2, i32:$src))]>;
 376:   def _ri_32 : ALU_RI<BPF_ALU, Opc, off,
 377:                    (outs GPR32:$dst),
 378:                    (ins GPR32:$src2, i32imm:$imm),
```

- EN: This range defines declarative TableGen records such as _rr, _ri, _rr_32, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 _rr, _ri, _rr_32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 379-396

```tablegen
 379:                    "$dst "#OpcodeStr#" $imm",
 380:                    [(set GPR32:$dst, (OpNode GPR32:$src2, i32immSExt32:$imm))]>;
 381: }
 382: 
 383: let Constraints = "$dst = $src2" in {
 384: let isAsCheapAsAMove = 1 in {
 385:   defm ADD : ALU<BPF_ADD, 0, "+=", add>;
 386:   defm SUB : ALU<BPF_SUB, 0, "-=", sub>;
 387:   defm OR  : ALU<BPF_OR, 0, "|=", or>;
 388:   defm AND : ALU<BPF_AND, 0, "&=", and>;
 389:   defm SLL : ALU<BPF_LSH, 0, "<<=", shl>;
 390:   defm SRL : ALU<BPF_RSH, 0, ">>=", srl>;
 391:   defm XOR : ALU<BPF_XOR, 0, "^=", xor>;
 392:   defm SRA : ALU<BPF_ARSH, 0, "s>>=", sra>;
 393: }
 394:   defm MUL : ALU<BPF_MUL, 0, "*=", mul>;
 395:   defm DIV : ALU<BPF_DIV, 0, "/=", udiv>;
 396:   defm MOD : ALU<BPF_MOD, 0, "%=", urem>;
```

- EN: This range defines declarative TableGen records such as ADD, SUB, OR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADD, SUB, OR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 397-414

```tablegen
 397: 
 398:   let Predicates = [BPFHasSdivSmod] in {
 399:     defm SDIV : ALU<BPF_DIV, 1, "s/=", sdiv>;
 400:     defm SMOD : ALU<BPF_MOD, 1, "s%=", srem>;
 401:   }
 402: }
 403: 
 404: class NEG_RR<BPFOpClass Class, BPFArithOp Opc,
 405:              dag outs, dag ins, string asmstr, list<dag> pattern>
 406:     : TYPE_ALU_JMP<Opc.Value, 0, outs, ins, asmstr, pattern> {
 407:   bits<4> dst;
 408: 
 409:   let Inst{51-48} = dst;
 410:   let BPFClass = Class;
 411: }
 412: 
 413: let Constraints = "$dst = $src", isAsCheapAsAMove = 1 in {
 414:   def NEG_64: NEG_RR<BPF_ALU64, BPF_NEG, (outs GPR:$dst), (ins GPR:$src),
```

- EN: This range defines declarative TableGen records such as SDIV, SMOD, NEG_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SDIV, SMOD, NEG_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 415-432

```tablegen
 415:                      "$dst = -$src",
 416:                      [(set GPR:$dst, (ineg i64:$src))]>;
 417:   def NEG_32: NEG_RR<BPF_ALU, BPF_NEG, (outs GPR32:$dst), (ins GPR32:$src),
 418:                      "$dst = -$src",
 419:                      [(set GPR32:$dst, (ineg i32:$src))]>;
 420: }
 421: 
 422: class LD_IMM64<bits<4> Pseudo, string OpcodeStr>
 423:     : TYPE_LD_ST<BPF_IMM.Value, BPF_DW.Value,
 424:                  (outs GPR:$dst),
 425:                  (ins u64imm:$imm),
 426:                  "$dst "#OpcodeStr#" ${imm} ll",
 427:                  [(set GPR:$dst, (i64 imm:$imm))]> {
 428: 
 429:   bits<4> dst;
 430:   bits<64> imm;
 431: 
 432:   let Inst{51-48} = dst;
```

- EN: This range defines declarative TableGen records such as NEG_32, LD_IMM64, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 NEG_32, LD_IMM64 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 433-450

```tablegen
 433:   let Inst{55-52} = Pseudo;
 434:   let Inst{47-32} = 0;
 435:   let Inst{31-0} = imm{31-0};
 436:   let BPFClass = BPF_LD;
 437: }
 438: 
 439: let isReMaterializable = 1, isAsCheapAsAMove = 1 in {
 440: def LD_imm64 : LD_IMM64<0, "=">;
 441: def MOV_rr : ALU_RR<BPF_ALU64, BPF_MOV, 0,
 442:                     (outs GPR:$dst),
 443:                     (ins GPR:$src),
 444:                     "$dst = $src",
 445:                     []>;
 446: def MOV_ri : ALU_RI<BPF_ALU64, BPF_MOV, 0,
 447:                     (outs GPR:$dst),
 448:                     (ins i64imm:$imm),
 449:                     "$dst = $imm",
 450:                     [(set GPR:$dst, (i64 i64immSExt32:$imm))]>;
```

- EN: This range defines declarative TableGen records such as LD_imm64, MOV_rr, MOV_ri, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LD_imm64, MOV_rr, MOV_ri 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 451-468

```tablegen
 451: def MOV_rr_32 : ALU_RR<BPF_ALU, BPF_MOV, 0,
 452:                     (outs GPR32:$dst),
 453:                     (ins GPR32:$src),
 454:                     "$dst = $src",
 455:                     []>;
 456: def MOV_ri_32 : ALU_RI<BPF_ALU, BPF_MOV, 0,
 457:                     (outs GPR32:$dst),
 458:                     (ins i32imm:$imm),
 459:                     "$dst = $imm",
 460:                     [(set GPR32:$dst, (i32 i32immSExt32:$imm))]>;
 461: 
 462: let Predicates = [BPFHasMovsx] in {
 463:   def MOVSX_rr_8 : ALU_RR<BPF_ALU64, BPF_MOV, 8,
 464:                       (outs GPR:$dst), (ins GPR:$src),
 465:                       "$dst = (s8)$src",
 466:                       [(set GPR:$dst, (sext_inreg GPR:$src, i8))]>;
 467:   def MOVSX_rr_16 : ALU_RR<BPF_ALU64, BPF_MOV, 16,
 468:                       (outs GPR:$dst), (ins GPR:$src),
```

- EN: This range defines declarative TableGen records such as MOV_rr_32, MOV_ri_32, MOVSX_rr_8, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MOV_rr_32, MOV_ri_32, MOVSX_rr_8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 469-486

```tablegen
 469:                       "$dst = (s16)$src",
 470:                       [(set GPR:$dst, (sext_inreg GPR:$src, i16))]>;
 471:   def MOVSX_rr_32 : ALU_RR<BPF_ALU64, BPF_MOV, 32,
 472:                       (outs GPR:$dst), (ins GPR:$src),
 473:                       "$dst = (s32)$src",
 474:                       [(set GPR:$dst, (sext_inreg GPR:$src, i32))]>;
 475:   def MOVSX_rr_32_8 : ALU_RR<BPF_ALU, BPF_MOV, 8,
 476:                       (outs GPR32:$dst), (ins GPR32:$src),
 477:                       "$dst = (s8)$src",
 478:                       [(set GPR32:$dst, (sext_inreg GPR32:$src, i8))]>;
 479:   def MOVSX_rr_32_16 : ALU_RR<BPF_ALU, BPF_MOV, 16,
 480:                       (outs GPR32:$dst), (ins GPR32:$src),
 481:                       "$dst = (s16)$src",
 482:                       [(set GPR32:$dst, (sext_inreg GPR32:$src, i16))]>;
 483: }
 484: }
 485: 
 486: def ADDR_SPACE_CAST
```

- EN: This range defines declarative TableGen records such as MOVSX_rr_32, MOVSX_rr_32_8, MOVSX_rr_32_16, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 MOVSX_rr_32, MOVSX_rr_32_8, MOVSX_rr_32_16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 487-504

```tablegen
 487:     : ALU_RR<BPF_ALU64, BPF_MOV, 64,
 488:              (outs GPR:$dst),
 489:              (ins GPR:$src, i64imm:$dst_as, i64imm:$src_as),
 490:              "$dst = addr_space_cast($src, $dst_as, $src_as)",
 491:              []> {
 492:   bits<64> dst_as;
 493:   bits<64> src_as;
 494: 
 495:   let Inst{47-32} = 1;
 496:   let Inst{31-16} = dst_as{15-0};
 497:   let Inst{15-0} = src_as{15-0};
 498: }
 499: 
 500: def SrcAddrSpace : SDNodeXForm<addrspacecast, [{
 501:   return CurDAG->getTargetConstant(
 502:     cast<AddrSpaceCastSDNode>(N)->getSrcAddressSpace(),
 503:     SDLoc(N), MVT::i64);
 504: }]>;
```

- EN: This range defines declarative TableGen records such as SrcAddrSpace, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SrcAddrSpace 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 505-522

```tablegen
 505: 
 506: def DstAddrSpace : SDNodeXForm<addrspacecast, [{
 507:   return CurDAG->getTargetConstant(
 508:     cast<AddrSpaceCastSDNode>(N)->getDestAddressSpace(),
 509:     SDLoc(N), MVT::i64);
 510: }]>;
 511: 
 512: def : Pat<(addrspacecast:$this GPR:$src),
 513:           (ADDR_SPACE_CAST $src, (DstAddrSpace $this), (SrcAddrSpace $this))>;
 514: 
 515: def FI_ri
 516:     : TYPE_LD_ST<BPF_IMM.Value, BPF_DW.Value,
 517:                  (outs GPR:$dst),
 518:                  (ins MEMri:$addr),
 519:                  "lea\t$dst, $addr",
 520:                  [(set i64:$dst, FIri:$addr)]> {
 521:   // This is a tentative instruction, and will be replaced
 522:   // with MOV_rr and ADD_ri in PEI phase
```

- EN: This range defines declarative TableGen records such as DstAddrSpace, FI_ri, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 DstAddrSpace, FI_ri 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 523-540

```tablegen
 523:   let Inst{51-48} = 0;
 524:   let Inst{55-52} = 2;
 525:   let Inst{47-32} = 0;
 526:   let Inst{31-0} = 0;
 527:   let BPFClass = BPF_LD;
 528:   bit isPseudo = true;
 529: }
 530: 
 531: def LD_pseudo
 532:     : TYPE_LD_ST<BPF_IMM.Value, BPF_DW.Value,
 533:                  (outs GPR:$dst),
 534:                  (ins i64imm:$pseudo, u64imm:$imm),
 535:                  "ld_pseudo\t$dst, $pseudo, $imm",
 536:                  [(set GPR:$dst, (int_bpf_pseudo imm:$pseudo, imm:$imm))]> {
 537: 
 538:   bits<4> dst;
 539:   bits<64> imm;
 540:   bits<4> pseudo;
```

- EN: This range defines declarative TableGen records such as LD_pseudo, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LD_pseudo 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 541-558

```tablegen
 541: 
 542:   let Inst{51-48} = dst;
 543:   let Inst{55-52} = pseudo;
 544:   let Inst{47-32} = 0;
 545:   let Inst{31-0} = imm{31-0};
 546:   let BPFClass = BPF_LD;
 547: }
 548: 
 549: // STORE instructions
 550: class STORE<BPFWidthModifer SizeOp, string OpcodeStr, list<dag> Pattern>
 551:     : TYPE_LD_ST<BPF_MEM.Value, SizeOp.Value,
 552:                  (outs),
 553:                  (ins GPR:$src, MEMri:$addr),
 554:                  "*("#OpcodeStr#" *)($addr) = $src",
 555:                  Pattern> {
 556:   bits<4> src;
 557:   bits<20> addr;
 558: 
```

- EN: This range defines declarative TableGen records such as STORE, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 STORE 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 559-576

```tablegen
 559:   let Inst{51-48} = addr{19-16}; // base reg
 560:   let Inst{55-52} = src;
 561:   let Inst{47-32} = addr{15-0}; // offset
 562:   let BPFClass = BPF_STX;
 563: }
 564: 
 565: class STOREi64<BPFWidthModifer Opc, string OpcodeStr, PatFrag OpNode>
 566:     : STORE<Opc, OpcodeStr, [(OpNode GPR:$src, ADDRri:$addr)]>;
 567: 
 568: let Predicates = [BPFNoALU32] in {
 569:   def STW : STOREi64<BPF_W, "u32", truncstorei32>;
 570:   def STH : STOREi64<BPF_H, "u16", truncstorei16>;
 571:   def STB : STOREi64<BPF_B, "u8", truncstorei8>;
 572: }
 573: def STD : STOREi64<BPF_DW, "u64", store>;
 574: 
 575: class STORE_imm<BPFWidthModifer SizeOp,
 576:                 string OpcodeStr, dag Pattern>
```

- EN: This range defines declarative TableGen records such as STOREi64, STW, STH, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 STOREi64, STW, STH 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 577-594

```tablegen
 577:     : TYPE_LD_ST<BPF_MEM.Value, SizeOp.Value,
 578:                  (outs),
 579:                  (ins i64imm:$imm, MEMri:$addr),
 580:                  "*("#OpcodeStr#" *)($addr) = $imm",
 581:                  [Pattern]> {
 582:   bits<20> addr;
 583:   bits<32> imm;
 584: 
 585:   let Inst{51-48} = addr{19-16}; // base reg
 586:   let Inst{47-32} = addr{15-0};  // offset
 587:   let Inst{31-0} = imm;
 588:   let BPFClass = BPF_ST;
 589: }
 590: 
 591: let Predicates = [BPFHasStoreImm] in {
 592:   // Opcode (BPF_ST | BPF_MEM | BPF_DW) implies sign extension for
 593:   // value stored to memory:
 594:   // - it is fine to generate such write when immediate is -1
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 595-612

```tablegen
 595:   // - it is incorrect to generate such write when immediate is
 596:   //   +0xffff_ffff.
 597:   //
 598:   // In the latter case two instructions would be generated instead of
 599:   // one BPF_ST:
 600:   //   rA = 0xffffffff ll     ; LD_imm64
 601:   //   *(u64 *)(rB + 0) = rA  ; STX
 602:   //
 603:   // For BPF_{B,H,W} the size of value stored matches size of the immediate.
 604:   def STD_imm : STORE_imm<BPF_DW, "u64", (store         (i64 i64immSExt32:$imm), ADDRri:$addr)>;
 605:   def STW_imm : STORE_imm<BPF_W,  "u32", (truncstorei32 (i64 i64immZExt32:$imm), ADDRri:$addr)>;
 606:   def STH_imm : STORE_imm<BPF_H,  "u16", (truncstorei16 (i64 i64immZExt32:$imm), ADDRri:$addr)>;
 607:   def STB_imm : STORE_imm<BPF_B,  "u8",  (truncstorei8  (i64 i64immZExt32:$imm), ADDRri:$addr)>;
 608: }
 609: 
 610: let Predicates = [BPFHasALU32, BPFHasStoreImm] in {
 611:   def : Pat<(store (i32 imm:$src), ADDRri:$dst),
 612:             (STW_imm (imm_to_i64 $src), ADDRri:$dst)>;
```

- EN: This range defines declarative TableGen records such as STD_imm, STW_imm, STH_imm, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 STD_imm, STW_imm, STH_imm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 613-630

```tablegen
 613:   def : Pat<(truncstorei16 (i32 imm:$src), ADDRri:$dst),
 614:             (STH_imm (imm_to_i64 imm:$src), ADDRri:$dst)>;
 615:   def : Pat<(truncstorei8 (i32 imm:$src), ADDRri:$dst),
 616:             (STB_imm (imm_to_i64 imm:$src), ADDRri:$dst)>;
 617: }
 618: 
 619: class STORE_RELEASE<BPFWidthModifer SizeOp, string OpcodeStr, RegisterClass RegTp>
 620:     : TYPE_LD_ST<BPF_ATOMIC.Value, SizeOp.Value,
 621:                  (outs),
 622:                  (ins RegTp:$src, MEMri:$addr),
 623:                  "store_release(("#OpcodeStr#" *)($addr), $src)",
 624:                  []> {
 625:   bits<4> src;
 626:   bits<20> addr;
 627: 
 628:   let Inst{51-48} = addr{19-16}; // base reg
 629:   let Inst{55-52} = src;
 630:   let Inst{47-32} = addr{15-0}; // offset
```

- EN: This range defines declarative TableGen records such as STORE_RELEASE, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 STORE_RELEASE 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 631-648

```tablegen
 631:   let Inst{8-4} = BPF_STORE_REL.Value;
 632:   let BPFClass = BPF_STX;
 633: }
 634: 
 635: class STORE_RELEASEi64<BPFWidthModifer Opc, string OpcodeStr>
 636:     : STORE_RELEASE<Opc, OpcodeStr, GPR>;
 637: 
 638: class relaxed_store<PatFrag base>
 639:   : PatFrag<(ops node:$val, node:$ptr), (base node:$val, node:$ptr)> {
 640:   let IsAtomic = 1;
 641:   let IsAtomicOrderingReleaseOrStronger = 0;
 642: }
 643: 
 644: class releasing_store<PatFrag base>
 645:   : PatFrag<(ops node:$val, node:$ptr), (base node:$val, node:$ptr)> {
 646:   let IsAtomic = 1;
 647:   let IsAtomicOrderingRelease = 1;
 648: }
```

- EN: This range defines declarative TableGen records such as STORE_RELEASEi64, relaxed_store, releasing_store, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STORE_RELEASEi64, relaxed_store, releasing_store 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 649-666

```tablegen
 649: 
 650: let Predicates = [BPFHasLoadAcqStoreRel] in {
 651:   def STDREL : STORE_RELEASEi64<BPF_DW, "u64">;
 652: 
 653:   foreach P = [[relaxed_store<atomic_store_64>, STD],
 654:                [releasing_store<atomic_store_64>, STDREL],
 655:               ] in {
 656:     def : Pat<(P[0] GPR:$val, ADDRri:$addr), (P[1] GPR:$val, ADDRri:$addr)>;
 657:   }
 658: }
 659: 
 660: // LOAD instructions
 661: class LOAD<BPFWidthModifer SizeOp, BPFModeModifer ModOp, string OpcodeStr, list<dag> Pattern>
 662:     : TYPE_LD_ST<ModOp.Value, SizeOp.Value,
 663:                  (outs GPR:$dst),
 664:                  (ins MEMri:$addr),
 665:                  "$dst = *("#OpcodeStr#" *)($addr)",
 666:                  Pattern> {
```

- EN: This range defines declarative TableGen records such as STDREL, LOAD, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 STDREL, LOAD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 667-684

```tablegen
 667:   bits<4> dst;
 668:   bits<20> addr;
 669: 
 670:   let Inst{51-48} = dst;
 671:   let Inst{55-52} = addr{19-16};
 672:   let Inst{47-32} = addr{15-0};
 673:   let BPFClass = BPF_LDX;
 674: }
 675: 
 676: class LOADi64<BPFWidthModifer SizeOp, BPFModeModifer ModOp, string OpcodeStr, PatFrag OpNode>
 677:     : LOAD<SizeOp, ModOp, OpcodeStr, [(set i64:$dst, (OpNode ADDRri:$addr))]>;
 678: 
 679: let isCodeGenOnly = 1 in {
 680:   class CORE_LD<RegisterClass RegClass, string Sz>
 681:                 : TYPE_LD_ST<BPF_MEM.Value, BPF_W.Value,
 682:                              (outs RegClass:$dst),
 683:                              (ins u64imm:$opcode, GPR:$src, u64imm:$offset),
 684:                              "$dst = core_ld"#Sz#"($opcode, $src, $offset)",
```

- EN: This range defines declarative TableGen records such as LOADi64, CORE_LD, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LOADi64, CORE_LD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 685-702

```tablegen
 685:                              []>;
 686:   def CORE_LD64 : CORE_LD<GPR, "64">;
 687:   def CORE_LD32 : CORE_LD<GPR32, "32">;
 688:   def CORE_ST   : TYPE_LD_ST<BPF_MEM.Value, BPF_W.Value,
 689:                              (outs),
 690:                              (ins gpr_or_imm:$src, u64imm:$opcode, GPR:$ptr, u64imm:$offset),
 691:                              "core_st($src, $opcode, $ptr, $offset)",
 692:                              []>;
 693:   let Constraints = "$dst = $src" in {
 694:     def CORE_SHIFT : ALU_RR<BPF_ALU64, BPF_LSH, 0,
 695:                              (outs GPR:$dst),
 696:                              (ins u64imm:$opcode, GPR:$src, u64imm:$offset),
 697:                              "$dst = core_shift($opcode, $src, $offset)",
 698:                              []>;
 699:   }
 700: }
 701: 
 702: let Predicates = [BPFNoALU32] in {
```

- EN: This range defines declarative TableGen records such as CORE_LD64, CORE_LD32, CORE_ST, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CORE_LD64, CORE_LD32, CORE_ST 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 703-720

```tablegen
 703:   def LDW : LOADi64<BPF_W, BPF_MEM, "u32", zextloadi32>;
 704:   def LDH : LOADi64<BPF_H, BPF_MEM, "u16", zextloadi16>;
 705:   def LDB : LOADi64<BPF_B, BPF_MEM, "u8", zextloadi8>;
 706: }
 707: 
 708: let Predicates = [BPFHasLdsx] in {
 709:   def LDWSX : LOADi64<BPF_W, BPF_MEMSX, "s32", sextloadi32>;
 710:   def LDHSX : LOADi64<BPF_H, BPF_MEMSX, "s16", sextloadi16>;
 711:   def LDBSX : LOADi64<BPF_B, BPF_MEMSX, "s8",  sextloadi8>;
 712: }
 713: 
 714: def LDD : LOADi64<BPF_DW, BPF_MEM, "u64", load>;
 715: 
 716: class LOAD_ACQUIRE<BPFWidthModifer SizeOp, string OpcodeStr, RegisterClass RegTp>
 717:     : TYPE_LD_ST<BPF_ATOMIC.Value, SizeOp.Value,
 718:                  (outs RegTp:$dst),
 719:                  (ins MEMri:$addr),
 720:                  "$dst = load_acquire(("#OpcodeStr#" *)($addr))",
```

- EN: This range defines declarative TableGen records such as LDW, LDH, LDB, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDW, LDH, LDB 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 721-738

```tablegen
 721:                  []> {
 722:   bits<4> dst;
 723:   bits<20> addr;
 724: 
 725:   let Inst{51-48} = dst;
 726:   let Inst{55-52} = addr{19-16}; // base reg
 727:   let Inst{47-32} = addr{15-0}; // offset
 728:   let Inst{8-4} = BPF_LOAD_ACQ.Value;
 729:   let BPFClass = BPF_STX;
 730: }
 731: 
 732: class LOAD_ACQUIREi64<BPFWidthModifer SizeOp, string OpcodeStr>
 733:     : LOAD_ACQUIRE<SizeOp, OpcodeStr, GPR>;
 734: 
 735: class relaxed_load<PatFrags base>
 736:     : PatFrag<(ops node:$ptr), (base node:$ptr)> {
 737:   let IsAtomic = 1;
 738:   let IsAtomicOrderingAcquireOrStronger = 0;
```

- EN: This range defines declarative TableGen records such as LOAD_ACQUIREi64, relaxed_load, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LOAD_ACQUIREi64, relaxed_load 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 739-756

```tablegen
 739: }
 740: 
 741: class acquiring_load<PatFrags base>
 742:     : PatFrag<(ops node:$ptr), (base node:$ptr)> {
 743:   let IsAtomic = 1;
 744:   let IsAtomicOrderingAcquire = 1;
 745: }
 746: 
 747: let Predicates = [BPFHasLoadAcqStoreRel] in {
 748:   def LDDACQ : LOAD_ACQUIREi64<BPF_DW, "u64">;
 749: 
 750:   foreach P = [[relaxed_load<atomic_load_nonext_64>, LDD],
 751:                [acquiring_load<atomic_load_nonext_64>, LDDACQ],
 752:               ] in {
 753:     def : Pat<(P[0] ADDRri:$addr), (P[1] ADDRri:$addr)>;
 754:   }
 755: }
 756: 
```

- EN: This range defines declarative TableGen records such as acquiring_load, LDDACQ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 acquiring_load, LDDACQ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 757-774

```tablegen
 757: class BRANCH<BPFJumpOp Opc, string OpcodeStr, list<dag> Pattern>
 758:     : TYPE_ALU_JMP<Opc.Value, BPF_K.Value,
 759:                    (outs),
 760:                    (ins brtarget:$BrDst),
 761:                    !strconcat(OpcodeStr, " $BrDst"),
 762:                    Pattern> {
 763:   bits<16> BrDst;
 764: 
 765:   let Inst{47-32} = BrDst;
 766:   let BPFClass = BPF_JMP;
 767: }
 768: 
 769: class BRANCH_LONG<BPFJumpOp Opc, string OpcodeStr, list<dag> Pattern>
 770:     : TYPE_ALU_JMP<Opc.Value, BPF_K.Value,
 771:                    (outs),
 772:                    (ins brtarget:$BrDst),
 773:                    !strconcat(OpcodeStr, " $BrDst"),
 774:                    Pattern> {
```

- EN: This range defines declarative TableGen records such as BRANCH, BRANCH_LONG, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BRANCH, BRANCH_LONG 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 775-792

```tablegen
 775:   bits<32> BrDst;
 776: 
 777:   let Inst{31-0} = BrDst;
 778:   let BPFClass = BPF_JMP32;
 779: }
 780: 
 781: class CALL<string OpcodeStr>
 782:     : TYPE_ALU_JMP<BPF_CALL.Value, BPF_K.Value,
 783:                    (outs),
 784:                    (ins calltarget:$BrDst),
 785:                    !strconcat(OpcodeStr, " $BrDst"),
 786:                    []> {
 787:   bits<32> BrDst;
 788: 
 789:   let Inst{31-0} = BrDst;
 790:   let BPFClass = BPF_JMP;
 791: }
 792: 
```

- EN: This range defines declarative TableGen records such as CALL, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CALL 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 793-810

```tablegen
 793: class CALLX<string OpcodeStr>
 794:     : TYPE_ALU_JMP<BPF_CALL.Value, BPF_X.Value,
 795:                    (outs),
 796:                    (ins GPR:$BrDst),
 797:                    !strconcat(OpcodeStr, " $BrDst"),
 798:                    []> {
 799:   bits<4> BrDst;
 800: 
 801:   let Inst{51-48} = BrDst;
 802:   let BPFClass = BPF_JMP;
 803: }
 804: 
 805: // Jump always
 806: let isBranch = 1, isTerminator = 1, hasDelaySlot=0, isBarrier = 1 in {
 807:   def JMP : BRANCH<BPF_JA, "goto", [(br bb:$BrDst)]>;
 808:   def JMPL : BRANCH_LONG<BPF_JA, "gotol", []>;
 809: }
 810: 
```

- EN: This range defines declarative TableGen records such as CALLX, JMP, JMPL, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CALLX, JMP, JMPL 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 811-828

```tablegen
 811: // Jump and link
 812: let isCall=1, hasDelaySlot=0, Uses = [R11] in {
 813:   def JAL  : CALL<"call">;
 814:   def JALX  : CALLX<"callx">;
 815: }
 816: 
 817: class NOP_I<string OpcodeStr>
 818:     : TYPE_ALU_JMP<BPF_MOV.Value, BPF_X.Value,
 819:                    (outs),
 820:                    (ins i32imm:$imm),
 821:                    !strconcat(OpcodeStr, "\t$imm"),
 822:                    []> {
 823:   // mov r0, r0 == nop
 824:   let Inst{55-52} = 0;
 825:   let Inst{51-48} = 0;
 826:   let BPFClass = BPF_ALU64;
 827: }
 828: 
```

- EN: This range defines declarative TableGen records such as JAL, JALX, NOP_I, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 JAL, JALX, NOP_I 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 829-846

```tablegen
 829: let hasSideEffects = 0, isCodeGenOnly = 1 in
 830:   def NOP : NOP_I<"nop">;
 831: 
 832: class RET<string OpcodeStr>
 833:     : TYPE_ALU_JMP<BPF_EXIT.Value, BPF_K.Value,
 834:                    (outs),
 835:                    (ins),
 836:                    !strconcat(OpcodeStr, ""),
 837:                    [(BPFretglue)]> {
 838:   let Inst{31-0} = 0;
 839:   let BPFClass = BPF_JMP;
 840: }
 841: 
 842: let isReturn = 1, isTerminator = 1, hasDelaySlot=0, isBarrier = 1,
 843:     isNotDuplicable = 1 in {
 844:   def RET : RET<"exit">;
 845: }
 846: 
```

- EN: This range defines declarative TableGen records such as NOP, RET, RET, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 NOP, RET, RET 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 847-864

```tablegen
 847: // ADJCALLSTACKDOWN/UP pseudo insns
 848: let Defs = [R11], Uses = [R11], isCodeGenOnly = 1 in {
 849: def ADJCALLSTACKDOWN : Pseudo<(outs), (ins i64imm:$amt1, i64imm:$amt2),
 850:                               "#ADJCALLSTACKDOWN $amt1 $amt2",
 851:                               [(BPFcallseq_start timm:$amt1, timm:$amt2)]>;
 852: def ADJCALLSTACKUP   : Pseudo<(outs), (ins i64imm:$amt1, i64imm:$amt2),
 853:                               "#ADJCALLSTACKUP $amt1 $amt2",
 854:                               [(BPFcallseq_end timm:$amt1, timm:$amt2)]>;
 855: }
 856: 
 857: let usesCustomInserter = 1, isCodeGenOnly = 1 in {
 858:   def Select : Pseudo<(outs GPR:$dst),
 859:                       (ins GPR:$lhs, GPR:$rhs, i64imm:$imm, GPR:$src, GPR:$src2),
 860:                       "# Select PSEUDO $dst = $lhs $imm $rhs ? $src : $src2",
 861:                       [(set i64:$dst,
 862:                        (BPFselectcc i64:$lhs, i64:$rhs, (i64 imm:$imm), i64:$src, i64:$src2))]>;
 863:   def Select_Ri : Pseudo<(outs GPR:$dst),
 864:                       (ins GPR:$lhs, i64imm:$rhs, i64imm:$imm, GPR:$src, GPR:$src2),
```

- EN: This range defines declarative TableGen records such as ADJCALLSTACKDOWN, ADJCALLSTACKUP, Select, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADJCALLSTACKDOWN, ADJCALLSTACKUP, Select 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 865-882

```tablegen
 865:                       "# Select PSEUDO $dst = $lhs $imm $rhs ? $src : $src2",
 866:                       [(set i64:$dst,
 867:                        (BPFselectcc i64:$lhs, (i64immSExt32:$rhs), (i64 imm:$imm), i64:$src, i64:$src2))]>;
 868:   def Select_64_32 : Pseudo<(outs GPR32:$dst),
 869:                       (ins GPR:$lhs, GPR:$rhs, i64imm:$imm, GPR32:$src, GPR32:$src2),
 870:                       "# Select PSEUDO $dst = $lhs $imm $rhs ? $src : $src2",
 871:                       [(set i32:$dst,
 872:                        (BPFselectcc i64:$lhs, i64:$rhs, (i64 imm:$imm), i32:$src, i32:$src2))]>;
 873:   def Select_Ri_64_32 : Pseudo<(outs GPR32:$dst),
 874:                       (ins GPR:$lhs, i64imm:$rhs, i64imm:$imm, GPR32:$src, GPR32:$src2),
 875:                       "# Select PSEUDO $dst = $lhs $imm $rhs ? $src : $src2",
 876:                       [(set i32:$dst,
 877:                        (BPFselectcc i64:$lhs, (i64immSExt32:$rhs), (i64 imm:$imm), i32:$src, i32:$src2))]>;
 878:   def Select_32 : Pseudo<(outs GPR32:$dst),
 879:                       (ins GPR32:$lhs, GPR32:$rhs, i32imm:$imm, GPR32:$src, GPR32:$src2),
 880:                       "# Select PSEUDO $dst = $lhs $imm $rhs ? $src : $src2",
 881:                       [(set i32:$dst,
 882:                        (BPFselectcc i32:$lhs, i32:$rhs, (i32 imm:$imm), i32:$src, i32:$src2))]>;
```

- EN: This range defines declarative TableGen records such as Select_64_32, Select_Ri_64_32, Select_32, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Select_64_32, Select_Ri_64_32, Select_32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 883-900

```tablegen
 883:   def Select_Ri_32 : Pseudo<(outs GPR32:$dst),
 884:                       (ins GPR32:$lhs, i32imm:$rhs, i32imm:$imm, GPR32:$src, GPR32:$src2),
 885:                       "# Select PSEUDO $dst = $lhs $imm $rhs ? $src : $src2",
 886:                       [(set i32:$dst,
 887:                        (BPFselectcc i32:$lhs, (i32immSExt32:$rhs), (i32 imm:$imm), i32:$src, i32:$src2))]>;
 888:   def Select_32_64 : Pseudo<(outs GPR:$dst),
 889:                       (ins GPR32:$lhs, GPR32:$rhs, i32imm:$imm, GPR:$src, GPR:$src2),
 890:                       "# Select PSEUDO $dst = $lhs $imm $rhs ? $src : $src2",
 891:                       [(set i64:$dst,
 892:                        (BPFselectcc i32:$lhs, i32:$rhs, (i32 imm:$imm), i64:$src, i64:$src2))]>;
 893:   def Select_Ri_32_64 : Pseudo<(outs GPR:$dst),
 894:                       (ins GPR32:$lhs, i32imm:$rhs, i32imm:$imm, GPR:$src, GPR:$src2),
 895:                       "# Select PSEUDO $dst = $lhs $imm $rhs ? $src : $src2",
 896:                       [(set i64:$dst,
 897:                        (BPFselectcc i32:$lhs, (i32immSExt32:$rhs), (i32 imm:$imm), i64:$src, i64:$src2))]>;
 898: }
 899: 
 900: // load 64-bit global addr into register
```

- EN: This range defines declarative TableGen records such as Select_Ri_32, Select_32_64, Select_Ri_32_64, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Select_Ri_32, Select_32_64, Select_Ri_32_64 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 901-918

```tablegen
 901: def : Pat<(BPFWrapper tconstpool:$in), (LD_imm64 tconstpool:$in)>;
 902: def : Pat<(BPFWrapper tjumptable:$in), (LD_imm64 tjumptable:$in)>;
 903: 
 904: // 0xffffFFFF doesn't fit into simm32, optimize common case
 905: def : Pat<(i64 (and (i64 GPR:$src), 0xffffFFFF)),
 906:           (SRL_ri (SLL_ri (i64 GPR:$src), 32), 32)>;
 907: 
 908: // Calls
 909: def : Pat<(BPFcall tglobaladdr:$dst), (JAL tglobaladdr:$dst)>;
 910: def : Pat<(BPFcall texternalsym:$dst), (JAL texternalsym:$dst)>;
 911: def : Pat<(BPFcall imm:$dst), (JAL imm:$dst)>;
 912: def : Pat<(BPFcall GPR:$dst), (JALX GPR:$dst)>;
 913: 
 914: // Loads
 915: let Predicates = [BPFNoALU32] in {
 916:   def : Pat<(i64 (extloadi8  ADDRri:$src)), (i64 (LDB ADDRri:$src))>;
 917:   def : Pat<(i64 (extloadi16 ADDRri:$src)), (i64 (LDH ADDRri:$src))>;
 918:   def : Pat<(i64 (extloadi32 ADDRri:$src)), (i64 (LDW ADDRri:$src))>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 919-936

```tablegen
 919: }
 920: 
 921: // Atomic add, and, or, xor
 922: class ATOMIC_NOFETCH<BPFWidthModifer SizeOp, string OpType, RegisterClass RegTp,
 923:                      BPFArithOp Opc, string Opstr>
 924:     : TYPE_LD_ST<BPF_ATOMIC.Value, SizeOp.Value,
 925:                  (outs RegTp:$dst),
 926:                  (ins MEMri:$addr, RegTp:$val),
 927:                  "lock *(" #OpType# " *)($addr) " #Opstr# "= $val",
 928:                  []> {
 929:   bits<4> dst;
 930:   bits<20> addr;
 931: 
 932:   let Inst{51-48} = addr{19-16}; // base reg
 933:   let Inst{55-52} = dst;
 934:   let Inst{47-32} = addr{15-0}; // offset
 935:   let Inst{7-4} = Opc.Value;
 936:   let BPFClass = BPF_STX;
```

- EN: This range defines declarative TableGen records such as ATOMIC_NOFETCH, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ATOMIC_NOFETCH 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 937-954

```tablegen
 937: }
 938: 
 939: let Constraints = "$dst = $val" in {
 940:   let Predicates = [BPFHasALU32], DecoderNamespace = "BPFALU32" in {
 941:     def XADDW32 : ATOMIC_NOFETCH<BPF_W, "u32", GPR32, BPF_ADD, "+">;
 942:     def XANDW32 : ATOMIC_NOFETCH<BPF_W, "u32", GPR32, BPF_AND, "&">;
 943:     def XORW32  : ATOMIC_NOFETCH<BPF_W, "u32", GPR32, BPF_OR, "|">;
 944:     def XXORW32 : ATOMIC_NOFETCH<BPF_W, "u32", GPR32, BPF_XOR, "^">;
 945:   }
 946:   def XADDW  : ATOMIC_NOFETCH<BPF_W,  "u32", GPR, BPF_ADD, "+">;
 947:   def XADDD  : ATOMIC_NOFETCH<BPF_DW, "u64", GPR, BPF_ADD, "+">;
 948:   def XANDD  : ATOMIC_NOFETCH<BPF_DW, "u64", GPR, BPF_AND, "&">;
 949:   def XORD   : ATOMIC_NOFETCH<BPF_DW, "u64", GPR, BPF_OR, "|">;
 950:   def XXORD  : ATOMIC_NOFETCH<BPF_DW, "u64", GPR, BPF_XOR, "^">;
 951: }
 952: 
 953: let Predicates = [BPFNoALU32] in {
 954:   def : Pat<(atomic_load_add_i32 ADDRri:$addr, GPR:$val),
```

- EN: This range defines declarative TableGen records such as XADDW32, XANDW32, XORW32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 XADDW32, XANDW32, XORW32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 955-972

```tablegen
 955:             (XADDW ADDRri:$addr, GPR:$val)>;
 956:   def : Pat<(atomic_load_add_i64 ADDRri:$addr, GPR:$val),
 957:             (XADDD ADDRri:$addr, GPR:$val)>;
 958: }
 959: 
 960: // Atomic Fetch-and-<add, and, or, xor> operations
 961: class XFALU64<BPFWidthModifer SizeOp, BPFArithOp Opc, string OpcodeStr, string OpcStr>
 962:     : TYPE_LD_ST<BPF_ATOMIC.Value, SizeOp.Value,
 963:                  (outs GPR:$dst),
 964:                  (ins MEMri:$addr, GPR:$val),
 965:                  "$dst = atomic_fetch_"#OpcStr#"(("#OpcodeStr#" *)($addr), $val)",
 966:                  []> {
 967:   bits<4> dst;
 968:   bits<20> addr;
 969: 
 970:   let Inst{51-48} = addr{19-16}; // base reg
 971:   let Inst{55-52} = dst;
 972:   let Inst{47-32} = addr{15-0}; // offset
```

- EN: This range defines declarative TableGen records such as XFALU64, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 XFALU64 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 973-990

```tablegen
 973:   let Inst{7-4} = Opc.Value;
 974:   let Inst{3-0} = BPF_FETCH.Value;
 975:   let BPFClass = BPF_STX;
 976: }
 977: 
 978: class XFALU32<BPFWidthModifer SizeOp, BPFArithOp Opc, string OpcodeStr, string OpcStr>
 979:     : TYPE_LD_ST<BPF_ATOMIC.Value, SizeOp.Value,
 980:                  (outs GPR32:$dst),
 981:                  (ins MEMri:$addr, GPR32:$val),
 982:                  "$dst = atomic_fetch_"#OpcStr#"(("#OpcodeStr#" *)($addr), $val)",
 983:                  []> {
 984:   bits<4> dst;
 985:   bits<20> addr;
 986: 
 987:   let Inst{51-48} = addr{19-16}; // base reg
 988:   let Inst{55-52} = dst;
 989:   let Inst{47-32} = addr{15-0}; // offset
 990:   let Inst{7-4} = Opc.Value;
```

- EN: This range defines declarative TableGen records such as XFALU32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 XFALU32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 991-1008

```tablegen
 991:   let Inst{3-0} = BPF_FETCH.Value;
 992:   let BPFClass = BPF_STX;
 993: }
 994: 
 995: let Constraints = "$dst = $val" in {
 996:   let Predicates = [BPFHasALU32], DecoderNamespace = "BPFALU32" in {
 997:     def XFADDW32 : XFALU32<BPF_W, BPF_ADD, "u32", "add">;
 998:     def XFANDW32 : XFALU32<BPF_W, BPF_AND, "u32", "and">;
 999:     def XFORW32  : XFALU32<BPF_W, BPF_OR,  "u32", "or">;
1000:     def XFXORW32 : XFALU32<BPF_W, BPF_XOR, "u32", "xor">;
1001:   }
1002: 
1003:   let Predicates = [BPFHasALU32] in {
1004:     def XFADDD : XFALU64<BPF_DW, BPF_ADD, "u64", "add">;
1005:   }
1006:   def XFANDD : XFALU64<BPF_DW, BPF_AND, "u64", "and">;
1007:   def XFORD  : XFALU64<BPF_DW, BPF_OR,  "u64", "or">;
1008:   def XFXORD : XFALU64<BPF_DW, BPF_XOR, "u64", "xor">;
```

- EN: This range defines declarative TableGen records such as XFADDW32, XFANDW32, XFORW32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 XFADDW32, XFANDW32, XFORW32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1009-1026

```tablegen
1009: }
1010: 
1011: let Predicates = [BPFHasALU32] in {
1012:     foreach P = [// add
1013:                  [atomic_load_add_i32_monotonic,  XADDW32],
1014:                  [atomic_load_add_i32_acquire,   XFADDW32],
1015:                  [atomic_load_add_i32_release,   XFADDW32],
1016:                  [atomic_load_add_i32_acq_rel,   XFADDW32],
1017:                  [atomic_load_add_i32_seq_cst,   XFADDW32],
1018:                  // and
1019:                  [atomic_load_and_i32_monotonic,  XANDW32],
1020:                  [atomic_load_and_i32_acquire,   XFANDW32],
1021:                  [atomic_load_and_i32_release,   XFANDW32],
1022:                  [atomic_load_and_i32_acq_rel,   XFANDW32],
1023:                  [atomic_load_and_i32_seq_cst,   XFANDW32],
1024:                  // or
1025:                  [atomic_load_or_i32_monotonic,   XORW32],
1026:                  [atomic_load_or_i32_acquire,    XFORW32],
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1027-1044

```tablegen
1027:                  [atomic_load_or_i32_release,    XFORW32],
1028:                  [atomic_load_or_i32_acq_rel,    XFORW32],
1029:                  [atomic_load_or_i32_seq_cst,    XFORW32],
1030:                  // xor
1031:                  [atomic_load_xor_i32_monotonic,  XXORW32],
1032:                  [atomic_load_xor_i32_acquire,   XFXORW32],
1033:                  [atomic_load_xor_i32_release,   XFXORW32],
1034:                  [atomic_load_xor_i32_acq_rel,   XFXORW32],
1035:                  [atomic_load_xor_i32_seq_cst,   XFXORW32],
1036:                 ] in {
1037:       def : Pat<(P[0] ADDRri:$addr, GPR32:$val), (P[1]  ADDRri:$addr, GPR32:$val)>;
1038:     }
1039: 
1040:     // atomic_load_sub can be represented as a neg followed
1041:     // by an atomic_load_add.
1042:     foreach P = [[atomic_load_sub_i32_monotonic,  XADDW32],
1043:                  [atomic_load_sub_i32_acquire,   XFADDW32],
1044:                  [atomic_load_sub_i32_release,   XFADDW32],
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1045-1062

```tablegen
1045:                  [atomic_load_sub_i32_acq_rel,   XFADDW32],
1046:                  [atomic_load_sub_i32_seq_cst,   XFADDW32],
1047:                 ] in {
1048:       def : Pat<(P[0] ADDRri:$addr, GPR32:$val), (P[1]  ADDRri:$addr, (NEG_32 GPR32:$val))>;
1049:     }
1050: 
1051:     foreach P = [// add
1052:                  [atomic_load_add_i64_monotonic,  XADDD],
1053:                  [atomic_load_add_i64_acquire,   XFADDD],
1054:                  [atomic_load_add_i64_release,   XFADDD],
1055:                  [atomic_load_add_i64_acq_rel,   XFADDD],
1056:                  [atomic_load_add_i64_seq_cst,   XFADDD],
1057:                 ] in {
1058:       def : Pat<(P[0] ADDRri:$addr, GPR:$val), (P[1]  ADDRri:$addr, GPR:$val)>;
1059:     }
1060: }
1061: 
1062: foreach P = [[atomic_load_sub_i64_monotonic,  XADDD],
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1063-1080

```tablegen
1063:              [atomic_load_sub_i64_acquire,   XFADDD],
1064:              [atomic_load_sub_i64_release,   XFADDD],
1065:              [atomic_load_sub_i64_acq_rel,   XFADDD],
1066:              [atomic_load_sub_i64_seq_cst,   XFADDD],
1067:             ] in {
1068:   def : Pat<(P[0] ADDRri:$addr, GPR:$val), (P[1]  ADDRri:$addr, (NEG_64 GPR:$val))>;
1069: }
1070: 
1071: // Borrow the idea from X86InstrFragments.td
1072: class binop_no_use<SDPatternOperator operator>
1073:       : PatFrag<(ops node:$A, node:$B),
1074:                 (operator node:$A, node:$B),
1075:                 [{ return SDValue(N, 0).use_empty(); }]>;
1076: 
1077: class binop_has_use<SDPatternOperator operator>
1078:       : PatFrag<(ops node:$A, node:$B),
1079:                 (operator node:$A, node:$B),
1080:                 [{ return !SDValue(N, 0).use_empty(); }]>;
```

- EN: This range defines declarative TableGen records such as binop_no_use, binop_has_use, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 binop_no_use, binop_has_use 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1081-1098

```tablegen
1081: 
1082: foreach op = [add, and, or, xor] in {
1083: def atomic_load_ # op # _i64_monotonic_nu:
1084:     binop_no_use <!cast<SDPatternOperator>("atomic_load_"#op# _i64_monotonic)>;
1085: def atomic_load_ # op # _i64_monotonic_hu:
1086:     binop_has_use<!cast<SDPatternOperator>("atomic_load_"#op# _i64_monotonic)>;
1087: }
1088: 
1089: foreach P = [// and
1090:              [atomic_load_and_i64_monotonic_nu, XANDD],
1091:              [atomic_load_and_i64_monotonic_hu, XFANDD],
1092:              [atomic_load_and_i64_acquire,   XFANDD],
1093:              [atomic_load_and_i64_release,   XFANDD],
1094:              [atomic_load_and_i64_acq_rel,   XFANDD],
1095:              [atomic_load_and_i64_seq_cst,   XFANDD],
1096:              // or
1097:              [atomic_load_or_i64_monotonic_nu, XORD],
1098:              [atomic_load_or_i64_monotonic_hu, XFORD],
```

- EN: This range defines declarative TableGen records such as atomic_load_, atomic_load_, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 atomic_load_, atomic_load_ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1099-1116

```tablegen
1099:              [atomic_load_or_i64_acquire,    XFORD],
1100:              [atomic_load_or_i64_release,    XFORD],
1101:              [atomic_load_or_i64_acq_rel,    XFORD],
1102:              [atomic_load_or_i64_seq_cst,    XFORD],
1103:              // xor
1104:              [atomic_load_xor_i64_monotonic_nu, XXORD],
1105:              [atomic_load_xor_i64_monotonic_hu, XFXORD],
1106:              [atomic_load_xor_i64_acquire,   XFXORD],
1107:              [atomic_load_xor_i64_release,   XFXORD],
1108:              [atomic_load_xor_i64_acq_rel,   XFXORD],
1109:              [atomic_load_xor_i64_seq_cst,   XFXORD],
1110:             ] in {
1111:   def : Pat<(P[0] ADDRri:$addr, GPR:$val), (P[1]  ADDRri:$addr, GPR:$val)>;
1112: }
1113: 
1114: // Atomic Exchange
1115: class XCHG<BPFWidthModifer SizeOp, string OpcodeStr, PatFrag OpNode>
1116:     : TYPE_LD_ST<BPF_ATOMIC.Value, SizeOp.Value,
```

- EN: This range defines declarative TableGen records such as XCHG, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 XCHG 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1117-1134

```tablegen
1117:                  (outs GPR:$dst),
1118:                  (ins MEMri:$addr, GPR:$val),
1119:                  "$dst = xchg_"#OpcodeStr#"($addr, $val)",
1120:                  [(set GPR:$dst, (OpNode ADDRri:$addr,GPR:$val))]> {
1121:   bits<4> dst;
1122:   bits<20> addr;
1123: 
1124:   let Inst{51-48} = addr{19-16}; // base reg
1125:   let Inst{55-52} = dst;
1126:   let Inst{47-32} = addr{15-0}; // offset
1127:   let Inst{7-4} = BPF_XCHG.Value;
1128:   let Inst{3-0} = BPF_FETCH.Value;
1129:   let BPFClass = BPF_STX;
1130: }
1131: 
1132: class XCHG32<BPFWidthModifer SizeOp, string OpcodeStr, PatFrag OpNode>
1133:     : TYPE_LD_ST<BPF_ATOMIC.Value, SizeOp.Value,
1134:                  (outs GPR32:$dst),
```

- EN: This range defines declarative TableGen records such as XCHG32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 XCHG32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1135-1152

```tablegen
1135:                  (ins MEMri:$addr, GPR32:$val),
1136:                  "$dst = xchg32_"#OpcodeStr#"($addr, $val)",
1137:                  [(set GPR32:$dst, (OpNode ADDRri:$addr,GPR32:$val))]> {
1138:   bits<4> dst;
1139:   bits<20> addr;
1140: 
1141:   let Inst{51-48} = addr{19-16}; // base reg
1142:   let Inst{55-52} = dst;
1143:   let Inst{47-32} = addr{15-0}; // offset
1144:   let Inst{7-4} = BPF_XCHG.Value;
1145:   let Inst{3-0} = BPF_FETCH.Value;
1146:   let BPFClass = BPF_STX;
1147: }
1148: 
1149: let Constraints = "$dst = $val" in {
1150:   let Predicates = [BPFHasALU32], DecoderNamespace = "BPFALU32" in {
1151:     def XCHGW32 : XCHG32<BPF_W, "32", atomic_swap_i32>;
1152:   }
```

- EN: This range defines declarative TableGen records such as XCHGW32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 XCHGW32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1153-1170

```tablegen
1153: 
1154:   def XCHGD : XCHG<BPF_DW, "64", atomic_swap_i64>;
1155: }
1156: 
1157: // Compare-And-Exchange
1158: class CMPXCHG<BPFWidthModifer SizeOp, string OpcodeStr, PatFrag OpNode>
1159:     : TYPE_LD_ST<BPF_ATOMIC.Value, SizeOp.Value,
1160:                  (outs),
1161:                  (ins MEMri:$addr, GPR:$new),
1162:                  "r0 = cmpxchg_"#OpcodeStr#"($addr, r0, $new)",
1163:                  [(set R0, (OpNode ADDRri:$addr, R0, GPR:$new))]> {
1164:   bits<4> new;
1165:   bits<20> addr;
1166: 
1167:   let Inst{51-48} = addr{19-16}; // base reg
1168:   let Inst{55-52} = new;
1169:   let Inst{47-32} = addr{15-0}; // offset
1170:   let Inst{7-4} = BPF_CMPXCHG.Value;
```

- EN: This range defines declarative TableGen records such as XCHGD, CMPXCHG, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 XCHGD, CMPXCHG 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1171-1188

```tablegen
1171:   let Inst{3-0} = BPF_FETCH.Value;
1172:   let BPFClass = BPF_STX;
1173: }
1174: 
1175: class CMPXCHG32<BPFWidthModifer SizeOp, string OpcodeStr, PatFrag OpNode>
1176:     : TYPE_LD_ST<BPF_ATOMIC.Value, SizeOp.Value,
1177:                  (outs),
1178:                  (ins MEMri:$addr, GPR32:$new),
1179:                  "w0 = cmpxchg32_"#OpcodeStr#"($addr, w0, $new)",
1180:                  [(set W0, (OpNode ADDRri:$addr, W0, GPR32:$new))]> {
1181:   bits<4> new;
1182:   bits<20> addr;
1183: 
1184:   let Inst{51-48} = addr{19-16}; // base reg
1185:   let Inst{55-52} = new;
1186:   let Inst{47-32} = addr{15-0}; // offset
1187:   let Inst{7-4} = BPF_CMPXCHG.Value;
1188:   let Inst{3-0} = BPF_FETCH.Value;
```

- EN: This range defines declarative TableGen records such as CMPXCHG32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CMPXCHG32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1189-1206

```tablegen
1189:   let BPFClass = BPF_STX;
1190: }
1191: 
1192: let Predicates = [BPFHasALU32], Defs = [W0], Uses = [W0],
1193:     DecoderNamespace = "BPFALU32" in {
1194:   def CMPXCHGW32 : CMPXCHG32<BPF_W, "32", atomic_cmp_swap_i32>;
1195: }
1196: 
1197: let Defs = [R0], Uses = [R0] in {
1198:   def CMPXCHGD : CMPXCHG<BPF_DW, "64", atomic_cmp_swap_i64>;
1199: }
1200: 
1201: // bswap16, bswap32, bswap64
1202: class BSWAP<BPFOpClass Class, bits<32> SizeOp, string OpcodeStr, BPFSrcType SrcType, list<dag> Pattern>
1203:     : TYPE_ALU_JMP<BPF_END.Value, SrcType.Value,
1204:                    (outs GPR:$dst),
1205:                    (ins GPR:$src),
1206:                    "$dst = "#OpcodeStr#" $src",
```

- EN: This range defines declarative TableGen records such as CMPXCHGW32, CMPXCHGD, BSWAP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 CMPXCHGW32, CMPXCHGD, BSWAP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1207-1224

```tablegen
1207:                    Pattern> {
1208:   bits<4> dst;
1209: 
1210:   let Inst{51-48} = dst;
1211:   let Inst{31-0} = SizeOp;
1212:   let BPFClass = Class;
1213: }
1214: 
1215: 
1216: let Constraints = "$dst = $src" in {
1217:   let Predicates = [BPFHasBswap] in {
1218:     def BSWAP16 : BSWAP<BPF_ALU64, 16, "bswap16", BPF_TO_LE, [(set GPR:$dst, (srl (bswap GPR:$src), (i64 48)))]>;
1219:     def BSWAP32 : BSWAP<BPF_ALU64, 32, "bswap32", BPF_TO_LE, [(set GPR:$dst, (srl (bswap GPR:$src), (i64 32)))]>;
1220:     def BSWAP64 : BSWAP<BPF_ALU64, 64, "bswap64", BPF_TO_LE, [(set GPR:$dst, (bswap GPR:$src))]>;
1221:   }
1222: 
1223:   let Predicates = [BPFNoBswap] in {
1224:     let Predicates = [BPFIsLittleEndian] in {
```

- EN: This range defines declarative TableGen records such as BSWAP16, BSWAP32, BSWAP64, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BSWAP16, BSWAP32, BSWAP64 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1225-1242

```tablegen
1225:         def BE16 : BSWAP<BPF_ALU, 16, "be16", BPF_TO_BE, [(set GPR:$dst, (srl (bswap GPR:$src), (i64 48)))]>;
1226:         def BE32 : BSWAP<BPF_ALU, 32, "be32", BPF_TO_BE, [(set GPR:$dst, (srl (bswap GPR:$src), (i64 32)))]>;
1227:         def BE64 : BSWAP<BPF_ALU, 64, "be64", BPF_TO_BE, [(set GPR:$dst, (bswap GPR:$src))]>;
1228:     }
1229:     let Predicates = [BPFIsBigEndian] in {
1230:         def LE16 : BSWAP<BPF_ALU, 16, "le16", BPF_TO_LE, [(set GPR:$dst, (srl (bswap GPR:$src), (i64 48)))]>;
1231:         def LE32 : BSWAP<BPF_ALU, 32, "le32", BPF_TO_LE, [(set GPR:$dst, (srl (bswap GPR:$src), (i64 32)))]>;
1232:         def LE64 : BSWAP<BPF_ALU, 64, "le64", BPF_TO_LE, [(set GPR:$dst, (bswap GPR:$src))]>;
1233:     }
1234:   }
1235: }
1236: 
1237: let Defs = [R0, R1, R2, R3, R4, R5], Uses = [R6], hasSideEffects = 1,
1238:     hasExtraDefRegAllocReq = 1, hasExtraSrcRegAllocReq = 1, mayLoad = 1 in {
1239: class LOAD_ABS<BPFWidthModifer SizeOp, string OpcodeStr, Intrinsic OpNode>
1240:     : TYPE_LD_ST<BPF_ABS.Value, SizeOp.Value,
1241:                  (outs), (ins i64imm:$imm),
1242:                  "r0 = *("#OpcodeStr#" *)skb[$imm]",
```

- EN: This range defines declarative TableGen records such as BE16, BE32, BE64, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BE16, BE32, BE64 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1243-1260

```tablegen
1243:                  [(set R0, (OpNode R6, i64immSExt32:$imm))]> {
1244:   bits<32> imm;
1245: 
1246:   let Inst{31-0} = imm;
1247:   let BPFClass = BPF_LD;
1248: }
1249: 
1250: class LOAD_IND<BPFWidthModifer SizeOp, string OpcodeStr, Intrinsic OpNode>
1251:     : TYPE_LD_ST<BPF_IND.Value, SizeOp.Value,
1252:                  (outs), (ins GPR:$val),
1253:                  "r0 = *("#OpcodeStr#" *)skb[$val]",
1254:                  [(set R0, (OpNode R6, GPR:$val))]> {
1255:   bits<4> val;
1256: 
1257:   let Inst{55-52} = val;
1258:   let BPFClass = BPF_LD;
1259: }
1260: }
```

- EN: This range defines declarative TableGen records such as LOAD_IND, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LOAD_IND 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1261-1278

```tablegen
1261: 
1262: def LD_ABS_B : LOAD_ABS<BPF_B, "u8", int_bpf_load_byte>;
1263: def LD_ABS_H : LOAD_ABS<BPF_H, "u16", int_bpf_load_half>;
1264: def LD_ABS_W : LOAD_ABS<BPF_W, "u32", int_bpf_load_word>;
1265: 
1266: def LD_IND_B : LOAD_IND<BPF_B, "u8", int_bpf_load_byte>;
1267: def LD_IND_H : LOAD_IND<BPF_H, "u16", int_bpf_load_half>;
1268: def LD_IND_W : LOAD_IND<BPF_W, "u32", int_bpf_load_word>;
1269: 
1270: let isCodeGenOnly = 1 in {
1271:   def MOV_32_64 : ALU_RR<BPF_ALU, BPF_MOV, 0,
1272:                          (outs GPR:$dst), (ins GPR32:$src),
1273:                          "$dst = $src", []>;
1274: }
1275: 
1276: let Predicates = [BPFNoMovsx] in {
1277:   def : Pat<(i64 (sext GPR32:$src)),
1278:             (SRA_ri (SLL_ri (MOV_32_64 GPR32:$src), 32), 32)>;
```

- EN: This range defines declarative TableGen records such as LD_ABS_B, LD_ABS_H, LD_ABS_W, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 LD_ABS_B, LD_ABS_H, LD_ABS_W 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1279-1296

```tablegen
1279: }
1280: 
1281: let Predicates = [BPFHasMovsx] in {
1282:   def : Pat<(i64 (sext GPR32:$src)),
1283:             (MOVSX_rr_32 (MOV_32_64 GPR32:$src))>;
1284: }
1285: 
1286: def : Pat<(i64 (zext GPR32:$src)), (MOV_32_64 GPR32:$src)>;
1287: 
1288: // For i64 -> i32 truncation, use the 32-bit subregister directly.
1289: def : Pat<(i32 (trunc GPR:$src)),
1290:           (i32 (EXTRACT_SUBREG GPR:$src, sub_32))>;
1291: 
1292: // For i32 -> i64 anyext, we don't care about the high bits.
1293: def : Pat<(i64 (anyext GPR32:$src)),
1294:           (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GPR32:$src, sub_32)>;
1295: 
1296: class STORE32<BPFWidthModifer SizeOp, string OpcodeStr, list<dag> Pattern>
```

- EN: This range defines declarative TableGen records such as STORE32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 STORE32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1297-1314

```tablegen
1297:     : TYPE_LD_ST<BPF_MEM.Value, SizeOp.Value,
1298:                  (outs),
1299:                  (ins GPR32:$src, MEMri:$addr),
1300:                  "*("#OpcodeStr#" *)($addr) = $src",
1301:                  Pattern> {
1302:   bits<4> src;
1303:   bits<20> addr;
1304: 
1305:   let Inst{51-48} = addr{19-16}; // base reg
1306:   let Inst{55-52} = src;
1307:   let Inst{47-32} = addr{15-0}; // offset
1308:   let BPFClass = BPF_STX;
1309: }
1310: 
1311: class STOREi32<BPFWidthModifer Opc, string OpcodeStr, PatFrag OpNode>
1312:     : STORE32<Opc, OpcodeStr, [(OpNode GPR32:$src, ADDRri:$addr)]>;
1313: 
1314: class STORE_RELEASEi32<BPFWidthModifer Opc, string OpcodeStr>
```

- EN: This range defines declarative TableGen records such as STOREi32, STORE_RELEASEi32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 STOREi32, STORE_RELEASEi32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1315-1332

```tablegen
1315:     : STORE_RELEASE<Opc, OpcodeStr, GPR32>;
1316: 
1317: let Predicates = [BPFHasALU32], DecoderNamespace = "BPFALU32" in {
1318:   def STW32 : STOREi32<BPF_W, "u32", store>;
1319:   def STH32 : STOREi32<BPF_H, "u16", truncstorei16>;
1320:   def STB32 : STOREi32<BPF_B, "u8", truncstorei8>;
1321: 
1322:   let Predicates = [BPFHasLoadAcqStoreRel] in {
1323:     def STWREL32 : STORE_RELEASEi32<BPF_W, "u32">;
1324:     def STHREL32 : STORE_RELEASEi32<BPF_H, "u16">;
1325:     def STBREL32 : STORE_RELEASEi32<BPF_B, "u8">;
1326:   }
1327: }
1328: 
1329: class LOAD32<BPFWidthModifer SizeOp, BPFModeModifer ModOp, string OpcodeStr, list<dag> Pattern>
1330:     : TYPE_LD_ST<ModOp.Value, SizeOp.Value,
1331:                 (outs GPR32:$dst),
1332:                 (ins MEMri:$addr),
```

- EN: This range defines declarative TableGen records such as STW32, STH32, STB32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 STW32, STH32, STB32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1333-1350

```tablegen
1333:                 "$dst = *("#OpcodeStr#" *)($addr)",
1334:                 Pattern> {
1335:   bits<4> dst;
1336:   bits<20> addr;
1337: 
1338:   let Inst{51-48} = dst;
1339:   let Inst{55-52} = addr{19-16};
1340:   let Inst{47-32} = addr{15-0};
1341:   let BPFClass = BPF_LDX;
1342: }
1343: 
1344: class LOADi32<BPFWidthModifer SizeOp, BPFModeModifer ModOp, string OpcodeStr, PatFrag OpNode>
1345:     : LOAD32<SizeOp, ModOp, OpcodeStr, [(set i32:$dst, (OpNode ADDRri:$addr))]>;
1346: 
1347: class LOAD_ACQUIREi32<BPFWidthModifer SizeOp, string OpcodeStr>
1348:     : LOAD_ACQUIRE<SizeOp, OpcodeStr, GPR32>;
1349: 
1350: let Predicates = [BPFHasALU32], DecoderNamespace = "BPFALU32" in {
```

- EN: This range defines declarative TableGen records such as LOADi32, LOAD_ACQUIREi32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 LOADi32, LOAD_ACQUIREi32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1351-1368

```tablegen
1351:   def LDW32 : LOADi32<BPF_W, BPF_MEM, "u32", load>;
1352:   def LDH32 : LOADi32<BPF_H, BPF_MEM, "u16", zextloadi16>;
1353:   def LDB32 : LOADi32<BPF_B, BPF_MEM, "u8", zextloadi8>;
1354: 
1355:   let Predicates = [BPFHasLoadAcqStoreRel] in {
1356:     def LDWACQ32 : LOAD_ACQUIREi32<BPF_W, "u32">;
1357:     def LDHACQ32 : LOAD_ACQUIREi32<BPF_H, "u16">;
1358:     def LDBACQ32 : LOAD_ACQUIREi32<BPF_B, "u8">;
1359:   }
1360: }
1361: 
1362: let Predicates = [BPFHasALU32] in {
1363:   def : Pat<(truncstorei8 GPR:$src, ADDRri:$dst),
1364:             (STB32 (EXTRACT_SUBREG GPR:$src, sub_32), ADDRri:$dst)>;
1365:   def : Pat<(truncstorei16 GPR:$src, ADDRri:$dst),
1366:             (STH32 (EXTRACT_SUBREG GPR:$src, sub_32), ADDRri:$dst)>;
1367:   def : Pat<(truncstorei32 GPR:$src, ADDRri:$dst),
1368:             (STW32 (EXTRACT_SUBREG GPR:$src, sub_32), ADDRri:$dst)>;
```

- EN: This range defines declarative TableGen records such as LDW32, LDH32, LDB32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 LDW32, LDH32, LDB32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1369-1386

```tablegen
1369:   def : Pat<(i32 (extloadi8 ADDRri:$src)), (i32 (LDB32 ADDRri:$src))>;
1370:   def : Pat<(i32 (extloadi16 ADDRri:$src)), (i32 (LDH32 ADDRri:$src))>;
1371: 
1372:   let Predicates = [BPFHasLdsx] in {
1373:     def : Pat<(i32 (sextloadi8 ADDRri:$src)), (EXTRACT_SUBREG (LDBSX ADDRri:$src), sub_32)>;
1374:     def : Pat<(i32 (sextloadi16 ADDRri:$src)), (EXTRACT_SUBREG (LDHSX ADDRri:$src), sub_32)>;
1375:   }
1376: 
1377:   def : Pat<(i64 (zextloadi8  ADDRri:$src)),
1378:             (SUBREG_TO_REG (LDB32 ADDRri:$src), sub_32)>;
1379:   def : Pat<(i64 (zextloadi16 ADDRri:$src)),
1380:             (SUBREG_TO_REG (LDH32 ADDRri:$src), sub_32)>;
1381:   def : Pat<(i64 (zextloadi32 ADDRri:$src)),
1382:             (SUBREG_TO_REG (LDW32 ADDRri:$src), sub_32)>;
1383:   def : Pat<(i64 (extloadi8  ADDRri:$src)),
1384:             (SUBREG_TO_REG (LDB32 ADDRri:$src), sub_32)>;
1385:   def : Pat<(i64 (extloadi16 ADDRri:$src)),
1386:             (SUBREG_TO_REG (LDH32 ADDRri:$src), sub_32)>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1387-1404

```tablegen
1387:   def : Pat<(i64 (extloadi32 ADDRri:$src)),
1388:             (SUBREG_TO_REG (LDW32 ADDRri:$src), sub_32)>;
1389: 
1390:   let Predicates = [BPFHasLoadAcqStoreRel] in {
1391:     foreach P = [[relaxed_load<atomic_load_nonext_32>, LDW32],
1392:                  [relaxed_load<atomic_load_azext_16>, LDH32],
1393:                  [relaxed_load<atomic_load_azext_8>, LDB32],
1394:                  [acquiring_load<atomic_load_nonext_32>, LDWACQ32],
1395:                  [acquiring_load<atomic_load_azext_16>, LDHACQ32],
1396:                  [acquiring_load<atomic_load_azext_8>, LDBACQ32],
1397:                 ] in {
1398:       def : Pat<(P[0] ADDRri:$addr), (P[1] ADDRri:$addr)>;
1399:     }
1400:   }
1401: 
1402:   let Predicates = [BPFHasLoadAcqStoreRel] in {
1403:     foreach P = [[relaxed_store<atomic_store_32>, STW32],
1404:                  [relaxed_store<atomic_store_16>, STH32],
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1405-1422

```tablegen
1405:                  [relaxed_store<atomic_store_8>, STB32],
1406:                  [releasing_store<atomic_store_32>, STWREL32],
1407:                  [releasing_store<atomic_store_16>, STHREL32],
1408:                  [releasing_store<atomic_store_8>, STBREL32],
1409:                 ] in {
1410:       def : Pat<(P[0] GPR32:$val, ADDRri:$addr), (P[1] GPR32:$val, ADDRri:$addr)>;
1411:     }
1412:   }
1413: }
1414: 
1415: let usesCustomInserter = 1, isCodeGenOnly = 1 in {
1416:     def MEMCPY : Pseudo<
1417:       (outs),
1418:       (ins GPR:$dst, GPR:$src, i64imm:$len, i64imm:$align, variable_ops),
1419:       "#memcpy dst: $dst, src: $src, len: $len, align: $align",
1420:       [(BPFmemcpy GPR:$dst, GPR:$src, imm:$len, imm:$align)]>;
1421: }
1422: 
```

- EN: This range defines declarative TableGen records such as MEMCPY, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 MEMCPY 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1423-1426

```tablegen
1423: // For GlobalValue and BlockAddress.
1424: let usesCustomInserter = 1, isCodeGenOnly = 1 in {
1425:   def LDIMM64 : Pseudo<(outs GPR:$dst), (ins i64imm:$addr), "", []>;
1426: }
```

- EN: This range defines declarative TableGen records such as LDIMM64, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDIMM64 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- TableGen instruction records / TableGen 指令记录
- Encoding and patterns / 编码与匹配模式
- Register classes / 寄存器类
- CPU feature modelling / CPU 特性建模
- Pseudo-instruction handling / 伪指令处理
- Pattern matching / 模式匹配

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPFInstrFormats.td`
- LLVM subsystems / LLVM 子系统: SelectionDAG, TableGen
- Local companions / 本地配套文件: `BPFInstrInfo.h`, `BPFInstrInfo.cpp`
