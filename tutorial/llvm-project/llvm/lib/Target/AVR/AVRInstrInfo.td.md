# AVRInstrInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRInstrInfo.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file describes the AVR instructions in TableGen format.
- 目的（中文）: 使用 TableGen 定义目标指令，描述操作码、操作数、匹配模式与调度信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- AVRInstrInfo.td - AVR Instruction defs -------------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the AVR instructions in TableGen format.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: include "AVRInstrFormats.td"
  14: 
  15: //===----------------------------------------------------------------------===//
  16: // AVR Type Profiles
  17: //===----------------------------------------------------------------------===//
  18: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。

### Lines 19-36

```tablegen
  19: def SDT_AVRCallSeqStart : SDCallSeqStart<[SDTCisVT<0, i16>, SDTCisVT<1, i16>]>;
  20: def SDT_AVRCallSeqEnd : SDCallSeqEnd<[SDTCisVT<0, i16>, SDTCisVT<1, i16>]>;
  21: def SDT_AVRCall : SDTypeProfile<0, -1, [SDTCisVT<0, iPTR>]>;
  22: def SDT_AVRWrapper : SDTypeProfile<1, 1, [SDTCisSameAs<0, 1>, SDTCisPtrTy<0>]>;
  23: def SDT_AVRBrcond
  24:     : SDTypeProfile<0, 2, [SDTCisVT<0, OtherVT>, SDTCisVT<1, i8>]>;
  25: def SDT_AVRCmp : SDTypeProfile<0, 2, [SDTCisSameAs<0, 1>]>;
  26: def SDT_AVRTst : SDTypeProfile<0, 1, [SDTCisInt<0>]>;
  27: def SDT_AVRSelectCC
  28:     : SDTypeProfile<1, 3,
  29:                     [SDTCisSameAs<0, 1>, SDTCisSameAs<1, 2>, SDTCisVT<3, i8>]>;
  30: 
  31: //===----------------------------------------------------------------------===//
  32: // AVR Specific Node Definitions
  33: //===----------------------------------------------------------------------===//
  34: 
  35: // Return from subroutine.
  36: def AVRretglue : SDNode<"AVRISD::RET_GLUE", SDTNone,
```

- EN: This range defines declarative TableGen records such as SDT_AVRCallSeqStart, SDT_AVRCallSeqEnd, SDT_AVRCall, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SDT_AVRCallSeqStart, SDT_AVRCallSeqEnd, SDT_AVRCall 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-54

```tablegen
  37:                         [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
  38: 
  39: // Return from ISR.
  40: def AVRretiglue : SDNode<"AVRISD::RETI_GLUE", SDTNone,
  41:                          [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
  42: 
  43: 
  44: def AVRcallseq_start : SDNode<"ISD::CALLSEQ_START", SDT_AVRCallSeqStart,
  45:                               [SDNPHasChain, SDNPOutGlue]>;
  46: def AVRcallseq_end : SDNode<"ISD::CALLSEQ_END", SDT_AVRCallSeqEnd,
  47:                             [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
  48: 
  49: // Represents an abstract call instruction,
  50: // which includes a bunch of information.
  51: def AVRcall : SDNode<"AVRISD::CALL", SDT_AVRCall,
  52:                      [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue, SDNPVariadic]>;
  53: 
  54: // A wrapper node for TargetConstantPool,
```

- EN: This range defines declarative TableGen records such as AVRretiglue, AVRcallseq_start, AVRcallseq_end, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 AVRretiglue, AVRcallseq_start, AVRcallseq_end 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55: // TargetExternalSymbol, and TargetGlobalAddress.
  56: def AVRWrapper : SDNode<"AVRISD::WRAPPER", SDT_AVRWrapper>;
  57: 
  58: // AVR conditional branches. Operand 0 is the chain operand, operand 1
  59: // is the block to branch if condition is true, operand 2 is the
  60: // condition code, and operand 3 is the flag operand produced by a CMP
  61: // or TEST instruction.
  62: def AVRbrcond
  63:     : SDNode<"AVRISD::BRCOND", SDT_AVRBrcond, [SDNPHasChain, SDNPInGlue]>;
  64: 
  65: // Compare instruction.
  66: def AVRcmp : SDNode<"AVRISD::CMP", SDT_AVRCmp, [SDNPOutGlue]>;
  67: 
  68: // Compare with carry instruction.
  69: def AVRcmpc : SDNode<"AVRISD::CMPC", SDT_AVRCmp, [SDNPInGlue, SDNPOutGlue]>;
  70: 
  71: // Test for zero or minus instruction.
  72: def AVRtst : SDNode<"AVRISD::TST", SDT_AVRTst, [SDNPOutGlue]>;
```

- EN: This range defines declarative TableGen records such as AVRWrapper, AVRbrcond, AVRcmp, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 AVRWrapper, AVRbrcond, AVRcmp 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-90

```tablegen
  73: 
  74: // Operand 0 and operand 1 are selection variable, operand 2
  75: // is condition code and operand 3 is flag operand.
  76: def AVRselectcc : SDNode<"AVRISD::SELECT_CC", SDT_AVRSelectCC, [SDNPInGlue]>;
  77: 
  78: // Shift nodes.
  79: def AVRlsl : SDNode<"AVRISD::LSL", SDTIntUnaryOp>;       // Logical shift left.
  80: def AVRlsr : SDNode<"AVRISD::LSR", SDTIntUnaryOp>;       // Logical shift right.
  81: def AVRrol : SDNode<"AVRISD::ROL", SDTIntUnaryOp>;       // Bit rotate left.
  82: def AVRror : SDNode<"AVRISD::ROR", SDTIntUnaryOp>;       // Bit rotate right.
  83: def AVRasr : SDNode<"AVRISD::ASR", SDTIntUnaryOp>;       // Arithmetic shift right.
  84: def AVRlslhi : SDNode<"AVRISD::LSLHI", SDTIntUnaryOp>;   // Higher 8-bit of word logical shift left.
  85: def AVRlsrlo : SDNode<"AVRISD::LSRLO", SDTIntUnaryOp>;   // Lower 8-bit of word logical shift right.
  86: def AVRasrlo : SDNode<"AVRISD::ASRLO", SDTIntUnaryOp>;   // Lower 8-bit of word arithmetic shift right.
  87: def AVRlslbn : SDNode<"AVRISD::LSLBN", SDTIntBinOp>;     // Byte logical shift left N bits.
  88: def AVRlsrbn : SDNode<"AVRISD::LSRBN", SDTIntBinOp>;     // Byte logical shift right N bits.
  89: def AVRasrbn : SDNode<"AVRISD::ASRBN", SDTIntBinOp>;     // Byte arithmetic shift right N bits.
  90: def AVRlslwn : SDNode<"AVRISD::LSLWN", SDTIntBinOp>;     // Higher 8-bit of word logical shift left.
```

- EN: This range defines declarative TableGen records such as AVRselectcc, AVRlsl, AVRlsr, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 AVRselectcc, AVRlsl, AVRlsr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 91-108

```tablegen
  91: def AVRlsrwn : SDNode<"AVRISD::LSRWN", SDTIntBinOp>;     // Word logical shift right N bits.
  92: def AVRasrwn : SDNode<"AVRISD::ASRWN", SDTIntBinOp>;     // Word arithmetic shift right N bits.
  93: def AVRlslw : SDNode<"AVRISD::LSLW", SDTIntShiftPairOp>; // Wide logical shift left.
  94: def AVRlsrw : SDNode<"AVRISD::LSRW", SDTIntShiftPairOp>; // Wide logical shift right.
  95: def AVRasrw : SDNode<"AVRISD::ASRW", SDTIntShiftPairOp>; // Wide arithmetic shift right.
  96: 
  97: // Pseudo shift nodes for non-constant shift amounts.
  98: def AVRlslLoop : SDNode<"AVRISD::LSLLOOP", SDTIntShiftOp>;
  99: def AVRlsrLoop : SDNode<"AVRISD::LSRLOOP", SDTIntShiftOp>;
 100: def AVRrolLoop : SDNode<"AVRISD::ROLLOOP", SDTIntShiftOp>;
 101: def AVRrorLoop : SDNode<"AVRISD::RORLOOP", SDTIntShiftOp>;
 102: def AVRasrLoop : SDNode<"AVRISD::ASRLOOP", SDTIntShiftOp>;
 103: 
 104: // SWAP node.
 105: def AVRSwap : SDNode<"AVRISD::SWAP", SDTIntUnaryOp>;
 106: 
 107: //===----------------------------------------------------------------------===//
 108: // AVR Operands, Complex Patterns and Transformations Definitions.
```

- EN: This range defines declarative TableGen records such as AVRlsrwn, AVRasrwn, AVRlslw, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 AVRlsrwn, AVRasrwn, AVRlslw 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 109-126

```tablegen
 109: //===----------------------------------------------------------------------===//
 110: 
 111: def imm8_neg_XFORM : SDNodeXForm<imm, [{
 112:   return CurDAG->getTargetConstant(-N->getAPIntValue(), SDLoc(N), MVT::i8);
 113: }]>;
 114: 
 115: def imm16_neg_XFORM : SDNodeXForm<imm, [{
 116:   return CurDAG->getTargetConstant(-N->getAPIntValue(), SDLoc(N), MVT::i16);
 117: }]>;
 118: 
 119: def imm0_63_neg : PatLeaf<(imm), [{
 120:   int64_t val = -N->getSExtValue();
 121:   return val >= 0 && val < 64;
 122: }], imm16_neg_XFORM>;
 123: 
 124: def uimm6 : PatLeaf<(imm), [{ return isUInt<6>(N->getZExtValue()); }]>;
 125: 
 126: // imm_com8_XFORM - Return the complement of a imm_com8 value
```

- EN: This range defines declarative TableGen records such as imm8_neg_XFORM, imm16_neg_XFORM, imm0_63_neg, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 imm8_neg_XFORM, imm16_neg_XFORM, imm0_63_neg 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 127-144

```tablegen
 127: def imm_com8_XFORM : SDNodeXForm<imm, [{
 128:   return CurDAG->getTargetConstant(
 129:       ~((uint8_t) N->getZExtValue()), SDLoc(N), MVT::i8);
 130: }]>;
 131: 
 132: // imm_com8 - Match an immediate that is a complement
 133: // of a 8-bit immediate.
 134: // Note: this pattern doesn't require an encoder method and such, as it's
 135: // only used on aliases (Pat<> and InstAlias<>). The actual encoding
 136: // is handled by the destination instructions, which use imm_com8.
 137: def imm_com8_asmoperand : AsmOperandClass { let Name = "ImmCom8"; }
 138: def imm_com8 : Operand<i8> { let ParserMatchClass = imm_com8_asmoperand; }
 139: 
 140: def ioaddr_XFORM : SDNodeXForm<imm, [{
 141:   uint8_t offset = Subtarget->getIORegisterOffset();
 142:   return CurDAG->getTargetConstant(
 143:       uint8_t(N->getZExtValue()) - offset, SDLoc(N), MVT::i8);
 144: }]>;
```

- EN: This range defines declarative TableGen records such as imm_com8_XFORM, imm_com8_asmoperand, imm_com8, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 imm_com8_XFORM, imm_com8_asmoperand, imm_com8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 145-162

```tablegen
 145: 
 146: def iobitpos8_XFORM : SDNodeXForm<imm, [{
 147:   return CurDAG->getTargetConstant(
 148:       Log2_32(uint8_t(N->getZExtValue())), SDLoc(N), MVT::i8);
 149: }]>;
 150: 
 151: def iobitposn8_XFORM : SDNodeXForm<imm, [{
 152:   return CurDAG->getTargetConstant(
 153:       Log2_32(uint8_t(~N->getZExtValue())), SDLoc(N), MVT::i8);
 154: }]>;
 155: 
 156: def ioaddr8 : PatLeaf<(imm), [{
 157:   uint8_t offset = Subtarget->getIORegisterOffset();
 158:   uint64_t val = N->getZExtValue() - offset;
 159:   return val < 0x40;
 160: }], ioaddr_XFORM>;
 161: 
 162: def lowioaddr8 : PatLeaf<(imm), [{
```

- EN: This range defines declarative TableGen records such as iobitpos8_XFORM, iobitposn8_XFORM, ioaddr8, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 iobitpos8_XFORM, iobitposn8_XFORM, ioaddr8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 163-180

```tablegen
 163:   uint8_t offset = Subtarget->getIORegisterOffset();
 164:   uint64_t val = N->getZExtValue() - offset;
 165:   return val < 0x20;
 166: }], ioaddr_XFORM>;
 167: 
 168: def ioaddr16 : PatLeaf<(imm), [{
 169:   uint8_t offset = Subtarget->getIORegisterOffset();
 170:   uint64_t val = N->getZExtValue() - offset;
 171:   return val < 0x3f;
 172: }], ioaddr_XFORM>;
 173: 
 174: def iobitpos8 : PatLeaf<(imm), [{
 175:   return isPowerOf2_32(uint8_t(N->getZExtValue()));
 176: }], iobitpos8_XFORM>;
 177: 
 178: def iobitposn8 : PatLeaf<(imm), [{
 179:   return isPowerOf2_32(uint8_t(~N->getZExtValue()));
 180: }], iobitposn8_XFORM>;
```

- EN: This range defines declarative TableGen records such as ioaddr16, iobitpos8, iobitposn8, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ioaddr16, iobitpos8, iobitposn8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 181-198

```tablegen
 181: 
 182: def MemriAsmOperand : AsmOperandClass {
 183:   let Name = "Memri";
 184:   let ParserMethod = "parseMemriOperand";
 185: }
 186: 
 187: /// Address operand for `reg+imm` used by STD and LDD.
 188: def memri : Operand<iPTR> {
 189:   let MIOperandInfo = (ops PTRDISPREGS, i16imm);
 190: 
 191:   let PrintMethod = "printMemri";
 192:   let EncoderMethod = "encodeMemri";
 193:   let DecoderMethod = "decodeMemri";
 194: 
 195:   let ParserMatchClass = MemriAsmOperand;
 196: }
 197: 
 198: // Address operand for `SP+imm` used by STD{W}SPQRr
```

- EN: This range defines declarative TableGen records such as MemriAsmOperand, memri, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MemriAsmOperand, memri 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-216

```tablegen
 199: def memspi : Operand<iPTR> {
 200:   let MIOperandInfo = (ops GPRSP, i16imm);
 201:   let PrintMethod = "printMemspi";
 202: }
 203: 
 204: def relbrtarget_7 : Operand<OtherVT> {
 205:   let PrintMethod = "printPCRelImm";
 206:   let EncoderMethod = "encodeRelCondBrTarget<AVR::fixup_7_pcrel>";
 207:   let DecoderMethod = "decodeRelCondBrTarget7";
 208: }
 209: 
 210: def brtarget_13 : Operand<OtherVT> {
 211:   let PrintMethod = "printPCRelImm";
 212:   let EncoderMethod = "encodeRelCondBrTarget<AVR::fixup_13_pcrel>";
 213:   let DecoderMethod = "decodeRelCondBrTarget13";
 214: }
 215: 
 216: def rcalltarget_13 : Operand<i16> {
```

- EN: This range defines declarative TableGen records such as memspi, relbrtarget_7, brtarget_13, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 memspi, relbrtarget_7, brtarget_13 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217:   let PrintMethod = "printPCRelImm";
 218:   let EncoderMethod = "encodeRelCondBrTarget<AVR::fixup_13_pcrel>";
 219:   let DecoderMethod = "decodeRelCondBrTarget13";
 220: }
 221: 
 222: // The target of a 22 or 16-bit call/jmp instruction.
 223: def call_target : Operand<iPTR> {
 224:   let EncoderMethod = "encodeCallTarget";
 225:   let DecoderMethod = "decodeCallTarget";
 226: }
 227: 
 228: // A 16-bit address (which can lead to an R_AVR_16 relocation).
 229: def imm16 : Operand<i16> { let EncoderMethod = "encodeImm<AVR::fixup_16, 2>"; }
 230: 
 231: // A 7-bit address (which can lead to an R_AVR_LDS_STS_16 relocation).
 232: def imm7tiny : Operand<i16> {
 233:   let EncoderMethod = "encodeImm<AVR::fixup_lds_sts_16, 0>";
 234: }
```

- EN: This range defines declarative TableGen records such as call_target, imm16, imm7tiny, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 call_target, imm16, imm7tiny 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 235-252

```tablegen
 235: 
 236: /// A 6-bit immediate used in the ADIW/SBIW instructions.
 237: def imm_arith6 : Operand<i16> {
 238:   let EncoderMethod = "encodeImm<AVR::fixup_6_adiw, 0>";
 239: }
 240: 
 241: /// An 8-bit immediate inside an instruction with the same format
 242: /// as the `LDI` instruction (the `FRdK` format).
 243: def imm_ldi8 : Operand<i8> {
 244:   let EncoderMethod = "encodeImm<AVR::fixup_ldi, 0>";
 245: }
 246: 
 247: /// A 5-bit port number used in SBIC and friends (the `FIOBIT` format).
 248: def imm_port5 : Operand<i8> {
 249:   let EncoderMethod = "encodeImm<AVR::fixup_port5, 0>";
 250: }
 251: 
 252: /// A 6-bit port number used in the `IN` instruction and friends (the
```

- EN: This range defines declarative TableGen records such as imm_arith6, imm_ldi8, imm_port5, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 imm_arith6, imm_ldi8, imm_port5 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 253-270

```tablegen
 253: /// `FIORdA` format.
 254: def imm_port6 : Operand<i8> {
 255:   let EncoderMethod = "encodeImm<AVR::fixup_port6, 0>";
 256: }
 257: 
 258: // Addressing mode pattern reg+imm6
 259: let WantsRoot = true in
 260: def addr : ComplexPattern<iPTR, 2, "SelectAddr">;
 261: 
 262: //===----------------------------------------------------------------------===//
 263: // AVR predicates for subtarget features
 264: //===----------------------------------------------------------------------===//
 265: 
 266: def HasSRAM : Predicate<"Subtarget->hasSRAM()">,
 267:               AssemblerPredicate<(all_of FeatureSRAM)>;
 268: 
 269: def HasJMPCALL : Predicate<"Subtarget->hasJMPCALL()">,
 270:                  AssemblerPredicate<(all_of FeatureJMPCALL)>;
```

- EN: This range defines declarative TableGen records such as imm_port6, addr, HasSRAM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 imm_port6, addr, HasSRAM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 271-288

```tablegen
 271: 
 272: def HasIJMPCALL : Predicate<"Subtarget->hasIJMPCALL()">,
 273:                   AssemblerPredicate<(all_of FeatureIJMPCALL)>;
 274: 
 275: def HasEIJMPCALL : Predicate<"Subtarget->hasEIJMPCALL()">,
 276:                    AssemblerPredicate<(all_of FeatureEIJMPCALL)>;
 277: 
 278: def HasADDSUBIW : Predicate<"Subtarget->hasADDSUBIW()">,
 279:                   AssemblerPredicate<(all_of FeatureADDSUBIW)>;
 280: 
 281: def HasSmallStack : Predicate<"Subtarget->HasSmallStack()">,
 282:                     AssemblerPredicate<(all_of FeatureSmallStack)>;
 283: 
 284: def HasMOVW : Predicate<"Subtarget->hasMOVW()">,
 285:               AssemblerPredicate<(all_of FeatureMOVW)>;
 286: 
 287: def HasLPM : Predicate<"Subtarget->hasLPM()">,
 288:              AssemblerPredicate<(all_of FeatureLPM)>;
```

- EN: This range defines declarative TableGen records such as HasIJMPCALL, HasEIJMPCALL, HasADDSUBIW, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasIJMPCALL, HasEIJMPCALL, HasADDSUBIW 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 289-306

```tablegen
 289: 
 290: def HasLPMX : Predicate<"Subtarget->hasLPMX()">,
 291:               AssemblerPredicate<(all_of FeatureLPMX)>;
 292: 
 293: def HasELPM : Predicate<"Subtarget->hasELPM()">,
 294:               AssemblerPredicate<(all_of FeatureELPM)>;
 295: 
 296: def HasELPMX : Predicate<"Subtarget->hasELPMX()">,
 297:                AssemblerPredicate<(all_of FeatureELPMX)>;
 298: 
 299: def HasSPM : Predicate<"Subtarget->hasSPM()">,
 300:              AssemblerPredicate<(all_of FeatureSPM)>;
 301: 
 302: def HasSPMX : Predicate<"Subtarget->hasSPMX()">,
 303:               AssemblerPredicate<(all_of FeatureSPMX)>;
 304: 
 305: def HasDES : Predicate<"Subtarget->hasDES()">,
 306:              AssemblerPredicate<(all_of FeatureDES)>;
```

- EN: This range defines declarative TableGen records such as HasLPMX, HasELPM, HasELPMX, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasLPMX, HasELPM, HasELPMX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 307-324

```tablegen
 307: 
 308: def SupportsRMW : Predicate<"Subtarget->supportsRMW()">,
 309:                   AssemblerPredicate<(all_of FeatureRMW)>;
 310: 
 311: def SupportsMultiplication : Predicate<"Subtarget->supportsMultiplication()">,
 312:                              AssemblerPredicate<(all_of FeatureMultiplication)>;
 313: 
 314: def HasBREAK : Predicate<"Subtarget->hasBREAK()">,
 315:                AssemblerPredicate<(all_of FeatureBREAK)>;
 316: 
 317: def HasTinyEncoding : Predicate<"Subtarget->hasTinyEncoding()">,
 318:                       AssemblerPredicate<(all_of FeatureTinyEncoding)>;
 319: 
 320: def HasNonTinyEncoding : Predicate<"!Subtarget->hasTinyEncoding()">,
 321:                          AssemblerPredicate<(any_of (not FeatureTinyEncoding))>;
 322: 
 323: // AVR specific condition code. These correspond to AVR_*_COND in
 324: // AVRInstrInfo.td. They must be kept in synch.
```

- EN: This range defines declarative TableGen records such as SupportsRMW, SupportsMultiplication, HasBREAK, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SupportsRMW, SupportsMultiplication, HasBREAK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 325-342

```tablegen
 325: def AVR_COND_EQ : PatLeaf<(i8 0)>;
 326: def AVR_COND_NE : PatLeaf<(i8 1)>;
 327: def AVR_COND_GE : PatLeaf<(i8 2)>;
 328: def AVR_COND_LT : PatLeaf<(i8 3)>;
 329: def AVR_COND_SH : PatLeaf<(i8 4)>;
 330: def AVR_COND_LO : PatLeaf<(i8 5)>;
 331: def AVR_COND_MI : PatLeaf<(i8 6)>;
 332: def AVR_COND_PL : PatLeaf<(i8 7)>;
 333: 
 334: //===----------------------------------------------------------------------===//
 335: //===----------------------------------------------------------------------===//
 336: // AVR Instruction list
 337: //===----------------------------------------------------------------------===//
 338: //===----------------------------------------------------------------------===//
 339: 
 340: // ADJCALLSTACKDOWN/UP implicitly use/def SP because they may be expanded into
 341: // a stack adjustment and the codegen must know that they may modify the stack
 342: // pointer before prolog-epilog rewriting occurs.
```

- EN: This range defines declarative TableGen records such as AVR_COND_EQ, AVR_COND_NE, AVR_COND_GE, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 AVR_COND_EQ, AVR_COND_NE, AVR_COND_GE 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 343-360

```tablegen
 343: // Pessimistically assume ADJCALLSTACKDOWN / ADJCALLSTACKUP will become
 344: // sub / add which can clobber SREG.
 345: let Defs = [SP, SREG], Uses = [SP] in {
 346:   def ADJCALLSTACKDOWN : Pseudo<(outs), (ins i16imm:$amt, i16imm:$amt2),
 347:                                 "#ADJCALLSTACKDOWN",
 348:                                 [(AVRcallseq_start timm:$amt, timm:$amt2)]>;
 349: 
 350:   // R31R30 is used to update SP. It is normally free because it is a
 351:   // call-clobbered register but it is necessary to set it as a def as the
 352:   // register allocator might use it in rare cases (for rematerialization, it
 353:   // seems). hasSideEffects needs to be set to true so this instruction isn't
 354:   // considered dead.
 355:   let Defs = [R31R30], hasSideEffects = 1 in
 356:   def ADJCALLSTACKUP : Pseudo<(outs), (ins i16imm:$amt1, i16imm:$amt2),
 357:                               "#ADJCALLSTACKUP",
 358:                               [(AVRcallseq_end timm:$amt1, timm:$amt2)]>;
 359: }
 360: 
```

- EN: This range defines declarative TableGen records such as ADJCALLSTACKDOWN, ADJCALLSTACKUP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADJCALLSTACKDOWN, ADJCALLSTACKUP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 361-378

```tablegen
 361: //===----------------------------------------------------------------------===//
 362: // Addition
 363: //===----------------------------------------------------------------------===//
 364: let hasSideEffects = 0, isCommutable = 1, Constraints = "$src = $rd",
 365:     Defs = [SREG] in {
 366:   // ADD Rd, Rr
 367:   // Adds two 8-bit registers.
 368:   def ADDRdRr : FRdRr<0b0000, 0b11, (outs GPR8:$rd),(ins GPR8:$src, GPR8:$rr),
 369:                       "add\t$rd, $rr",
 370:                       [(set i8:$rd, (add i8:$src, i8:$rr))]>;
 371: 
 372:   // ADDW Rd+1:Rd, Rr+1:Rr
 373:   // Pseudo instruction to add four 8-bit registers as two 16-bit values.
 374:   //
 375:   // Expands to:
 376:   // add Rd,    Rr
 377:   // adc Rd+1, Rr+1
 378:   def ADDWRdRr : Pseudo<(outs DREGS:$rd), (ins DREGS:$src, DREGS:$rr),
```

- EN: This range defines declarative TableGen records such as ADDRdRr, ADDWRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADDRdRr, ADDWRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 379-396

```tablegen
 379:                         "addw\t$rd, $rr",
 380:                         [(set i16:$rd, (add i16:$src, i16:$rr))]>;
 381: 
 382:   // ADC Rd, Rr
 383:   // Adds two 8-bit registers with carry.
 384:   let Uses = [SREG] in
 385:   def ADCRdRr : FRdRr<0b0001, 0b11, (outs GPR8:$rd), (ins GPR8:$src, GPR8:$rr),
 386:                       "adc\t$rd, $rr",
 387:                       [(set i8:$rd, (adde i8:$src, i8:$rr))]>;
 388: 
 389:   // ADCW Rd+1:Rd, Rr+1:Rr
 390:   // Pseudo instruction to add four 8-bit registers as two 16-bit values with
 391:   // carry.
 392:   //
 393:   // Expands to:
 394:   // adc Rd,   Rr
 395:   // adc Rd+1, Rr+1
 396:   let Uses = [SREG] in
```

- EN: This range defines declarative TableGen records such as ADCRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADCRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 397-414

```tablegen
 397:   def ADCWRdRr : Pseudo<(outs DREGS:$rd), (ins DREGS:$src, DREGS:$rr),
 398:                         "adcw\t$rd, $rr",
 399:                         [(set i16:$rd, (adde i16:$src, i16:$rr))]>;
 400: 
 401:   // AIDW Rd, k
 402:   // Adds an immediate 6-bit value K to Rd, placing the result in Rd.
 403:   def ADIWRdK : FWRdK<0b0, (outs IWREGS:$rd), (ins IWREGS :$src, imm_arith6:$k),
 404:                       "adiw\t$rd, $k",
 405:                       [(set i16:$rd, (add i16:$src, uimm6:$k))]>,
 406:                 Requires<[HasADDSUBIW]>;
 407: }
 408: 
 409: //===----------------------------------------------------------------------===//
 410: // Subtraction
 411: //===----------------------------------------------------------------------===//
 412: let hasSideEffects = 0, Constraints = "$rs = $rd", Defs = [SREG] in {
 413:   // SUB Rd, Rr
 414:   // Subtracts the 8-bit value of Rr from Rd and places the value in Rd.
```

- EN: This range defines declarative TableGen records such as ADCWRdRr, ADIWRdK, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADCWRdRr, ADIWRdK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 415-432

```tablegen
 415:   def SUBRdRr : FRdRr<0b0001, 0b10, (outs GPR8:$rd), (ins GPR8:$rs, GPR8:$rr),
 416:                       "sub\t$rd, $rr",
 417:                       [(set i8:$rd, (sub i8:$rs, i8:$rr))]>;
 418: 
 419:   // SUBW Rd+1:Rd, Rr+1:Rr
 420:   // Subtracts two 16-bit values and places the result into Rd.
 421:   //
 422:   // Expands to:
 423:   // sub Rd,   Rr
 424:   // sbc Rd+1, Rr+1
 425:   def SUBWRdRr : Pseudo<(outs DREGS:$rd), (ins DREGS:$rs, DREGS:$rr),
 426:                         "subw\t$rd, $rr",
 427:                         [(set i16:$rd, (sub i16:$rs, i16:$rr))]>;
 428: 
 429:   def SUBIRdK : FRdK<0b0101, (outs LD8:$rd), (ins LD8:$rs, imm_ldi8:$k),
 430:                      "subi\t$rd, $k",
 431:                      [(set i8:$rd, (sub i8:$rs, imm:$k))]>;
 432: 
```

- EN: This range defines declarative TableGen records such as SUBRdRr, SUBWRdRr, SUBIRdK, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SUBRdRr, SUBWRdRr, SUBIRdK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 433-450

```tablegen
 433:   // SUBIW Rd+1:Rd, K+1:K
 434:   //
 435:   // Expands to:
 436:   // subi Rd,   K
 437:   // sbci Rd+1, K+1
 438:   def SUBIWRdK : Pseudo<(outs DLDREGS:$rd), (ins DLDREGS:$rs, i16imm:$rr),
 439:                         "subiw\t$rd, $rr",
 440:                         [(set i16:$rd, (sub i16:$rs, imm:$rr))]>;
 441: 
 442:   def SBIWRdK : FWRdK<0b1, (outs IWREGS:$rd), (ins IWREGS:$rs, imm_arith6:$k),
 443:                       "sbiw\t$rd, $k",
 444:                       [(set i16:$rd, (sub i16:$rs, uimm6:$k))]>,
 445:                 Requires<[HasADDSUBIW]>;
 446: 
 447:   // Subtract with carry operations which must read the carry flag in SREG.
 448:   let Uses = [SREG] in {
 449:     def SBCRdRr : FRdRr<0b0000, 0b10, (outs GPR8:$rd), (ins GPR8:$rs, GPR8:$rr),
 450:                         "sbc\t$rd, $rr",
```

- EN: This range defines declarative TableGen records such as SUBIWRdK, SBIWRdK, SBCRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SUBIWRdK, SBIWRdK, SBCRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 451-468

```tablegen
 451:                         [(set i8:$rd, (sube i8:$rs, i8:$rr))]>;
 452: 
 453:     // SBCW Rd+1:Rd, Rr+1:Rr
 454:     //
 455:     // Expands to:
 456:     // sbc Rd,   Rr
 457:     // sbc Rd+1, Rr+1
 458:     def SBCWRdRr : Pseudo<(outs DREGS:$rd), (ins DREGS:$rs, DREGS:$rr),
 459:                           "sbcw\t$rd, $rr",
 460:                           [(set i16:$rd, (sube i16:$rs, i16:$rr))]>;
 461: 
 462:     def SBCIRdK : FRdK<0b0100, (outs LD8:$rd), (ins LD8:$rs, imm_ldi8:$k),
 463:                        "sbci\t$rd, $k",
 464:                        [(set i8:$rd, (sube i8:$rs, imm:$k))]>;
 465: 
 466:     // SBCIW Rd+1:Rd, K+1:K
 467:     // sbci Rd,   K
 468:     // sbci Rd+1, K+1
```

- EN: This range defines declarative TableGen records such as SBCWRdRr, SBCIRdK, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SBCWRdRr, SBCIRdK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 469-486

```tablegen
 469:     def SBCIWRdK : Pseudo<(outs DLDREGS:$rd), (ins DLDREGS:$rs, i16imm:$rr),
 470:                           "sbciw\t$rd, $rr",
 471:                           [(set i16:$rd, (sube i16:$rs, imm:$rr))]>;
 472:   }
 473: }
 474: 
 475: //===----------------------------------------------------------------------===//
 476: // Increment and Decrement
 477: //===----------------------------------------------------------------------===//
 478: let hasSideEffects = 0, Constraints = "$src = $rd", Defs = [SREG] in {
 479:   def INCRd : FRd<0b1001, 0b0100011, (outs GPR8:$rd), (ins GPR8:$src),
 480:                   "inc\t$rd",
 481:                   [(set i8:$rd, (add i8:$src, 1))]>;
 482: 
 483:   def DECRd : FRd<0b1001, 0b0101010, (outs GPR8:$rd), (ins GPR8:$src),
 484:                   "dec\t$rd",
 485:                   [(set i8:$rd, (add i8:$src, -1))]>;
 486: }
```

- EN: This range defines declarative TableGen records such as SBCIWRdK, INCRd, DECRd, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SBCIWRdK, INCRd, DECRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 487-504

```tablegen
 487: 
 488: //===----------------------------------------------------------------------===//
 489: // Multiplication
 490: //===----------------------------------------------------------------------===//
 491: 
 492: let hasSideEffects = 0, isCommutable = 1, Defs = [R1, R0, SREG] in {
 493:   // MUL Rd, Rr
 494:   // Multiplies Rd by Rr and places the result into R1:R0.
 495:   let usesCustomInserter = 1 in {
 496:     def MULRdRr : FRdRr<0b1001, 0b11, (outs), (ins GPR8:$rd, GPR8:$rr),
 497:                         "mul\t$rd, $rr", []>,
 498:                   Requires<[SupportsMultiplication]>;
 499: 
 500:     def MULSRdRr : FMULSRdRr<(outs), (ins LD8:$rd, LD8:$rr),
 501:                              "muls\t$rd, $rr", []>,
 502:                    Requires<[SupportsMultiplication]>;
 503:   }
 504: 
```

- EN: This range defines declarative TableGen records such as MULRdRr, MULSRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MULRdRr, MULSRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 505-522

```tablegen
 505:   def MULSURdRr : FMULSURdRr<(outs), (ins LD8lo:$rd, LD8lo:$rr),
 506:                              "mulsu\t$rd, $rr", []>,
 507:                   Requires<[SupportsMultiplication]>;
 508: 
 509:   def FMUL : FFMULRdRr<0b01, (outs), (ins LD8lo:$rd, LD8lo:$rr),
 510:                        "fmul\t$rd, $rr", []>,
 511:              Requires<[SupportsMultiplication]>;
 512: 
 513:   def FMULS : FFMULRdRr<0b10, (outs), (ins LD8lo:$rd, LD8lo:$rr),
 514:                         "fmuls\t$rd, $rr", []>,
 515:               Requires<[SupportsMultiplication]>;
 516: 
 517:   def FMULSU : FFMULRdRr<0b11, (outs), (ins LD8lo:$rd, LD8lo:$rr),
 518:                          "fmulsu\t$rd, $rr", []>,
 519:                Requires<[SupportsMultiplication]>;
 520: }
 521: 
 522: // Set hasSideEffects = 1 to prevent it is considered dead.
```

- EN: This range defines declarative TableGen records such as MULSURdRr, FMUL, FMULS, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 MULSURdRr, FMUL, FMULS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 523-540

```tablegen
 523: let hasSideEffects = 1, Defs =
 524:     [R15, R14, R13, R12, R11, R10, R9, R8, R7, R6, R5, R4, R3, R2, R1, R0] in
 525: def DESK : FDES<(outs), (ins i8imm:$k), "des\t$k", []>, Requires<[HasDES]>;
 526: 
 527: //===----------------------------------------------------------------------===//
 528: // Logic
 529: //===----------------------------------------------------------------------===//
 530: let hasSideEffects = 0, Constraints = "$src = $rd", Defs = [SREG] in {
 531:   // Register-Register logic instructions (which have the
 532:   // property of commutativity).
 533:   let isCommutable = 1 in {
 534:     def ANDRdRr : FRdRr<0b0010, 0b00, (outs GPR8:$rd),
 535:                         (ins GPR8:$src, GPR8:$rr), "and\t$rd, $rr",
 536:                         [(set i8:$rd, (and i8:$src, i8:$rr))]>;
 537: 
 538:     // ANDW Rd+1:Rd, Rr+1:Rr
 539:     //
 540:     // Expands to:
```

- EN: This range defines declarative TableGen records such as DESK, ANDRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 DESK, ANDRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 541-558

```tablegen
 541:     // and Rd,   Rr
 542:     // and Rd+1, Rr+1
 543:     def ANDWRdRr : Pseudo<(outs DREGS:$rd), (ins DREGS:$src, DREGS:$rr),
 544:                           "andw\t$rd, $rr",
 545:                           [(set i16:$rd, (and i16:$src, i16:$rr))]>;
 546: 
 547:     def ORRdRr : FRdRr<0b0010, 0b10, (outs GPR8:$rd), (ins GPR8:$src, GPR8:$rr),
 548:                        "or\t$rd, $rr",
 549:                        [(set i8:$rd, (or i8:$src, i8:$rr))]>;
 550: 
 551:     // ORW Rd+1:Rd, Rr+1:Rr
 552:     //
 553:     // Expands to:
 554:     // or Rd,   Rr
 555:     // or Rd+1, Rr+1
 556:     def ORWRdRr : Pseudo<(outs DREGS:$rd), (ins DREGS:$src, DREGS:$rr),
 557:                          "orw\t$rd, $rr",
 558:                          [(set i16:$rd, (or i16:$src, i16:$rr))]>;
```

- EN: This range defines declarative TableGen records such as ANDWRdRr, ORRdRr, ORWRdRr, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ANDWRdRr, ORRdRr, ORWRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 559-576

```tablegen
 559: 
 560:     def EORRdRr : FRdRr<0b0010, 0b01, (outs GPR8:$rd),
 561:                         (ins GPR8:$src, GPR8:$rr), "eor\t$rd, $rr",
 562:                         [(set i8:$rd, (xor i8:$src, i8:$rr))]>;
 563: 
 564:     // EORW Rd+1:Rd, Rr+1:Rr
 565:     //
 566:     // Expands to:
 567:     // eor Rd,   Rr
 568:     // eor Rd+1, Rr+1
 569:     def EORWRdRr : Pseudo<(outs DREGS:$rd), (ins DREGS:$src, DREGS:$rr),
 570:                           "eorw\t$rd, $rr",
 571:                           [(set i16:$rd, (xor i16:$src, i16:$rr))]>;
 572:   }
 573: 
 574:   def ANDIRdK : FRdK<0b0111, (outs LD8:$rd), (ins LD8:$src, imm_ldi8:$k),
 575:                      "andi\t$rd, $k",
 576:                      [(set i8:$rd, (and i8:$src, imm:$k))]>;
```

- EN: This range defines declarative TableGen records such as EORRdRr, EORWRdRr, ANDIRdK, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 EORRdRr, EORWRdRr, ANDIRdK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 577-594

```tablegen
 577: 
 578:   // ANDI Rd+1:Rd, K+1:K
 579:   //
 580:   // Expands to:
 581:   // andi Rd,   K
 582:   // andi Rd+1, K+1
 583:   def ANDIWRdK : Pseudo<(outs DLDREGS:$rd), (ins DLDREGS:$src, i16imm:$k),
 584:                         "andiw\t$rd, $k",
 585:                         [(set i16:$rd, (and i16:$src, imm:$k))]>;
 586: 
 587:   def ORIRdK : FRdK<0b0110, (outs LD8:$rd), (ins LD8:$src, imm_ldi8:$k),
 588:                     "ori\t$rd, $k",
 589:                     [(set i8:$rd, (or i8:$src, imm:$k))]>;
 590: 
 591:   // ORIW Rd+1:Rd, K+1,K
 592:   //
 593:   // Expands to:
 594:   // ori Rd,   K
```

- EN: This range defines declarative TableGen records such as ANDIWRdK, ORIRdK, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ANDIWRdK, ORIRdK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 595-612

```tablegen
 595:   // ori Rd+1, K+1
 596:   def ORIWRdK : Pseudo<(outs DLDREGS:$rd), (ins DLDREGS:$src, i16imm:$rr),
 597:                        "oriw\t$rd, $rr",
 598:                        [(set i16:$rd, (or i16:$src, imm:$rr))]>;
 599: }
 600: 
 601: //===----------------------------------------------------------------------===//
 602: // One's/Two's Complement
 603: //===----------------------------------------------------------------------===//
 604: let hasSideEffects = 0, Constraints = "$src = $rd", Defs = [SREG] in {
 605:   def COMRd : FRd<0b1001, 0b0100000, (outs GPR8:$rd), (ins GPR8:$src),
 606:                   "com\t$rd", [(set i8:$rd, (not i8:$src))]>;
 607: 
 608:   // COMW Rd+1:Rd
 609:   //
 610:   // Expands to:
 611:   // com Rd
 612:   // com Rd+1
```

- EN: This range defines declarative TableGen records such as ORIWRdK, COMRd, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ORIWRdK, COMRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 613-630

```tablegen
 613:   def COMWRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src), "comw\t$rd",
 614:                       [(set i16:$rd, (not i16:$src))]>;
 615: 
 616:   def NEGRd : FRd<0b1001, 0b0100001, (outs GPR8:$rd), (ins GPR8:$src),
 617:                   "neg\t$rd", [(set i8:$rd, (ineg i8:$src))]>;
 618: 
 619:   // NEGW Rd+1:Rd
 620:   //
 621:   // Expands to:
 622:   // neg Rd+1
 623:   // neg Rd
 624:   // sbc Rd+1, r1
 625:   def NEGWRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src, GPR8:$zero),
 626:                       "negw\t$rd", []>;
 627: }
 628: 
 629: // TST Rd
 630: // Test for zero of minus.
```

- EN: This range defines declarative TableGen records such as COMWRd, NEGRd, NEGWRd, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 COMWRd, NEGRd, NEGWRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 631-648

```tablegen
 631: // This operation is identical to a `Rd AND Rd`.
 632: def : InstAlias<"tst\t$rd", (ANDRdRr GPR8:$rd, GPR8:$rd)>;
 633: 
 634: // SBR Rd, K
 635: //
 636: // Mnemonic alias to 'ORI Rd, K'. Same bit pattern, same operands,
 637: // same everything.
 638: def : InstAlias<"sbr\t$rd, $k", (ORIRdK LD8:$rd, imm_ldi8:$k),
 639:                 /* Disable display, so we don't override ORI */ 0>;
 640: 
 641: //===----------------------------------------------------------------------===//
 642: // Jump instructions
 643: //===----------------------------------------------------------------------===//
 644: let isBarrier = 1, isBranch = 1, isTerminator = 1 in {
 645:   def RJMPk : FBRk<0, (outs), (ins brtarget_13:$k), "rjmp\t$k", [(br bb:$k)]>;
 646: 
 647:   let isIndirectBranch = 1, Uses = [R31R30] in
 648:   def IJMP : F16<0b1001010000001001, (outs), (ins), "ijmp", []>,
```

- EN: This range defines declarative TableGen records such as RJMPk, IJMP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 RJMPk, IJMP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 649-666

```tablegen
 649:              Requires<[HasIJMPCALL]>;
 650: 
 651:   let isIndirectBranch = 1, Uses = [R31R30] in
 652:   def EIJMP : F16<0b1001010000011001, (outs), (ins), "eijmp", []>,
 653:               Requires<[HasEIJMPCALL]>;
 654: 
 655:   def JMPk : F32BRk<0b110, (outs), (ins call_target:$k), "jmp\t$k", []>,
 656:              Requires<[HasJMPCALL]>;
 657: }
 658: 
 659: //===----------------------------------------------------------------------===//
 660: // Call instructions
 661: //===----------------------------------------------------------------------===//
 662: let isCall = 1 in {
 663:   // SP is marked as a use to prevent stack-pointer assignments that appear
 664:   // immediately before calls from potentially appearing dead.
 665:   let Uses = [SP] in
 666:   def RCALLk : FBRk<1, (outs), (ins rcalltarget_13:$k), "rcall\t$k",
```

- EN: This range defines declarative TableGen records such as EIJMP, JMPk, RCALLk, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 EIJMP, JMPk, RCALLk 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 667-684

```tablegen
 667:                     [(AVRcall imm:$k)]>;
 668: 
 669:   // SP is marked as a use to prevent stack-pointer assignments that appear
 670:   // immediately before calls from potentially appearing dead.
 671:   let Uses = [SP, R31R30] in
 672:   def ICALL : F16<0b1001010100001001, (outs), (ins variable_ops), "icall", []>,
 673:               Requires<[HasIJMPCALL]>;
 674: 
 675:   // SP is marked as a use to prevent stack-pointer assignments that appear
 676:   // immediately before calls from potentially appearing dead.
 677:   let Uses = [SP, R31R30] in
 678:   def EICALL : F16<0b1001010100011001, (outs), (ins variable_ops), "eicall",
 679:                    []>,
 680:       Requires<[HasEIJMPCALL]>;
 681: 
 682:   // SP is marked as a use to prevent stack-pointer assignments that appear
 683:   // immediately before calls from potentially appearing dead.
 684:   //
```

- EN: This range defines declarative TableGen records such as ICALL, EICALL, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ICALL, EICALL 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 685-702

```tablegen
 685:   // TODO: the imm field can be either 16 or 22 bits in devices with more
 686:   // than 64k of ROM, fix it once we support the largest devices.
 687:   let Uses = [SP] in
 688:   def CALLk : F32BRk<0b111, (outs), (ins call_target:$k), "call\t$k",
 689:                      [(AVRcall imm:$k)]>,
 690:               Requires<[HasJMPCALL]>;
 691: }
 692: 
 693: //===----------------------------------------------------------------------===//
 694: // Return instructions.
 695: //===----------------------------------------------------------------------===//
 696: let isTerminator = 1, isReturn = 1, isBarrier = 1 in {
 697:   def RET : F16<0b1001010100001000, (outs), (ins), "ret", [(AVRretglue)]>;
 698: 
 699:   def RETI : F16<0b1001010100011000, (outs), (ins), "reti", [(AVRretiglue)]>;
 700: }
 701: 
 702: //===----------------------------------------------------------------------===//
```

- EN: This range defines declarative TableGen records such as CALLk, RET, RETI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CALLk, RET, RETI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 703-720

```tablegen
 703: // Compare operations.
 704: //===----------------------------------------------------------------------===//
 705: let hasSideEffects = 0, Defs = [SREG] in {
 706:   // CPSE Rd, Rr
 707:   // Compare Rd and Rr, skipping the next instruction if they are equal.
 708:   let isBarrier = 1, isBranch = 1, isTerminator = 1 in
 709:   def CPSE : FRdRr<0b0001, 0b00, (outs), (ins GPR8:$rd, GPR8:$rr),
 710:                    "cpse\t$rd, $rr", []>;
 711: 
 712:   def CPRdRr : FRdRr<0b0001, 0b01, (outs), (ins GPR8:$rd, GPR8:$rr),
 713:                      "cp\t$rd, $rr",
 714:                      [(AVRcmp i8:$rd, i8:$rr)]>;
 715: 
 716:   // CPW Rd+1:Rd, Rr+1:Rr
 717:   //
 718:   // Expands to:
 719:   // cp  Rd,   Rr
 720:   // cpc Rd+1, Rr+1
```

- EN: This range defines declarative TableGen records such as CPSE, CPRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CPSE, CPRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 721-738

```tablegen
 721:   def CPWRdRr : Pseudo<(outs), (ins DREGS:$src, DREGS:$src2),
 722:                        "cpw\t$src, $src2",
 723:                        [(AVRcmp i16:$src, i16:$src2)]>;
 724: 
 725:   let Uses = [SREG] in
 726:   def CPCRdRr : FRdRr<0b0000, 0b01, (outs), (ins GPR8:$rd, GPR8:$rr),
 727:                       "cpc\t$rd, $rr",
 728:                       [(AVRcmpc i8:$rd, i8:$rr)]>;
 729: 
 730:   // CPCW Rd+1:Rd. Rr+1:Rr
 731:   //
 732:   // Expands to:
 733:   // cpc Rd,   Rr
 734:   // cpc Rd+1, Rr+1
 735:   let Uses = [SREG] in
 736:   def CPCWRdRr : Pseudo<(outs), (ins DREGS:$src, DREGS:$src2),
 737:                         "cpcw\t$src, $src2",
 738:                         [(AVRcmpc i16:$src, i16:$src2)]>;
```

- EN: This range defines declarative TableGen records such as CPWRdRr, CPCRdRr, CPCWRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CPWRdRr, CPCRdRr, CPCWRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 739-756

```tablegen
 739: 
 740:   // CPI Rd, K
 741:   // Compares a register with an 8 bit immediate.
 742:   def CPIRdK : FRdK<0b0011, (outs), (ins LD8:$rd, imm_ldi8:$k), "cpi\t$rd, $k",
 743:                     [(AVRcmp i8:$rd, imm:$k)]>;
 744: }
 745: 
 746: //===----------------------------------------------------------------------===//
 747: // Register conditional skipping/branching operations.
 748: //===----------------------------------------------------------------------===//
 749: let isBranch = 1, isTerminator = 1 in {
 750:   // Conditional skipping on GPR register bits, and
 751:   // conditional skipping on IO register bits.
 752:   let isBarrier = 1 in {
 753:     def SBRCRrB : FRdB<0b10, (outs), (ins GPR8:$rd, i8imm:$b), "sbrc\t$rd, $b",
 754:                        []>;
 755: 
 756:     def SBRSRrB : FRdB<0b11, (outs), (ins GPR8:$rd, i8imm:$b), "sbrs\t$rd, $b",
```

- EN: This range defines declarative TableGen records such as CPIRdK, SBRCRrB, SBRSRrB, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CPIRdK, SBRCRrB, SBRSRrB 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 757-774

```tablegen
 757:                        []>;
 758: 
 759:     def SBICAb : FIOBIT<0b01, (outs), (ins imm_port5:$addr, i8imm:$b),
 760:                         "sbic\t$addr, $b", []>;
 761: 
 762:     def SBISAb : FIOBIT<0b11, (outs), (ins imm_port5:$addr, i8imm:$b),
 763:                         "sbis\t$addr, $b", []>;
 764:   }
 765: 
 766:   // Relative branches on status flag bits.
 767:   let Uses = [SREG] in {
 768:     // BRBS s, k
 769:     // Branch if `s` flag in status register is set.
 770:     def BRBSsk : FSK<0, (outs), (ins i8imm:$s, relbrtarget_7:$k),
 771:                      "brbs\t$s, $k", []>;
 772: 
 773:     // BRBC s, k
 774:     // Branch if `s` flag in status register is clear.
```

- EN: This range defines declarative TableGen records such as SBICAb, SBISAb, BRBSsk, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SBICAb, SBISAb, BRBSsk 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 775-792

```tablegen
 775:     def BRBCsk : FSK<1, (outs), (ins i8imm:$s, relbrtarget_7:$k),
 776:                      "brbc\t$s, $k", []>;
 777:   }
 778: }
 779: 
 780: // BRCS k
 781: // Branch if carry flag is set
 782: def : InstAlias<"brcs\t$k", (BRBSsk 0, relbrtarget_7 : $k)>;
 783: 
 784: // BRCC k
 785: // Branch if carry flag is clear
 786: def : InstAlias<"brcc\t$k", (BRBCsk 0, relbrtarget_7 : $k)>;
 787: 
 788: // BRHS k
 789: // Branch if half carry flag is set
 790: def : InstAlias<"brhs\t$k", (BRBSsk 5, relbrtarget_7 : $k)>;
 791: 
 792: // BRHC k
```

- EN: This range defines declarative TableGen records such as BRBCsk, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 BRBCsk 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 793-810

```tablegen
 793: // Branch if half carry flag is clear
 794: def : InstAlias<"brhc\t$k", (BRBCsk 5, relbrtarget_7 : $k)>;
 795: 
 796: // BRTS k
 797: // Branch if the T flag is set
 798: def : InstAlias<"brts\t$k", (BRBSsk 6, relbrtarget_7 : $k)>;
 799: 
 800: // BRTC k
 801: // Branch if the T flag is clear
 802: def : InstAlias<"brtc\t$k", (BRBCsk 6, relbrtarget_7 : $k)>;
 803: 
 804: // BRVS k
 805: // Branch if the overflow flag is set
 806: def : InstAlias<"brvs\t$k", (BRBSsk 3, relbrtarget_7 : $k)>;
 807: 
 808: // BRVC k
 809: // Branch if the overflow flag is clear
 810: def : InstAlias<"brvc\t$k", (BRBCsk 3, relbrtarget_7 : $k)>;
```

- EN: This range continues the implementation of the backend component described by AVRInstrInfo.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 811-828

```tablegen
 811: 
 812: // BRIE k
 813: // Branch if the global interrupt flag is enabled
 814: def : InstAlias<"brie\t$k", (BRBSsk 7, relbrtarget_7 : $k)>;
 815: 
 816: // BRID k
 817: // Branch if the global interrupt flag is disabled
 818: def : InstAlias<"brid\t$k", (BRBCsk 7, relbrtarget_7 : $k)>;
 819: 
 820: //===----------------------------------------------------------------------===//
 821: // PC-relative conditional branches
 822: //===----------------------------------------------------------------------===//
 823: // Based on status register. We cannot simplify these into instruction aliases
 824: // because we also need to be able to specify a pattern to match for ISel.
 825: let isBranch = 1, isTerminator = 1, Uses = [SREG] in {
 826:   def BREQk : FBRsk<0, 0b001, (outs), (ins relbrtarget_7:$k), "breq\t$k",
 827:                     [(AVRbrcond bb:$k, AVR_COND_EQ)]>;
 828: 
```

- EN: This range defines declarative TableGen records such as BREQk, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BREQk 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 829-846

```tablegen
 829:   def BRNEk : FBRsk<1, 0b001, (outs), (ins relbrtarget_7:$k), "brne\t$k",
 830:                     [(AVRbrcond bb:$k, AVR_COND_NE)]>;
 831: 
 832:   def BRSHk : FBRsk<1, 0b000, (outs), (ins relbrtarget_7:$k), "brsh\t$k",
 833:                     [(AVRbrcond bb:$k, AVR_COND_SH)]>;
 834: 
 835:   def BRLOk : FBRsk<0, 0b000, (outs), (ins relbrtarget_7:$k), "brlo\t$k",
 836:                     [(AVRbrcond bb:$k, AVR_COND_LO)]>;
 837: 
 838:   def BRMIk : FBRsk<0, 0b010, (outs), (ins relbrtarget_7:$k), "brmi\t$k",
 839:                     [(AVRbrcond bb:$k, AVR_COND_MI)]>;
 840: 
 841:   def BRPLk : FBRsk<1, 0b010, (outs), (ins relbrtarget_7:$k), "brpl\t$k",
 842:                     [(AVRbrcond bb:$k, AVR_COND_PL)]>;
 843: 
 844:   def BRGEk : FBRsk<1, 0b100, (outs), (ins relbrtarget_7:$k), "brge\t$k",
 845:                     [(AVRbrcond bb:$k, AVR_COND_GE)]>;
 846: 
```

- EN: This range defines declarative TableGen records such as BRNEk, BRSHk, BRLOk, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 BRNEk, BRSHk, BRLOk 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 847-864

```tablegen
 847:   def BRLTk : FBRsk<0, 0b100, (outs), (ins relbrtarget_7:$k), "brlt\t$k",
 848:                     [(AVRbrcond bb:$k, AVR_COND_LT)]>;
 849: }
 850: 
 851: //===----------------------------------------------------------------------===//
 852: // Data transfer instructions
 853: //===----------------------------------------------------------------------===//
 854: // 8 and 16-bit register move instructions.
 855: let hasSideEffects = 0 in {
 856:   def MOVRdRr : FRdRr<0b0010, 0b11, (outs GPR8:$rd), (ins GPR8:$rr),
 857:                       "mov\t$rd, $rr", []>;
 858: 
 859:   def MOVWRdRr : FMOVWRdRr<(outs DREGS:$rd), (ins DREGS:$rr), "movw\t$rd, $rr",
 860:                            []>,
 861:                  Requires<[HasMOVW]>;
 862: }
 863: 
 864: // Load immediate values into registers.
```

- EN: This range defines declarative TableGen records such as BRLTk, MOVRdRr, MOVWRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BRLTk, MOVRdRr, MOVWRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 865-882

```tablegen
 865: let hasSideEffects = 0, isReMaterializable = 1 in {
 866:   def LDIRdK : FRdK<0b1110, (outs LD8:$rd), (ins imm_ldi8:$k), "ldi\t$rd, $k",
 867:                     [(set i8:$rd, imm:$k)]>;
 868: 
 869:   // LDIW Rd+1:Rd, K+1:K
 870:   //
 871:   // Expands to:
 872:   // ldi Rd,   K
 873:   // ldi Rd+1, K+1
 874:   def LDIWRdK : Pseudo<(outs DLDREGS:$dst), (ins i16imm:$src),
 875:                        "ldiw\t$dst, $src", [(set i16:$dst, imm:$src)]>;
 876: }
 877: 
 878: // Load from data space into register.
 879: let hasSideEffects = 0, mayLoad = 1, isReMaterializable = 1 in {
 880:   def LDSRdK : F32DM<0b0, (outs GPR8:$rd), (ins imm16:$k), "lds\t$rd, $k",
 881:                      [(set i8:$rd, (load imm:$k))]>,
 882:                Requires<[HasSRAM, HasNonTinyEncoding]>;
```

- EN: This range defines declarative TableGen records such as LDIRdK, LDIWRdK, LDSRdK, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDIRdK, LDIWRdK, LDSRdK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 883-900

```tablegen
 883: 
 884:   // Load from data space into register, which is only available on AVRTiny.
 885:   def LDSRdKTiny : FLDSSTSTINY<0b0, (outs LD8:$rd), (ins imm7tiny:$k),
 886:                                "lds\t$rd, $k", [(set i8:$rd, (load imm:$k))]>,
 887:                    Requires<[HasSRAM, HasTinyEncoding]>;
 888: 
 889:   // LDSW Rd+1:Rd, K+1:K
 890:   //
 891:   // Expands to:
 892:   // lds Rd,  (K+1:K)
 893:   // lds Rd+1 (K+1:K) + 1
 894:   def LDSWRdK : Pseudo<(outs DREGS:$dst), (ins i16imm:$src), "ldsw\t$dst, $src",
 895:                        [(set i16:$dst, (load imm:$src))]>,
 896:                 Requires<[HasSRAM, HasNonTinyEncoding]>;
 897: }
 898: 
 899: // Indirect loads.
 900: let hasSideEffects = 0, mayLoad = 1 in {
```

- EN: This range defines declarative TableGen records such as LDSRdKTiny, LDSWRdK, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDSRdKTiny, LDSWRdK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 901-918

```tablegen
 901:   let isReMaterializable = 1 in {
 902:     def LDRdPtr : FSTLD<0, 0b00, (outs GPR8:$reg), (ins PTRREGS:$ptrreg),
 903:                         "ld\t$reg, $ptrreg",
 904:                         [(set GPR8:$reg, (load i16:$ptrreg))]>,
 905:                   Requires<[HasSRAM]>;
 906: 
 907:     // LDW Rd+1:Rd, P
 908:     //
 909:     // Expands to:
 910:     //   ld  Rd,   P
 911:     //   ldd Rd+1, P+1
 912:     // On reduced tiny cores, this instruction expands to:
 913:     //   ld    Rd,   P+
 914:     //   ld    Rd+1, P+
 915:     //   subiw P,    2
 916:     let Constraints = "@earlyclobber $reg" in
 917:     def LDWRdPtr : Pseudo<(outs DREGSNOZ:$reg), (ins PTRDISPREGS:$ptrreg),
 918:                           "ldw\t$reg, $ptrreg",
```

- EN: This range defines declarative TableGen records such as LDRdPtr, LDWRdPtr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDRdPtr, LDWRdPtr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 919-936

```tablegen
 919:                           [(set i16:$reg, (load i16:$ptrreg))]>,
 920:                    Requires<[HasSRAM]>;
 921:   }
 922: 
 923:   // Indirect loads (with postincrement or predecrement).
 924:   let isReMaterializable = 0,
 925:       Constraints = "$ptrreg = $base_wb,@earlyclobber $reg" in {
 926:     def LDRdPtrPi : FSTLD<0, 0b01, (outs GPR8:$reg, PTRREGS:$base_wb),
 927:                           (ins PTRREGS:$ptrreg), "ld\t$reg, $ptrreg+", []>,
 928:                     Requires<[HasSRAM]>;
 929: 
 930:     // LDW Rd+1:Rd, P+
 931:     // Expands to:
 932:     // ld Rd,   P+
 933:     // ld Rd+1, P+
 934:     def LDWRdPtrPi : Pseudo<(outs DREGS:$reg, PTRREGS:$base_wb),
 935:                             (ins PTRREGS:$ptrreg), "ldw\t$reg, $ptrreg+", []>,
 936:                      Requires<[HasSRAM]>;
```

- EN: This range defines declarative TableGen records such as LDRdPtrPi, LDWRdPtrPi, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDRdPtrPi, LDWRdPtrPi 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 937-954

```tablegen
 937: 
 938:     def LDRdPtrPd : FSTLD<0, 0b10, (outs GPR8:$reg, PTRREGS:$base_wb),
 939:                           (ins PTRREGS:$ptrreg), "ld\t$reg, -$ptrreg", []>,
 940:                     Requires<[HasSRAM]>;
 941: 
 942:     // LDW Rd+1:Rd, -P
 943:     //
 944:     // Expands to:
 945:     // ld Rd+1, -P
 946:     // ld Rd,   -P
 947:     def LDWRdPtrPd : Pseudo<(outs DREGS:$reg, PTRREGS:$base_wb),
 948:                             (ins PTRREGS:$ptrreg), "ldw\t$reg, -$ptrreg", []>,
 949:                      Requires<[HasSRAM]>;
 950:   }
 951: 
 952:   // Load indirect with displacement operations.
 953:   let isReMaterializable = 1 in {
 954:     def LDDRdPtrQ : FSTDLDD<0, (outs GPR8:$reg), (ins memri:$memri),
```

- EN: This range defines declarative TableGen records such as LDRdPtrPd, LDWRdPtrPd, LDDRdPtrQ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDRdPtrPd, LDWRdPtrPd, LDDRdPtrQ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 955-972

```tablegen
 955:                             "ldd\t$reg, $memri",
 956:                             [(set i8:$reg, (load addr:$memri))]>,
 957:                     Requires<[HasSRAM, HasNonTinyEncoding]>;
 958: 
 959:     // LDDW Rd+1:Rd, P+q
 960:     //
 961:     // Expands to:
 962:     //   ldd Rd,   P+q
 963:     //   ldd Rd+1, P+q+1
 964:     // On reduced tiny cores, this instruction expands to:
 965:     //   subiw P,    -q
 966:     //   ld    Rd,   P+
 967:     //   ld    Rd+1, P+
 968:     //   subiw P,    q+2
 969:     let Constraints = "@earlyclobber $dst" in
 970:     def LDDWRdPtrQ : Pseudo<(outs DREGS:$dst), (ins memri:$memri),
 971:                             "lddw\t$dst, $memri",
 972:                             [(set i16:$dst, (load addr:$memri))]>,
```

- EN: This range defines declarative TableGen records such as LDDWRdPtrQ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDDWRdPtrQ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 973-990

```tablegen
 973:                      Requires<[HasSRAM]>;
 974: 
 975:     // An identical pseudo instruction to LDDWRdPtrQ, expect restricted to the Y
 976:     // register and without the @earlyclobber flag.
 977:     //
 978:     // Used to work around a bug caused by the register allocator not
 979:     // being able to handle the expansion of a COPY into an machine instruction
 980:     // that has an earlyclobber flag. This is because the register allocator will
 981:     // try expand a copy from a register slot into an earlyclobber instruction.
 982:     // Instructions that are earlyclobber need to be in a dedicated earlyclobber
 983:     // slot.
 984:     //
 985:     // This pseudo instruction can be used pre-AVR pseudo expansion in order to
 986:     // get a frame index load without directly using earlyclobber instructions.
 987:     //
 988:     // The pseudo expansion pass trivially expands this into LDDWRdPtrQ.
 989:     //
 990:     // This instruction may be removed once PR13375 is fixed.
```

- EN: This range continues the implementation of the backend component described by AVRInstrInfo.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 991-1008

```tablegen
 991:     def LDDWRdYQ : Pseudo<(outs DREGS:$dst), (ins memri:$memri),
 992:                           "lddw\t$dst, $memri", []>,
 993:                    Requires<[HasSRAM]>;
 994:   }
 995: }
 996: 
 997: let hasSideEffects = 0, mayLoad = 1, isReMaterializable = 1 in
 998: class AtomicLoad<PatFrag Op, RegisterClass DRC, RegisterClass PTRRC>
 999:     : Pseudo<(outs DRC:$rd), (ins PTRRC:$rr), "atomic_op",
1000:              [(set DRC:$rd, (Op i16:$rr))]>;
1001: 
1002: let hasSideEffects = 0, mayStore = 1 in
1003: class AtomicStore<PatFrag Op, RegisterClass DRC, RegisterClass PTRRC>
1004:     : Pseudo<(outs), (ins PTRRC:$rd, DRC:$rr), "atomic_op",
1005:              [(Op DRC:$rr, i16:$rd)]>;
1006: 
1007: let hasSideEffects = 0, mayLoad = 1, mayStore = 1 in
1008: class AtomicLoadOp<PatFrag Op, RegisterClass DRC, RegisterClass PTRRC>
```

- EN: This range defines declarative TableGen records such as LDDWRdYQ, AtomicLoad, AtomicStore, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDDWRdYQ, AtomicLoad, AtomicStore 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1009-1026

```tablegen
1009:     : Pseudo<(outs DRC:$rd), (ins PTRRC:$rr, DRC:$operand), "atomic_op",
1010:              [(set DRC:$rd, (Op i16:$rr, DRC:$operand))]>;
1011: 
1012: // Atomic instructions
1013: // ===================
1014: //
1015: // 8-bit operations can use any pointer register because
1016: // they are expanded directly into an LD/ST instruction.
1017: //
1018: // 16-bit operations use 16-bit load/store postincrement instructions,
1019: // which require PTRDISPREGS.
1020: 
1021: def AtomicLoad8 : AtomicLoad<atomic_load_nonext_8, GPR8, PTRREGS>;
1022: def AtomicLoad16 : AtomicLoad<atomic_load_nonext_16, DREGSNOZ, PTRDISPREGS>;
1023: 
1024: def AtomicStore8 : AtomicStore<atomic_store_8, GPR8, PTRREGS>;
1025: def AtomicStore16 : AtomicStore<atomic_store_16, DREGSNOZ, PTRDISPREGS>;
1026: 
```

- EN: This range defines declarative TableGen records such as AtomicLoad8, AtomicLoad16, AtomicStore8, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 AtomicLoad8, AtomicLoad16, AtomicStore8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 1027-1044

```tablegen
1027: class AtomicLoadOp8<PatFrag Op> : AtomicLoadOp<Op, GPR8, PTRREGS>;
1028: class AtomicLoadOp16<PatFrag Op> : AtomicLoadOp<Op, DREGSNOZ, PTRDISPREGS>;
1029: 
1030: let usesCustomInserter = 1, Defs = [SREG] in {
1031:   def AtomicLoadAdd8 : AtomicLoadOp8<atomic_load_add_i8>;
1032:   def AtomicLoadAdd16 : AtomicLoadOp16<atomic_load_add_i16>;
1033:   def AtomicLoadSub8 : AtomicLoadOp8<atomic_load_sub_i8>;
1034:   def AtomicLoadSub16 : AtomicLoadOp16<atomic_load_sub_i16>;
1035:   def AtomicLoadAnd8 : AtomicLoadOp8<atomic_load_and_i8>;
1036:   def AtomicLoadAnd16 : AtomicLoadOp16<atomic_load_and_i16>;
1037:   def AtomicLoadOr8 : AtomicLoadOp8<atomic_load_or_i8>;
1038:   def AtomicLoadOr16 : AtomicLoadOp16<atomic_load_or_i16>;
1039:   def AtomicLoadXor8 : AtomicLoadOp8<atomic_load_xor_i8>;
1040:   def AtomicLoadXor16 : AtomicLoadOp16<atomic_load_xor_i16>;
1041: }
1042: 
1043: let hasSideEffects = 1 in
1044: def AtomicFence
```

- EN: This range defines declarative TableGen records such as AtomicLoadOp8, AtomicLoadOp16, AtomicLoadAdd8, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 AtomicLoadOp8, AtomicLoadOp16, AtomicLoadAdd8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1045-1062

```tablegen
1045:     : Pseudo<(outs), (ins), "atomic_fence", [(atomic_fence timm, timm)]>;
1046: 
1047: // Indirect store from register to data space.
1048: let hasSideEffects = 0, mayStore = 1 in {
1049:   def STSKRr : F32DM<0b1, (outs), (ins imm16:$k, GPR8:$rd), "sts\t$k, $rd",
1050:                      [(store i8:$rd, imm:$k)]>,
1051:                Requires<[HasSRAM, HasNonTinyEncoding]>;
1052: 
1053:   // Store from register to data space, which is only available on AVRTiny.
1054:   def STSKRrTiny : FLDSSTSTINY<0b1, (outs), (ins imm7tiny:$k, LD8:$rd),
1055:                                "sts\t$k, $rd", [(store i8:$rd, imm:$k)]>,
1056:                    Requires<[HasSRAM, HasTinyEncoding]>;
1057: }
1058: 
1059: // STSW K+1:K, Rr+1:Rr
1060: //
1061: // Expands to:
1062: // sts Rr+1, (K+1:K) + 1
```

- EN: This range defines declarative TableGen records such as STSKRr, STSKRrTiny, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STSKRr, STSKRrTiny 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1063-1080

```tablegen
1063: // sts Rr,   (K+1:K)
1064: let hasSideEffects = 0, mayStore = 1 in
1065: def STSWKRr : Pseudo<(outs), (ins i16imm:$dst, DREGS:$src),
1066:                      "stsw\t$dst, $src", [(store i16:$src, imm:$dst)]>,
1067:               Requires<[HasSRAM, HasNonTinyEncoding]>;
1068: 
1069: // Indirect stores.
1070: // ST P, Rr
1071: // Stores the value of Rr into the location addressed by pointer P.
1072: let hasSideEffects = 0, mayStore = 1 in
1073: def STPtrRr : FSTLD<1, 0b00, (outs), (ins PTRREGS:$ptrreg, GPR8:$reg),
1074:                     "st\t$ptrreg, $reg", [(store GPR8:$reg, i16:$ptrreg)]>,
1075:               Requires<[HasSRAM]>;
1076: 
1077: // STW P, Rr+1:Rr
1078: // Stores the value of Rr into the location addressed by pointer P.
1079: //
1080: // Expands to:
```

- EN: This range defines declarative TableGen records such as STSWKRr, STPtrRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STSWKRr, STPtrRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1081-1098

```tablegen
1081: //   st P, Rr
1082: //   std P+1, Rr+1
1083: // On reduced tiny cores, this instruction expands to:
1084: //   st    P+, Rr
1085: //   st    P+, Rr+1
1086: //   subiw P,  q+2
1087: let hasSideEffects = 0, mayStore = 1 in
1088: def STWPtrRr : Pseudo<(outs), (ins PTRDISPREGS:$ptrreg, DREGSNOZ:$reg),
1089:                       "stw\t$ptrreg, $reg", [(store i16:$reg, i16:$ptrreg)]>,
1090:                Requires<[HasSRAM]>;
1091: 
1092: // Indirect stores (with postincrement or predecrement).
1093: let hasSideEffects = 0, mayStore = 1,
1094:     Constraints = "$ptrreg = $base_wb,@earlyclobber $base_wb" in {
1095:   // ST P+, Rr
1096:   // Stores the value of Rr into the location addressed by pointer P.
1097:   // Post increments P.
1098:   def STPtrPiRr : FSTLD<1, 0b01, (outs PTRREGS:$base_wb),
```

- EN: This range defines declarative TableGen records such as STWPtrRr, STPtrPiRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STWPtrRr, STPtrPiRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1099-1116

```tablegen
1099:                         (ins PTRREGS:$ptrreg, GPR8:$reg, i8imm:$offs),
1100:                         "st\t$ptrreg+, $reg",
1101:                         [(set i16:$base_wb, (post_store GPR8:$reg, i16:$ptrreg,
1102:                          imm:$offs))]>,
1103:                   Requires<[HasSRAM]>;
1104: 
1105:   // STW P+, Rr+1:Rr
1106:   // Stores the value of Rr into the location addressed by pointer P.
1107:   // Post increments P.
1108:   //
1109:   // Expands to:
1110:   // st P+, Rr
1111:   // st P+, Rr+1
1112:   def STWPtrPiRr : Pseudo<(outs PTRREGS:$base_wb),
1113:                           (ins PTRREGS:$ptrreg, DREGS:$trh, i8imm:$offs),
1114:                           "stw\t$ptrreg+, $trh",
1115:                           [(set PTRREGS:$base_wb,
1116:                            (post_store DREGS:$trh, PTRREGS:$ptrreg,
```

- EN: This range defines declarative TableGen records such as STWPtrPiRr, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 STWPtrPiRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 1117-1134

```tablegen
1117:                             imm:$offs))]>,
1118:                    Requires<[HasSRAM]>;
1119: 
1120:   // ST -P, Rr
1121:   // Stores the value of Rr into the location addressed by pointer P.
1122:   // Pre decrements P.
1123:   def STPtrPdRr : FSTLD<1, 0b10, (outs PTRREGS:$base_wb),
1124:                         (ins PTRREGS:$ptrreg, GPR8:$reg, i8imm:$offs),
1125:                         "st\t-$ptrreg, $reg",
1126:                         [(set i16: $base_wb,
1127:                          (pre_store GPR8:$reg, i16:$ptrreg, imm:$offs))]>,
1128:                   Requires<[HasSRAM]>;
1129: 
1130:   // STW -P, Rr+1:Rr
1131:   // Stores the value of Rr into the location addressed by pointer P.
1132:   // Pre decrements P.
1133:   //
1134:   // Expands to:
```

- EN: This range defines declarative TableGen records such as STPtrPdRr, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 STPtrPdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 1135-1152

```tablegen
1135:   // st -P, Rr+1
1136:   // st -P, Rr
1137:   def STWPtrPdRr : Pseudo<(outs PTRREGS:$base_wb),
1138:                           (ins PTRREGS:$ptrreg, DREGS:$reg, i8imm:$offs),
1139:                           "stw\t-$ptrreg, $reg",
1140:                           [(set PTRREGS:$base_wb,
1141:                            (pre_store i16:$reg, i16:$ptrreg, imm:$offs))]>,
1142:                    Requires<[HasSRAM]>;
1143: }
1144: 
1145: // Store indirect with displacement operations.
1146: // STD P+q, Rr
1147: // Stores the value of Rr into the location addressed by pointer P with a
1148: // displacement of q. Does not modify P.
1149: let hasSideEffects = 0, mayStore = 1 in
1150: def STDPtrQRr : FSTDLDD<1, (outs), (ins memri:$memri, GPR8:$reg),
1151:                         "std\t$memri, $reg", [(store i8:$reg, addr:$memri)]>,
1152:                 Requires<[HasSRAM, HasNonTinyEncoding]>;
```

- EN: This range defines declarative TableGen records such as STWPtrPdRr, STDPtrQRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STWPtrPdRr, STDPtrQRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1153-1170

```tablegen
1153: 
1154: // STDW P+q, Rr+1:Rr
1155: // Stores the value of Rr into the location addressed by pointer P with a
1156: // displacement of q. Does not modify P.
1157: //
1158: // Expands to:
1159: //   std P+q,   Rr
1160: //   std P+q+1, Rr+1
1161: // On reduced tiny cores, this instruction expands to:
1162: //   subiw P,  -q
1163: //   st    P+, Rr
1164: //   st    P+, Rr+1
1165: //   subiw P,  q+2
1166: let hasSideEffects = 0, mayStore = 1 in
1167: def STDWPtrQRr : Pseudo<(outs), (ins memri:$memri, DREGS:$src),
1168:                         "stdw\t$memri, $src", [(store i16:$src, addr:$memri)]>,
1169:                  Requires<[HasSRAM]>;
1170: 
```

- EN: This range defines declarative TableGen records such as STDWPtrQRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STDWPtrQRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1171-1188

```tablegen
1171: // Load program memory operations.
1172: let isReMaterializable = 1, mayLoad = 1, hasSideEffects = 0 in {
1173:   let Defs = [R0], Uses = [R31R30] in
1174:       def LPM : F16<0b1001010111001000, (outs), (ins), "lpm", []>,
1175:                 Requires<[HasLPM]>;
1176: 
1177:   // These pseudo instructions are combination of the OUT and LPM instructions.
1178:   let Defs = [R0] in {
1179:     def LPMBRdZ : Pseudo<(outs GPR8:$dst), (ins ZREG:$z), "lpmb\t$dst, $z", []>,
1180:                   Requires<[HasLPM]>;
1181: 
1182:     let Constraints = "@earlyclobber $dst" in
1183:     def LPMWRdZ : Pseudo<(outs DREGS:$dst), (ins ZREG:$z), "lpmw\t$dst, $z", []>,
1184:                   Requires<[HasLPM]>;
1185:   }
1186: 
1187:   def LPMRdZ : FLPMX<0, 0, (outs GPR8:$rd), (ins ZREG:$z), "lpm\t$rd, $z", []>,
1188:                Requires<[HasLPMX]>;
```

- EN: This range defines declarative TableGen records such as LPM, LPMBRdZ, LPMWRdZ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LPM, LPMBRdZ, LPMWRdZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1189-1206

```tablegen
1189: }
1190: 
1191: let mayLoad = 1, hasSideEffects = 0 in {
1192:   // Load program memory, while postincrementing the Z register.
1193:   let Defs = [R31R30] in {
1194:     def LPMRdZPi : FLPMX<0, 1, (outs GPR8:$rd), (ins ZREG:$z),
1195:                          "lpm\t$rd, $z+", []>,
1196:                    Requires<[HasLPMX]>;
1197: 
1198:     def LPMWRdZPi : Pseudo<(outs DREGS:$dst), (ins ZREG:$z),
1199:                            "lpmw\t$dst, $z+", []>,
1200:                     Requires<[HasLPMX]>;
1201:   }
1202: }
1203: 
1204: // Extended load program memory operations.
1205: let mayLoad = 1, hasSideEffects = 0 in {
1206:   let Defs = [R0], Uses = [R31R30] in
```

- EN: This range defines declarative TableGen records such as LPMRdZPi, LPMWRdZPi, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LPMRdZPi, LPMWRdZPi 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1207-1224

```tablegen
1207:   def ELPM : F16<0b1001010111011000, (outs), (ins), "elpm", []>,
1208:              Requires<[HasELPM]>;
1209: 
1210:   def ELPMRdZ : FLPMX<1, 0, (outs GPR8:$rd), (ins ZREG:$z), "elpm\t$rd, $z",
1211:                       []>,
1212:                 Requires<[HasELPMX]>;
1213: 
1214:   let Defs = [R31R30] in
1215:   def ELPMRdZPi : FLPMX<1, 1, (outs GPR8:$rd), (ins ZREG:$z), "elpm\t$rd, $z+",
1216:                         []>,
1217:                   Requires<[HasELPMX]>;
1218: 
1219:   // These pseudo instructions are combination of the OUT and ELPM instructions.
1220:   let Defs = [R0], mayStore = 1 in {
1221:     def ELPMBRdZ : Pseudo<(outs GPR8:$dst), (ins ZREG:$z, LD8:$p),
1222:                           "elpmb\t$dst, $z, $p", []>,
1223:                    Requires<[HasELPM]>;
1224: 
```

- EN: This range defines declarative TableGen records such as ELPM, ELPMRdZ, ELPMRdZPi, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ELPM, ELPMRdZ, ELPMRdZPi 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1225-1242

```tablegen
1225:     let Constraints = "@earlyclobber $dst" in
1226:     def ELPMWRdZ : Pseudo<(outs DREGS:$dst), (ins ZREG:$z, LD8:$p),
1227:                           "elpmw\t$dst, $z, $p", []>,
1228:                    Requires<[HasELPM]>;
1229:   }
1230: 
1231:   // These pseudos are combination of the OUT and ELPM instructions.
1232:   let Defs = [R31R30], mayStore = 1 in {
1233:     def ELPMBRdZPi : Pseudo<(outs GPR8:$dst), (ins ZREG:$z, LD8:$p),
1234:                             "elpmb\t$dst, $z+, $p", []>,
1235:                      Requires<[HasELPMX]>;
1236: 
1237:     def ELPMWRdZPi : Pseudo<(outs DREGS:$dst), (ins ZREG:$z, LD8:$p),
1238:                             "elpmw\t$dst, $z+, $p", []>,
1239:                      Requires<[HasELPMX]>;
1240:   }
1241: }
1242: 
```

- EN: This range defines declarative TableGen records such as ELPMWRdZ, ELPMBRdZPi, ELPMWRdZPi, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ELPMWRdZ, ELPMBRdZPi, ELPMWRdZPi 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1243-1260

```tablegen
1243: // Store program memory operations.
1244: let Uses = [R1, R0], mayStore = 1, hasSideEffects = 0 in {
1245:   let Uses = [R31R30] in
1246:   def SPM : F16<0b1001010111101000, (outs), (ins), "spm", []>,
1247:             Requires<[HasSPM]>;
1248: 
1249:   let Defs = [R31R30] in
1250:   def SPMZPi : F16<0b1001010111111000, (outs), (ins ZREG:$z), "spm $z+", []>,
1251:                Requires<[HasSPMX]> {
1252:     bits<0> z;
1253:   }
1254: }
1255: 
1256: // Read data from IO location operations.
1257: let hasSideEffects = 1, mayLoad = 1 in {
1258:   def INRdA : FIORdA<(outs GPR8:$rd), (ins imm_port6:$A), "in\t$rd, $A",
1259:                      [(set i8:$rd, (load ioaddr8:$A))]>;
1260: 
```

- EN: This range defines declarative TableGen records such as SPM, SPMZPi, INRdA, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SPM, SPMZPi, INRdA 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1261-1278

```tablegen
1261:   def INWRdA : Pseudo<(outs DREGS:$d), (ins imm_port6:$s), "inw\t$d, $s",
1262:                       [(set i16:$d, (load ioaddr16:$s))]>;
1263: }
1264: 
1265: // Write data to IO location operations.
1266: let hasSideEffects = 1, mayStore = 1 in {
1267:   def OUTARr : FIOARr<(outs), (ins imm_port6:$A, GPR8:$rr), "out\t$A, $rr",
1268:                       [(store i8:$rr, ioaddr8:$A)]>;
1269: 
1270:   def OUTWARr : Pseudo<(outs), (ins imm_port6:$dst, DREGS:$src),
1271:                        "outw\t$dst, $src", [(store i16:$src, ioaddr16:$dst)]>;
1272: }
1273: 
1274: // Stack push/pop operations.
1275: let Defs = [SP], Uses = [SP], hasSideEffects = 0 in {
1276:   // Stack push operations.
1277:   let mayStore = 1 in {
1278:     def PUSHRr : FRd<0b1001, 0b0011111, (outs), (ins GPR8:$rd), "push\t$rd",
```

- EN: This range defines declarative TableGen records such as INWRdA, OUTARr, OUTWARr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 INWRdA, OUTARr, OUTWARr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1279-1296

```tablegen
1279:                      []>,
1280:                  Requires<[HasSRAM]>;
1281: 
1282:     def PUSHWRr : Pseudo<(outs), (ins DREGS:$reg), "pushw\t$reg", []>,
1283:                   Requires<[HasSRAM]>;
1284:   }
1285: 
1286:   // Stack pop operations.
1287:   let mayLoad = 1 in {
1288:     def POPRd : FRd<0b1001, 0b0001111, (outs GPR8:$rd), (ins), "pop\t$rd", []>,
1289:                 Requires<[HasSRAM]>;
1290: 
1291:     def POPWRd : Pseudo<(outs DREGS:$reg), (ins), "popw\t$reg", []>,
1292:                  Requires<[HasSRAM]>;
1293:   }
1294: }
1295: 
1296: // Read-Write-Modify (RMW) instructions.
```

- EN: This range defines declarative TableGen records such as PUSHWRr, POPRd, POPWRd, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 PUSHWRr, POPRd, POPWRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1297-1314

```tablegen
1297: let hasSideEffects = 0, mayLoad = 1, mayStore = 1 in {
1298:   def XCHZRd : FZRd<0b100, (outs GPR8:$rd), (ins ZREG:$z), "xch\t$z, $rd", []>,
1299:                Requires<[SupportsRMW]>;
1300: 
1301:   def LASZRd : FZRd<0b101, (outs GPR8:$rd), (ins ZREG:$z), "las\t$z, $rd", []>,
1302:                Requires<[SupportsRMW]>;
1303: 
1304:   def LACZRd : FZRd<0b110, (outs GPR8:$rd), (ins ZREG:$z), "lac\t$z, $rd", []>,
1305:                Requires<[SupportsRMW]>;
1306: 
1307:   def LATZRd : FZRd<0b111, (outs GPR8:$rd), (ins ZREG:$z), "lat\t$z, $rd", []>,
1308:                Requires<[SupportsRMW]>;
1309: }
1310: 
1311: //===----------------------------------------------------------------------===//
1312: // Bit and bit-test instructions
1313: //===----------------------------------------------------------------------===//
1314: 
```

- EN: This range defines declarative TableGen records such as XCHZRd, LASZRd, LACZRd, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 XCHZRd, LASZRd, LACZRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1315-1332

```tablegen
1315: // Bit shift/rotate operations.
1316: let hasSideEffects = 0, Constraints = "$src = $rd", Defs = [SREG] in {
1317:   // 8-bit LSL is an alias of ADD Rd, Rd
1318: 
1319:   def LSLWRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src), "lslw\t$rd",
1320:                       [(set i16:$rd, (AVRlsl i16:$src))]>;
1321: 
1322:   def LSLWHiRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src), "lslwhi\t$rd",
1323:                         [(set i16:$rd, (AVRlslhi i16:$src))]>;
1324: 
1325:   def LSLWNRd : Pseudo<(outs DLDREGS:$rd), (ins DREGS:$src, imm16:$bits),
1326:                        "lslwn\t$rd, $bits",
1327:                        [(set i16:$rd, (AVRlslwn i16:$src, imm:$bits))]>;
1328: 
1329:   def LSLBNRd : Pseudo<(outs LD8:$rd), (ins GPR8:$src, imm_ldi8:$bits),
1330:                        "lslbn\t$rd, $bits",
1331:                        [(set i8:$rd, (AVRlslbn i8:$src, imm:$bits))]>;
1332: 
```

- EN: This range defines declarative TableGen records such as LSLWRd, LSLWHiRd, LSLWNRd, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LSLWRd, LSLWHiRd, LSLWNRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1333-1350

```tablegen
1333:   def LSRRd : FRd<0b1001, 0b0100110, (outs GPR8:$rd), (ins GPR8:$src), "lsr\t$rd",
1334:                   [(set i8:$rd, (AVRlsr i8:$src))]>;
1335: 
1336:   def LSRWRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src), "lsrw\t$rd",
1337:                       [(set i16:$rd, (AVRlsr i16:$src))]>;
1338: 
1339:   def LSRWLoRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src), "lsrwlo\t$rd",
1340:                         [(set i16:$rd, (AVRlsrlo i16:$src))]>;
1341: 
1342:   def LSRWNRd : Pseudo<(outs DLDREGS:$rd), (ins DREGS:$src, imm16:$bits),
1343:                        "lsrwn\t$rd, $bits",
1344:                        [(set i16:$rd, (AVRlsrwn i16:$src, imm:$bits))]>;
1345: 
1346:   def LSRBNRd : Pseudo<(outs LD8:$rd), (ins GPR8:$src, imm_ldi8:$bits),
1347:                        "lsrbn\t$rd, $bits",
1348:                        [(set i8:$rd, (AVRlsrbn i8:$src, imm:$bits))]>;
1349: 
1350:   def ASRRd : FRd<0b1001, 0b0100101, (outs GPR8:$rd), (ins GPR8:$src), "asr\t$rd",
```

- EN: This range defines declarative TableGen records such as LSRRd, LSRWRd, LSRWLoRd, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 LSRRd, LSRWRd, LSRWLoRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 1351-1368

```tablegen
1351:                   [(set i8:$rd, (AVRasr i8:$src))]>;
1352: 
1353:   def ASRWNRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src, imm16:$bits),
1354:                        "asrwn\t$rd, $bits",
1355:                        [(set i16:$rd, (AVRasrwn i16:$src, imm:$bits))]>;
1356: 
1357:   def ASRBNRd : Pseudo<(outs LD8:$rd), (ins GPR8:$src, imm_ldi8:$bits),
1358:                        "asrbn\t$rd, $bits",
1359:                        [(set i8:$rd, (AVRasrbn i8:$src, imm:$bits))]>;
1360: 
1361:   def ASRWRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src), "asrw\t$rd",
1362:                       [(set i16:$rd, (AVRasr i16:$src))]>;
1363: 
1364:   def ASRWLoRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src), "asrwlo\t$rd",
1365:                         [(set i16:$rd, (AVRasrlo i16:$src))]>;
1366: 
1367:   let Uses = [R1] in
1368:   def ROLBRdR1 : Pseudo<(outs GPR8:$rd), (ins GPR8:$src), "rolb\t$rd",
```

- EN: This range defines declarative TableGen records such as ASRWNRd, ASRBNRd, ASRWRd, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ASRWNRd, ASRBNRd, ASRWRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1369-1386

```tablegen
1369:                         [(set i8:$rd, (AVRrol i8:$src))]>,
1370:                  Requires<[HasNonTinyEncoding]>;
1371: 
1372:   let Uses = [R17] in
1373:   def ROLBRdR17 : Pseudo<(outs GPR8:$rd), (ins GPR8:$src), "rolb\t$rd",
1374:                          [(set i8:$rd, (AVRrol i8:$src))]>,
1375:                   Requires<[HasTinyEncoding]>;
1376: 
1377:   def RORBRd : Pseudo<(outs GPR8:$rd), (ins GPR8:$src), "rorb\t$rd",
1378:                       [(set i8:$rd, (AVRror i8:$src))]>;
1379: 
1380:   // Bit rotate operations.
1381:   let Uses = [SREG] in {
1382:     def ROLWRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src), "rolw\t$rd",
1383:                         [(set i16:$rd, (AVRrol i16:$src))]>;
1384: 
1385:     def RORRd : FRd<0b1001, 0b0100111, (outs GPR8:$rd), (ins GPR8:$src),
1386:                     "ror\t$rd", []>;
```

- EN: This range defines declarative TableGen records such as ROLBRdR17, RORBRd, ROLWRd, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ROLBRdR17, RORBRd, ROLWRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1387-1404

```tablegen
1387: 
1388:     def RORWRd : Pseudo<(outs DREGS:$rd), (ins DREGS:$src), "rorw\t$rd",
1389:                         [(set i16:$rd, (AVRror i16:$src))]>;
1390:   }
1391: }
1392: 
1393: // SWAP Rd
1394: // Swaps the high and low nibbles in a register.
1395: let hasSideEffects = 0, Constraints = "$src = $rd" in
1396: def SWAPRd : FRd<0b1001, 0b0100010, (outs GPR8:$rd), (ins GPR8:$src),
1397:                  "swap\t$rd", [(set i8:$rd, (AVRSwap i8:$src))]>;
1398: 
1399: // IO register bit set/clear operations.
1400: //: TODO: add patterns when popcount(imm)==2 to be expanded with 2 sbi/cbi
1401: // instead of in+ori+out which requires one more instr.
1402: let hasSideEffects = 1, mayStore = 1 in {
1403:   def SBIAb : FIOBIT<0b10, (outs), (ins imm_port5:$addr, i8imm:$b),
1404:                      "sbi\t$addr, $b",
```

- EN: This range defines declarative TableGen records such as RORWRd, SWAPRd, SBIAb, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 RORWRd, SWAPRd, SBIAb 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1405-1422

```tablegen
1405:                      [(store(or(i8(load lowioaddr8:$addr)), iobitpos8:$b),
1406:                        lowioaddr8:$addr)]>;
1407: 
1408:   def CBIAb : FIOBIT<0b00, (outs), (ins imm_port5:$addr, i8imm :$b),
1409:                      "cbi\t$addr, $b",
1410:                      [(store(and(i8(load lowioaddr8:$addr)), iobitposn8:$b),
1411:                        lowioaddr8:$addr)]>;
1412: }
1413: 
1414: // Status register bit load/store operations.
1415: let Defs = [SREG], hasSideEffects = 1 in
1416: def BST : FRdB<0b01, (outs), (ins GPR8:$rd, i8imm:$b), "bst\t$rd, $b", []>;
1417: 
1418: let Constraints = "$src = $rd", Uses = [SREG], hasSideEffects = 0 in
1419: def BLD : FRdB<0b00, (outs GPR8:$rd), (ins GPR8:$src, i8imm:$b), "bld\t$rd, $b",
1420:                []>;
1421: 
1422: def CBR : InstAlias<"cbr\t$rd, $k", (ANDIRdK LD8 : $rd, imm_com8 : $k), 0>;
```

- EN: This range defines declarative TableGen records such as CBIAb, BST, BLD, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CBIAb, BST, BLD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1423-1440

```tablegen
1423: 
1424: // CLR Rd
1425: // Alias for EOR Rd, Rd
1426: // -------------
1427: // Clears all bits in a register.
1428: def CLR : InstAlias<"clr\t$rd", (EORRdRr GPR8 : $rd, GPR8 : $rd)>;
1429: 
1430: // LSL Rd
1431: // Alias for ADD Rd, Rd
1432: // --------------
1433: // Logical shift left one bit.
1434: def LSL : InstAlias<"lsl\t$rd", (ADDRdRr GPR8 : $rd, GPR8 : $rd)>;
1435: 
1436: def ROL : InstAlias<"rol\t$rd", (ADCRdRr GPR8 : $rd, GPR8 : $rd)>;
1437: 
1438: // SER Rd
1439: // Alias for LDI Rd, 0xff
1440: // ---------
```

- EN: This range defines declarative TableGen records such as CLR, LSL, ROL, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CLR, LSL, ROL 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 1441-1458

```tablegen
1441: // Sets all bits in a register.
1442: def : InstAlias<"ser\t$rd", (LDIRdK LD8 : $rd, 0xff), 0>;
1443: 
1444: let hasSideEffects = 1 in {
1445:   let Defs = [SREG] in def BSETs : FS<0, (outs), (ins i8imm:$s), "bset\t$s", []>;
1446:   let Defs = [SREG] in def BCLRs : FS<1, (outs), (ins i8imm:$s), "bclr\t$s", []>;
1447: }
1448: 
1449: // Set/clear aliases for the carry (C) status flag (bit 0).
1450: def : InstAlias<"sec", (BSETs 0)>;
1451: def : InstAlias<"clc", (BCLRs 0)>;
1452: 
1453: // Set/clear aliases for the zero (Z) status flag (bit 1).
1454: def : InstAlias<"sez", (BSETs 1)>;
1455: def : InstAlias<"clz", (BCLRs 1)>;
1456: 
1457: // Set/clear aliases for the negative (N) status flag (bit 2).
1458: def : InstAlias<"sen", (BSETs 2)>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1459-1476

```tablegen
1459: def : InstAlias<"cln", (BCLRs 2)>;
1460: 
1461: // Set/clear aliases for the overflow (V) status flag (bit 3).
1462: def : InstAlias<"sev", (BSETs 3)>;
1463: def : InstAlias<"clv", (BCLRs 3)>;
1464: 
1465: // Set/clear aliases for the signed (S) status flag (bit 4).
1466: def : InstAlias<"ses", (BSETs 4)>;
1467: def : InstAlias<"cls", (BCLRs 4)>;
1468: 
1469: // Set/clear aliases for the half-carry (H) status flag (bit 5).
1470: def : InstAlias<"seh", (BSETs 5)>;
1471: def : InstAlias<"clh", (BCLRs 5)>;
1472: 
1473: // Set/clear aliases for the T status flag (bit 6).
1474: def : InstAlias<"set", (BSETs 6)>;
1475: def : InstAlias<"clt", (BCLRs 6)>;
1476: 
```

- EN: This range continues the implementation of the backend component described by AVRInstrInfo.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1477-1494

```tablegen
1477: // Set/clear aliases for the interrupt (I) status flag (bit 7).
1478: def : InstAlias<"sei", (BSETs 7)>;
1479: def : InstAlias<"cli", (BCLRs 7)>;
1480: 
1481: //===----------------------------------------------------------------------===//
1482: // Special/Control instructions
1483: //===----------------------------------------------------------------------===//
1484: 
1485: // BREAK
1486: // Breakpoint instruction
1487: // ---------
1488: // <|1001|0101|1001|1000>
1489: let hasSideEffects = 1 in
1490: def BREAK : F16<0b1001010110011000, (outs), (ins), "break", []>,
1491:             Requires<[HasBREAK]>;
1492: 
1493: // NOP
1494: // No-operation instruction
```

- EN: This range defines declarative TableGen records such as BREAK, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BREAK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1495-1512

```tablegen
1495: // ---------
1496: // <|0000|0000|0000|0000>
1497: // Set `hasSideEffects = 1` in case it is considered dead.
1498: let hasSideEffects = 1 in
1499: def NOP : F16<0b0000000000000000, (outs), (ins), "nop", []>;
1500: 
1501: // SLEEP
1502: // Sleep instruction
1503: // ---------
1504: // <|1001|0101|1000|1000>
1505: let hasSideEffects = 1 in
1506: def SLEEP : F16<0b1001010110001000, (outs), (ins), "sleep", []>;
1507: 
1508: // WDR
1509: // Watchdog reset
1510: // ---------
1511: // <|1001|0101|1010|1000>
1512: let hasSideEffects = 1 in
```

- EN: This range defines declarative TableGen records such as NOP, SLEEP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 NOP, SLEEP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1513-1530

```tablegen
1513: def WDR : F16<0b1001010110101000, (outs), (ins), "wdr", []>;
1514: 
1515: //===----------------------------------------------------------------------===//
1516: // Pseudo instructions for later expansion
1517: //===----------------------------------------------------------------------===//
1518: 
1519: //: TODO: Optimize this for wider types AND optimize the following code
1520: //       compile int foo(char a, char b, char c, char d) {return d+b;}
1521: //       looks like a missed sext_inreg opportunity.
1522: def SEXT : ExtensionPseudo<(outs DREGS:$dt), (ins GPR8:$src), "sext\t$dt, $src",
1523:                            [(set i16:$dt, (sext i8:$src))]>;
1524: 
1525: def ZEXT : ExtensionPseudo<(outs DREGS:$dt), (ins GPR8:$src), "zext\t$dt, $src",
1526:                            [(set i16:$dt, (zext i8:$src))]>;
1527: 
1528: // This pseudo gets expanded into a movw+adiw thus it clobbers SREG.
1529: let Defs = [SREG], hasSideEffects = 0 in
1530: def FRMIDX : Pseudo<(outs DLDREGS:$dst), (ins DLDREGS:$src, i16imm:$src2),
```

- EN: This range defines declarative TableGen records such as WDR, SEXT, ZEXT, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 WDR, SEXT, ZEXT 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1531-1548

```tablegen
1531:                     "frmidx\t$dst, $src, $src2", []>;
1532: 
1533: // The instructions STDSPQRr and STDWSPQRr are used to store to the stack
1534: // frame. The most accurate implementation would be to load the SP into
1535: // a temporary pointer variable and then STDPtrQRr. However for efficiency,
1536: // we assume that R29R28 contains the current call frame pointer.
1537: // However in the PEI pass we sometimes rewrite a ADJCALLSTACKDOWN pseudo,
1538: // plus one or more STDSPQRr/STDWSPQRr pseudo instructions to use Z for a
1539: // stack adjustment then as a base pointer. To avoid corruption, we thus
1540: // specify special classes of registers, like GPR8 and DREGS, but with
1541: // the Z register removed, as the source/input to these instructions.
1542: // This pseudo is either converted to a regular store or a push which clobbers
1543: // SP.
1544: let Defs = [SP], Uses = [SP], hasSideEffects = 0, mayStore = 1 in
1545: def STDSPQRr : StorePseudo<(outs), (ins memspi:$dst, GPR8NOZ:$src),
1546:                            "stdstk\t$dst, $src", [(store i8:$src, addr:$dst)]>;
1547: 
1548: // See the comment on STDSPQRr.
```

- EN: This range defines declarative TableGen records such as STDSPQRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STDSPQRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1549-1566

```tablegen
1549: // This pseudo is either converted to a regular store or a push which clobbers
1550: // SP.
1551: let Defs = [SP], Uses = [SP], hasSideEffects = 0, mayStore = 1 in
1552: def STDWSPQRr : StorePseudo<(outs), (ins memspi:$dt, DREGSNOZ:$src),
1553:                             "stdwstk\t$dt, $src", [(store i16:$src, addr:$dt)]>;
1554: 
1555: // SP read/write pseudos.
1556: let hasSideEffects = 0 in {
1557:   let Uses = [SP], mayLoad = 1 in
1558:   def SPREAD : Pseudo<(outs DREGS:$dst), (ins GPRSP:$src), "spread\t$dst, $src",
1559:                       []>;
1560:   let Defs = [SP], mayStore = 1 in
1561:   def SPWRITE : Pseudo<(outs GPRSP:$dst), (ins DREGS:$src),
1562:                        "spwrite\t$dst, $src", []>;
1563: }
1564: 
1565: def Select8 : SelectPseudo<(outs GPR8:$dst),
1566:                            (ins GPR8:$src, GPR8:$src2, i8imm:$cc),
```

- EN: This range defines declarative TableGen records such as STDWSPQRr, SPREAD, SPWRITE, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STDWSPQRr, SPREAD, SPWRITE 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1567-1584

```tablegen
1567:                            "# Select8 PSEUDO",
1568:                            [(set i8:$dst,
1569:                              (AVRselectcc i8:$src, i8:$src2, imm:$cc))]>;
1570: 
1571: def Select16 : SelectPseudo<(outs DREGS:$dst),
1572:                             (ins DREGS:$src, DREGS:$src2, i8imm:$cc),
1573:                             "# Select16 PSEUDO",
1574:                             [(set i16:$dst,
1575:                               (AVRselectcc i16:$src, i16:$src2, imm:$cc))]>;
1576: 
1577: def Lsl8 : ShiftPseudo<(outs GPR8:$dst), (ins GPR8:$src, GPR8:$cnt),
1578:                        "# Lsl8 PSEUDO",
1579:                        [(set i8:$dst, (AVRlslLoop i8:$src, i8:$cnt))]>;
1580: 
1581: def Lsl16 : ShiftPseudo<(outs DREGS:$dst), (ins DREGS:$src, GPR8:$cnt),
1582:                         "# Lsl16 PSEUDO",
1583:                         [(set i16:$dst, (AVRlslLoop i16:$src, i8:$cnt))]>;
1584: 
```

- EN: This range defines declarative TableGen records such as Select16, Lsl8, Lsl16, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Select16, Lsl8, Lsl16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 1585-1602

```tablegen
1585: def Lsl32 : ShiftPseudo<(outs DREGS:$dstlo, DREGS:$dsthi),
1586:                         (ins DREGS:$srclo, DREGS:$srchi, i8imm:$cnt),
1587:                         "# Lsl32 PSEUDO",
1588:                         [(set i16:$dstlo, i16:$dsthi,
1589:                           (AVRlslw i16:$srclo, i16:$srchi, i8:$cnt))]>;
1590: 
1591: def Lsr8 : ShiftPseudo<(outs GPR8:$dst), (ins GPR8:$src, GPR8:$cnt),
1592:                        "# Lsr8 PSEUDO",
1593:                        [(set i8:$dst, (AVRlsrLoop i8:$src, i8:$cnt))]>;
1594: 
1595: def Lsr16 : ShiftPseudo<(outs DREGS:$dst), (ins DREGS:$src, GPR8:$cnt),
1596:                         "# Lsr16 PSEUDO",
1597:                         [(set i16:$dst, (AVRlsrLoop i16:$src, i8:$cnt))]>;
1598: 
1599: def Lsr32 : ShiftPseudo<(outs DREGS:$dstlo, DREGS:$dsthi),
1600:                         (ins DREGS:$srclo, DREGS:$srchi, i8imm:$cnt),
1601:                         "# Lsr32 PSEUDO",
1602:                         [(set i16:$dstlo, i16:$dsthi,
```

- EN: This range defines declarative TableGen records such as Lsl32, Lsr8, Lsr16, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Lsl32, Lsr8, Lsr16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 1603-1620

```tablegen
1603:                           (AVRlsrw i16:$srclo, i16:$srchi, i8:$cnt))]>;
1604: 
1605: def Rol8 : ShiftPseudo<(outs GPR8:$dst), (ins GPR8:$src, GPR8:$cnt),
1606:                        "# Rol8 PSEUDO",
1607:                        [(set i8:$dst, (AVRrolLoop i8:$src, i8:$cnt))]>;
1608: 
1609: def Rol16 : ShiftPseudo<(outs DREGS:$dst), (ins DREGS:$src, GPR8:$cnt),
1610:                         "# Rol16 PSEUDO",
1611:                         [(set i16:$dst, (AVRrolLoop i16:$src, i8:$cnt))]>;
1612: 
1613: def Ror8 : ShiftPseudo<(outs GPR8:$dst), (ins GPR8:$src, GPR8:$cnt),
1614:                        "# Ror8 PSEUDO",
1615:                        [(set i8:$dst, (AVRrorLoop i8:$src, i8:$cnt))]>;
1616: 
1617: def Ror16 : ShiftPseudo<(outs DREGS:$dst), (ins DREGS:$src, GPR8:$cnt),
1618:                         "# Ror16 PSEUDO",
1619:                         [(set i16:$dst, (AVRrorLoop i16:$src, i8:$cnt))]>;
1620: 
```

- EN: This range defines declarative TableGen records such as Rol8, Rol16, Ror8, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Rol8, Rol16, Ror8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 1621-1638

```tablegen
1621: def Asr8 : ShiftPseudo<(outs GPR8:$dst), (ins GPR8:$src, GPR8:$cnt),
1622:                        "# Asr8 PSEUDO",
1623:                        [(set i8:$dst, (AVRasrLoop i8:$src, i8:$cnt))]>;
1624: 
1625: def Asr16 : ShiftPseudo<(outs DREGS:$dst), (ins DREGS:$src, GPR8:$cnt),
1626:                         "# Asr16 PSEUDO",
1627:                         [(set i16:$dst, (AVRasrLoop i16:$src, i8:$cnt))]>;
1628: 
1629: def Asr32 : ShiftPseudo<(outs DREGS:$dstlo, DREGS:$dsthi),
1630:                         (ins DREGS:$srclo, DREGS:$srchi, i8imm:$cnt),
1631:                         "# Asr32 PSEUDO",
1632:                         [(set i16:$dstlo, i16:$dsthi,
1633:                           (AVRasrw i16:$srclo, i16:$srchi, i8:$cnt))]>;
1634: 
1635: // lowered to a copy from the zero register.
1636: let usesCustomInserter = 1, hasSideEffects = 0 in
1637: def CopyZero : Pseudo<(outs GPR8:$rd), (ins), "clrz\t$rd", [(set i8:$rd, 0)]>;
1638: 
```

- EN: This range defines declarative TableGen records such as Asr8, Asr16, Asr32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 Asr8, Asr16, Asr32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1639-1656

```tablegen
1639: //===----------------------------------------------------------------------===//
1640: // Non-Instruction Patterns
1641: //===----------------------------------------------------------------------===//
1642: 
1643: //: TODO: look in x86InstrCompiler.td for odd encoding trick related to
1644: // add x, 128 -> sub x, -128. Clang is emitting an eor for this (ldi+eor)
1645: 
1646: // the add instruction always writes the carry flag
1647: def : Pat<(addc i8 : $src, i8 : $src2), (ADDRdRr i8 : $src, i8 : $src2)>;
1648: def : Pat<(addc DREGS:$src, DREGS:$src2), (ADDWRdRr DREGS:$src, DREGS:$src2)>;
1649: 
1650: // all sub instruction variants always writes the carry flag
1651: def : Pat<(subc i8 : $src, i8 : $src2), (SUBRdRr i8 : $src, i8 : $src2)>;
1652: def : Pat<(subc i16 : $src, i16 : $src2), (SUBWRdRr i16 : $src, i16 : $src2)>;
1653: def : Pat<(subc i8 : $src, imm : $src2), (SUBIRdK i8 : $src, imm : $src2)>;
1654: def : Pat<(subc i16 : $src, imm : $src2), (SUBIWRdK i16 : $src, imm : $src2)>;
1655: 
1656: // These patterns convert add (x, -imm) to sub (x, imm) since we dont have
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1657-1674

```tablegen
1657: // any add with imm instructions. Also take care of the adiw/sbiw instructions.
1658: def : Pat<(add i16:$s1, imm0_63_neg:$s2), (SBIWRdK i16:$s1, (imm0_63_neg:$s2))>,
1659:       Requires<[HasADDSUBIW]>;
1660: 
1661: def : Pat<(add i16:$src1, imm:$src2),
1662:           (SUBIWRdK i16:$src1, (imm16_neg_XFORM imm:$src2))>;
1663: 
1664: def : Pat<(addc i16:$src1, imm:$src2),
1665:           (SUBIWRdK i16:$src1, (imm16_neg_XFORM imm:$src2))>;
1666: 
1667: def : Pat<(add i8:$src1, imm:$src2),
1668:           (SUBIRdK i8:$src1, (imm8_neg_XFORM imm:$src2))>;
1669: 
1670: def : Pat<(addc i8:$src1, imm:$src2),
1671:           (SUBIRdK i8:$src1, (imm8_neg_XFORM imm:$src2))>;
1672: 
1673: def : Pat<(adde i8:$src1, imm:$src2),
1674:           (SBCIRdK i8:$src1, (imm8_neg_XFORM imm:$src2))>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1675-1692

```tablegen
1675: 
1676: // Emit NEGWRd with an extra zero register operand.
1677: def : Pat<(ineg i16:$src), (NEGWRd i16:$src, (CopyZero))>;
1678: 
1679: // Calls.
1680: let Predicates = [HasJMPCALL] in {
1681:   def : Pat<(AVRcall(i16 tglobaladdr:$dst)), (CALLk tglobaladdr:$dst)>;
1682:   def : Pat<(AVRcall(i16 texternalsym:$dst)), (CALLk texternalsym:$dst)>;
1683: }
1684: def : Pat<(AVRcall(i16 tglobaladdr:$dst)), (RCALLk tglobaladdr:$dst)>;
1685: def : Pat<(AVRcall(i16 texternalsym:$dst)), (RCALLk texternalsym:$dst)>;
1686: 
1687: // `anyext`
1688: // FIXME: Using INSERT_SUBREG is more efficient, but leads to issue #132203.
1689: def : Pat<(i16(anyext i8:$src)),
1690: //        (INSERT_SUBREG(i16(IMPLICIT_DEF)), i8:$src, sub_lo)>;
1691:           (ZEXT i8:$src)>;
1692: 
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1693-1710

```tablegen
1693: // `trunc`
1694: def : Pat<(i8(trunc i16 : $src)), (EXTRACT_SUBREG i16 : $src, sub_lo)>;
1695: 
1696: // sext_inreg
1697: def : Pat<(sext_inreg i16:$s, i8), (SEXT(i8(EXTRACT_SUBREG i16:$s, sub_lo)))>;
1698: 
1699: // GlobalAddress
1700: def : Pat<(i16(AVRWrapper tglobaladdr : $dst)), (LDIWRdK tglobaladdr : $dst)>;
1701: def : Pat<(add i16:$src, (AVRWrapper tglobaladdr:$src2)),
1702:           (SUBIWRdK i16:$src, tglobaladdr:$src2)>;
1703: def : Pat<(i8(load(AVRWrapper tglobaladdr:$dst))),
1704:           (LDSRdK tglobaladdr:$dst)>,
1705:       Requires<[HasSRAM, HasNonTinyEncoding]>;
1706: def : Pat<(i8(load(AVRWrapper tglobaladdr:$dst))),
1707:           (LDSRdKTiny tglobaladdr:$dst)>,
1708:       Requires<[HasSRAM, HasTinyEncoding]>;
1709: def : Pat<(i16(load(AVRWrapper tglobaladdr:$dst))),
1710:           (LDSWRdK tglobaladdr:$dst)>,
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1711-1728

```tablegen
1711:       Requires<[HasSRAM, HasNonTinyEncoding]>;
1712: def : Pat<(store i8:$src, (i16(AVRWrapper tglobaladdr:$dst))),
1713:           (STSKRr tglobaladdr:$dst, i8:$src)>,
1714:       Requires<[HasSRAM, HasNonTinyEncoding]>;
1715: def : Pat<(store i8:$src, (i16(AVRWrapper tglobaladdr:$dst))),
1716:           (STSKRrTiny tglobaladdr:$dst, i8:$src)>,
1717:       Requires<[HasSRAM, HasTinyEncoding]>;
1718: def : Pat<(store i16:$src, (i16(AVRWrapper tglobaladdr:$dst))),
1719:           (STSWKRr tglobaladdr:$dst, i16:$src)>,
1720:       Requires<[HasSRAM, HasNonTinyEncoding]>;
1721: 
1722: // BlockAddress
1723: def : Pat<(i16(AVRWrapper tblockaddress :$dst)), (LDIWRdK tblockaddress:$dst)>;
1724: 
1725: def : Pat<(i8(trunc(AVRlsrwn DLDREGS:$src, (i16 8)))),
1726:           (EXTRACT_SUBREG DREGS:$src, sub_hi)>;
1727: 
1728: // :FIXME: DAGCombiner produces an shl node after legalization from these seq:
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1729-1738

```tablegen
1729: // BR_JT -> (mul x, 2) -> (shl x, 1)
1730: def : Pat<(shl i16 : $src1, (i8 1)), (LSLWRd i16 : $src1)>;
1731: 
1732: // Lowering of 'tst' node to 'TST' instruction.
1733: // TST is an alias of AND Rd, Rd.
1734: def : Pat<(AVRtst i8 : $rd), (ANDRdRr GPR8 : $rd, GPR8 : $rd)>;
1735: 
1736: // Lowering of 'lsl' node to 'LSL' instruction.
1737: // LSL is an alias of 'ADD Rd, Rd'
1738: def : Pat<(AVRlsl i8 : $rd), (ADDRdRr GPR8 : $rd, GPR8 : $rd)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- TableGen instruction records / TableGen 指令记录
- Encoding and patterns / 编码与匹配模式
- Register classes / 寄存器类
- CPU feature modelling / CPU 特性建模
- Pseudo-instruction handling / 伪指令处理
- Pattern matching / 模式匹配

## Dependencies / 依赖关系

- Direct includes / 直接包含: `AVRInstrFormats.td`
- LLVM subsystems / LLVM 子系统: TableGen
- Local companions / 本地配套文件: `AVRInstrInfo.h`, `AVRInstrInfo.cpp`
