# ARCInstrInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCInstrInfo.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file describes the ARC instructions in TableGen format.
- 目的（中文）: 使用 TableGen 定义目标指令，描述操作码、操作数、匹配模式与调度信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- ARCInstrInfo.td - Target Description for ARC --------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the ARC instructions in TableGen format.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: include "ARCInstrFormats.td"
  14: 
  15: //===----------------------------------------------------------------------===//
  16: // Operand Pattern Stuff.
  17: //===----------------------------------------------------------------------===//
  18: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 19-36

```tablegen
  19: // Operand for printing out a condition code.
  20: let PrintMethod = "printCCOperand" in
  21:   def CCOp : PredicateOperand<i32, (ops), (ops)>;
  22: 
  23: // The "u6" operand of a RRU6-type instruction
  24: let PrintMethod = "printU6" in {
  25:   def u6 : Operand<i32>, ImmLeaf<i32, [{
  26:     return isUInt<6>(Imm);
  27:   }]>;
  28:   def wide_u6 : Operand<i64>, ImmLeaf<i64, [{
  29:     return isUInt<6>(Imm);
  30:   }]>;
  31: }
  32: 
  33: // ---------------------------------------------------------------------------
  34: // Selection DAG Nodes.
  35: // ---------------------------------------------------------------------------
  36: 
```

- EN: This range defines declarative TableGen records such as CCOp, u6, wide_u6, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CCOp, u6, wide_u6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37: // Selection DAG types.
  38: def SDT_ARCcmptst : SDTypeProfile<0, 2, [SDTCisSameAs<0, 1>]>;
  39: def SDT_ARCcmov : SDTypeProfile<1, 3, [SDTCisSameAs<0, 1>]>;
  40: def SDT_ARCmov : SDTypeProfile<1, 1, [SDTCisSameAs<0, 1>]>;
  41: def SDT_ARCbrcc : SDTypeProfile<0, 4, []>;
  42: def SDT_ARCBranchLink : SDTypeProfile<0, 1, [SDTCisPtrTy<0>]>;
  43: def SDT_ARCCallSeqStart : SDCallSeqStart<[ SDTCisVT<0, i32>,
  44:                                            SDTCisVT<1, i32> ]>;
  45: def SDT_ARCCallSeqEnd   : SDCallSeqEnd<[ SDTCisVT<0, i32>,
  46:                                          SDTCisVT<1, i32> ]>;
  47: 
  48: // Global Address.
  49: def ARCGAWrapper : SDNode<"ARCISD::GAWRAPPER", SDT_ARCmov, []>;
  50: 
  51: // Comparison
  52: def ARCcmp : SDNode<"ARCISD::CMP", SDT_ARCcmptst, [SDNPOutGlue]>;
  53: 
  54: // Conditional mov
```

- EN: This range defines declarative TableGen records such as SDT_ARCcmptst, SDT_ARCcmov, SDT_ARCmov, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SDT_ARCcmptst, SDT_ARCcmov, SDT_ARCmov 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55: def ARCcmov : SDNode<"ARCISD::CMOV", SDT_ARCcmov, [SDNPInGlue]>;
  56: 
  57: // Conditional Branch
  58: def ARCbrcc : SDNode<"ARCISD::BRcc", SDT_ARCbrcc, [SDNPHasChain]>;
  59: 
  60: // Direct Call
  61: def ARCBranchLink     : SDNode<"ARCISD::BL",SDT_ARCBranchLink,
  62:                             [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
  63:                              SDNPVariadic]>;
  64: 
  65: // Indirect Call
  66: def ARCJumpLink       : SDNode<"ARCISD::JL",SDT_ARCBranchLink,
  67:                                  [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
  68:                                   SDNPVariadic]>;
  69: // Call return
  70: def ret      : SDNode<"ARCISD::RET", SDTNone,
  71:                       [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
  72: 
```

- EN: This range defines declarative TableGen records such as ARCcmov, ARCbrcc, ARCBranchLink, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ARCcmov, ARCbrcc, ARCBranchLink 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-90

```tablegen
  73: // Call sequencing nodes.
  74: // These are target-independent nodes, but have target-specific formats.
  75: def callseq_start : SDNode<"ISD::CALLSEQ_START", SDT_ARCCallSeqStart,
  76:                            [SDNPHasChain, SDNPOutGlue]>;
  77: def callseq_end   : SDNode<"ISD::CALLSEQ_END",   SDT_ARCCallSeqEnd,
  78:                            [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
  79: 
  80: //===----------------------------------------------------------------------===//
  81: // Instruction predicates
  82: //===----------------------------------------------------------------------===//
  83: 
  84: def HasNorm  :       Predicate<"Subtarget->hasNorm()">;
  85: 
  86: //===----------------------------------------------------------------------===//
  87: // Instruction Pattern Stuff
  88: //===----------------------------------------------------------------------===//
  89: 
  90: def imm32 : ImmLeaf<i32, [{
```

- EN: This range defines declarative TableGen records such as callseq_start, callseq_end, HasNorm, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 callseq_start, callseq_end, HasNorm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 91-108

```tablegen
  91:   return (Imm & 0xFFFFFFFF) == Imm;
  92: }]>;
  93: 
  94: // Addressing modes
  95: def FrameADDR_ri : ComplexPattern<i32, 2, "SelectFrameADDR_ri",
  96:                                   [add, frameindex], []>;
  97: def AddrModeS9 : ComplexPattern<i32, 2, "SelectAddrModeS9", []>;
  98: def AddrModeImm : ComplexPattern<i32, 2, "SelectAddrModeImm", []>;
  99: def AddrModeFar : ComplexPattern<i32, 2, "SelectAddrModeFar", []>;
 100: 
 101: //===----------------------------------------------------------------------===//
 102: // Instruction Class Templates
 103: //===----------------------------------------------------------------------===//
 104: 
 105: //===----------------------------------------------------------------------===//
 106: // Pseudo Instructions
 107: //===----------------------------------------------------------------------===//
 108: 
```

- EN: This range defines declarative TableGen records such as FrameADDR_ri, AddrModeS9, AddrModeImm, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 FrameADDR_ri, AddrModeS9, AddrModeImm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 109-126

```tablegen
 109: let Defs = [SP], Uses = [SP] in {
 110: def ADJCALLSTACKDOWN : PseudoInstARC<(outs), (ins i32imm:$amt, i32imm:$amt2),
 111:                                "# ADJCALLSTACKDOWN $amt, $amt2",
 112:                                [(callseq_start timm:$amt, timm:$amt2)]>;
 113: def ADJCALLSTACKUP : PseudoInstARC<(outs), (ins i32imm:$amt1, i32imm:$amt2),
 114:                             "# ADJCALLSTACKUP $amt1",
 115:                             [(callseq_end timm:$amt1, timm:$amt2)]>;
 116: }
 117: 
 118: def GETFI : PseudoInstARC<(outs GPR32:$dst), (ins MEMii:$addr),
 119:                              "pldfi $dst, $addr",
 120:                              [(set GPR32:$dst, FrameADDR_ri:$addr)]>;
 121: 
 122: 
 123: def ST_FAR : PseudoInstARC<(outs), (ins GPR32:$dst, MEMrlimm:$addr),
 124:                              "ST_FAR $dst, $addr",
 125:                              [(store GPR32:$dst, AddrModeFar:$addr)]>;
 126: 
```

- EN: This range defines declarative TableGen records such as ADJCALLSTACKDOWN, ADJCALLSTACKUP, GETFI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADJCALLSTACKDOWN, ADJCALLSTACKUP, GETFI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127: def STH_FAR : PseudoInstARC<(outs), (ins GPR32:$dst, MEMrlimm:$addr),
 128:                              "STH_FAR $dst, $addr",
 129:                              [(truncstorei16 GPR32:$dst, AddrModeFar:$addr)]>;
 130: 
 131: def STB_FAR : PseudoInstARC<(outs), (ins GPR32:$dst, MEMrlimm:$addr),
 132:                              "STB_FAR $dst, $addr",
 133:                              [(truncstorei8 GPR32:$dst, AddrModeFar:$addr)]>;
 134: 
 135: // TODO: Add `Requires<[HasBitScan]>` predicate to these when available.
 136: let Defs = [STATUS32] in {
 137:   def CTLZ : PseudoInstARC<(outs GPR32:$A),
 138:                            (ins GPR32:$B),
 139:                            "error.fls $A, $B",
 140:                            [(set GPR32:$A, (ctlz i32:$B))]>;
 141: 
 142:   def CTTZ : PseudoInstARC<(outs GPR32:$A),
 143:                            (ins GPR32:$B),
 144:                            "error.ffs $A, $B",
```

- EN: This range defines declarative TableGen records such as STH_FAR, STB_FAR, CTLZ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STH_FAR, STB_FAR, CTLZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 145-162

```tablegen
 145:                            [(set GPR32:$A, (cttz i32:$B))]>;
 146: }
 147: 
 148: //===----------------------------------------------------------------------===//
 149: // Instruction Generation multiclasses.
 150: // Generate many variants of a single instruction with a single defining
 151: // multiclass.  These classes do not contain Selection DAG patterns.
 152: //===----------------------------------------------------------------------===//
 153: 
 154: // Generic 3 operand binary instructions (i.e., add r0, r1, r2).
 155: multiclass ArcBinaryInst<bits<5> major, bits<6> mincode,
 156:                        string opasm, bit Commutable> {
 157:   // 3 register variant.
 158:   def _rrr : F32_DOP_RR<major, mincode, 0, (outs GPR32:$A),
 159:                         (ins GPR32:$B, GPR32:$C),
 160:                         !strconcat(opasm, "\t$A, $B, $C"),
 161:                         []>
 162:   { let isCommutable = Commutable; }
```

- EN: This range defines declarative TableGen records such as ArcBinaryInst, _rrr, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ArcBinaryInst, _rrr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 163-180

```tablegen
 163:   def _f_rrr : F32_DOP_RR<major, mincode, 1, (outs GPR32:$A),
 164:                           (ins GPR32:$B, GPR32:$C),
 165:                           !strconcat(opasm, ".f\t$A, $B, $C"),
 166:                           []>
 167:   { let Defs = [STATUS32]; }
 168: 
 169:   // 2 register with unsigned 6-bit immediate variant.
 170:   def _rru6 : F32_DOP_RU6<major, mincode, 0, (outs GPR32:$A),
 171:                           (ins GPR32:$B, immU6:$U6),
 172:                           !strconcat(opasm, "\t$A, $B, $U6"),
 173:                           []>;
 174:   def _f_rru6 : F32_DOP_RU6<major, mincode, 1, (outs GPR32:$A),
 175:                             (ins GPR32:$B, immU6:$U6),
 176:                             !strconcat(opasm, ".f\t$A, $B, $U6"),
 177:                             []>
 178:   { let Defs = [STATUS32]; }
 179: 
 180:   def _cc_rru6 : F32_DOP_CC_RRU6<major, mincode, 0, (outs GPR32:$A),
```

- EN: This range defines declarative TableGen records such as _f_rrr, _rru6, _f_rru6, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 _f_rrr, _rru6, _f_rru6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 181-198

```tablegen
 181:                                  (ins immU6:$U6, ccond:$cc, GPR32:$B),
 182:                                  !strconcat(opasm, ".$cc\t$A, $B, $U6"),
 183:                                  []> {
 184:                                    let Uses = [STATUS32];
 185:                                    let Constraints = "$A = $B";
 186:                                  }
 187: 
 188:   def _cc_f_rru6 : F32_DOP_CC_RRU6<major, mincode, 1, (outs GPR32:$A),
 189:                                    (ins immU6:$U6, ccond:$cc, GPR32:$B),
 190:                                    !strconcat(opasm, ".$cc.f\t$A, $B, $U6"),
 191:                                    []> {
 192:                                      let Defs = [STATUS32];
 193:                                      let Uses = [STATUS32];
 194:                                      let Constraints = "$A = $B";
 195:                                    }
 196: 
 197:   // 2 register with 32-bit immediate variant.
 198:   def _rrlimm : F32_DOP_RLIMM<major, mincode, 0,
```

- EN: This range defines declarative TableGen records such as _cc_f_rru6, _rrlimm, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 _cc_f_rru6, _rrlimm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-216

```tablegen
 199:                               (outs GPR32:$A),
 200:                               (ins GPR32:$B, i32imm:$LImm),
 201:                               !strconcat(opasm, "\t$A, $B, $LImm"),
 202:                               []>;
 203:   def _f_rrlimm : F32_DOP_RLIMM<major, mincode, 1,
 204:                                 (outs GPR32:$A),
 205:                                 (ins GPR32:$B, i32imm:$LImm),
 206:                                 !strconcat(opasm, ".f\t$A, $B, $LImm"),
 207:                                 []>
 208:   { let Defs = [STATUS32]; }
 209: 
 210:   // 2 matched-register with signed 12-bit immediate variant (add r0, r0, -1).
 211:   def _rrs12 : F32_DOP_RS12<major, mincode, 0,
 212:                             (outs GPR32:$B),
 213:                             (ins GPR32:$in, immS<12>:$S12),
 214:                             !strconcat(opasm, "\t$B, $in, $S12"),
 215:                             []>
 216:   { let Constraints = "$B = $in"; }
```

- EN: This range defines declarative TableGen records such as _f_rrlimm, _rrs12, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 _f_rrlimm, _rrs12 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 217-234

```tablegen
 217:   def _f_rrs12 : F32_DOP_RS12<major, mincode, 1,
 218:                               (outs GPR32:$B),
 219:                               (ins GPR32:$in, immS<12>:$S12),
 220:                               !strconcat(opasm, ".f\t$B, $in, $S12"),
 221:                               []>
 222:   { let Constraints = "$B = $in"; let Defs = [STATUS32]; }
 223: }
 224: 
 225: // Special multivariant GEN4 DOP format instruction that take 2 registers.
 226: // This is the class that is used for various comparison instructions.
 227: multiclass ArcSpecialDOPInst<bits<6> subop, string opasm, bit F> {
 228:   def _rr : F32_DOP_RR<0b00100, subop, F, (outs), (ins GPR32:$B, GPR32:$C),
 229:                !strconcat(opasm, "\t$B, $C"),
 230:                []>;
 231: 
 232:   def _ru6 : F32_DOP_RU6<0b00100, subop, F, (outs), (ins GPR32:$B, i32imm:$U6),
 233:                !strconcat(opasm, "\t$B, $U6"),
 234:                []>;
```

- EN: This range defines declarative TableGen records such as _f_rrs12, ArcSpecialDOPInst, _rr, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 _f_rrs12, ArcSpecialDOPInst, _rr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 235-252

```tablegen
 235: 
 236:   def _rlimm : F32_DOP_RLIMM<0b00100, subop, F, (outs),
 237:                (ins GPR32:$B, i32imm:$LImm),
 238:                !strconcat(opasm, "\t$B, $LImm"),
 239:                []>;
 240: }
 241: 
 242: // Generic 2-operand unary instructions.
 243: multiclass ArcUnaryInst<bits<5> major, bits<6> subop,
 244:                         string opasm> {
 245:   def _rr : F32_SOP_RR<major, subop, 0, (outs GPR32:$B), (ins GPR32:$C),
 246:                        !strconcat(opasm, "\t$B, $C"), []>;
 247: 
 248:   def _f_rr : F32_SOP_RR<major, subop, 1, (outs GPR32:$B), (ins GPR32:$C),
 249:                        !strconcat(opasm, ".f\t$B, $C"), []>
 250:   { let Defs = [STATUS32]; }
 251: }
 252: 
```

- EN: This range defines declarative TableGen records such as _rlimm, ArcUnaryInst, _rr, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 _rlimm, ArcUnaryInst, _rr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 253-270

```tablegen
 253: 
 254: multiclass ArcBinaryGEN4Inst<bits<6> mincode, string opasm, bit Commutable = 0> :
 255:   ArcBinaryInst<0b00100, mincode, opasm, Commutable>;
 256: multiclass ArcBinaryEXT5Inst<bits<6> mincode, string opasm> :
 257:   ArcBinaryInst<0b00101, mincode, opasm, 0>;
 258: 
 259: multiclass ArcUnaryGEN4Inst<bits<6> mincode, string opasm> :
 260:   ArcUnaryInst<0b00100, mincode, opasm>;
 261: multiclass ArcUnaryEXT5Inst<bits<6> mincode, string opasm> :
 262:   ArcUnaryInst<0b00101, mincode, opasm>;
 263: 
 264: // Pattern generation for different instruction variants.
 265: multiclass MultiPat<SDPatternOperator InFrag,
 266:                Instruction RRR, Instruction RRU6, Instruction RRLImm> {
 267:   def _rrr : Pat<(InFrag i32:$B, i32:$C), (RRR i32:$B, i32:$C)>;
 268:   def _rru6 : Pat<(InFrag i32:$B, immU6:$U6), (RRU6 i32:$B, immU6:$U6)>;
 269:   def _rrlimm : Pat<(InFrag i32:$B, imm32:$LImm), (RRLImm i32:$B, imm32:$LImm)>;
 270: }
```

- EN: This range defines declarative TableGen records such as ArcBinaryGEN4Inst, ArcBinaryEXT5Inst, ArcUnaryGEN4Inst, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 ArcBinaryGEN4Inst, ArcBinaryEXT5Inst, ArcUnaryGEN4Inst 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 271-288

```tablegen
 271: 
 272: // NOTE: This could be specialized later with a custom `PrintMethod` for
 273: //       displaying the aux register name. E.g. `[%count0]` instead of [33].
 274: def AuxReg : Operand<i32>;
 275: 
 276: def LR_rs12 : F32_SOP_RS12<0b00100, 0b101010, 0,
 277:                            (outs GPR32:$B), (ins AuxReg:$C),
 278:                            "lr\t$B, [$C]", []>;
 279: def LR_ru6 : F32_SOP_RU6<0b00100, 0b101010, 0,
 280:                          (outs GPR32:$B), (ins AuxReg:$C),
 281:                          "lr\t$B, [$C]", []>;
 282: 
 283: def: Pat<(i32 readcyclecounter), (LR_rs12 0x21) >;  // read timer
 284: 
 285: // ---------------------------------------------------------------------------
 286: // Instruction definitions and patterns for 3 operand binary instructions.
 287: // ---------------------------------------------------------------------------
 288: 
```

- EN: This range defines declarative TableGen records such as AuxReg, LR_rs12, LR_ru6, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 AuxReg, LR_rs12, LR_ru6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 289-306

```tablegen
 289: // Definitions for 3 operand binary instructions.
 290: defm ADD : ArcBinaryGEN4Inst<0b000000, "add",1>;
 291: defm SUB : ArcBinaryGEN4Inst<0b000010, "sub">;
 292: defm SUB1 : ArcBinaryGEN4Inst<0b010111, "sub1">;
 293: defm SUB2 : ArcBinaryGEN4Inst<0b011000, "sub2">;
 294: defm SUB3 : ArcBinaryGEN4Inst<0b011001, "sub3">;
 295: defm RSUB : ArcBinaryGEN4Inst<0b001110, "rsub">;
 296: defm OR  : ArcBinaryGEN4Inst<0b000101, "or",1>;
 297: defm AND : ArcBinaryGEN4Inst<0b000100, "and",1>;
 298: defm XOR : ArcBinaryGEN4Inst<0b000111, "xor",1>;
 299: defm MAX : ArcBinaryGEN4Inst<0b001000, "max",1>;
 300: defm MIN : ArcBinaryGEN4Inst<0b001001, "min",1>;
 301: defm ASL : ArcBinaryEXT5Inst<0b000000, "asl">;
 302: defm LSR : ArcBinaryEXT5Inst<0b000001, "lsr">;
 303: defm ASR : ArcBinaryEXT5Inst<0b000010, "asr">;
 304: defm ROR : ArcBinaryEXT5Inst<0b000011, "ror">;
 305: defm MPY  : ArcBinaryGEN4Inst<0b011010, "mpy",1>;
 306: defm MPYM : ArcBinaryGEN4Inst<0b011011, "mpym",1>;
```

- EN: This range defines declarative TableGen records such as ADD, SUB, SUB1, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ADD, SUB, SUB1 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 307-324

```tablegen
 307: defm MPYMU : ArcBinaryGEN4Inst<0b011100, "mpymu",1>;
 308: defm SETEQ : ArcBinaryGEN4Inst<0b111000, "seteq",1>;
 309: let Uses=[STATUS32], isAsCheapAsAMove=0, isReMaterializable=0 in {
 310:   defm ADC : ArcBinaryGEN4Inst<0b000001, "adc",1>;
 311:   defm SBC : ArcBinaryGEN4Inst<0b000011, "sbc">;
 312: }
 313: 
 314: // Patterns for 3 operand binary instructions.
 315: defm : MultiPat<add, ADD_rrr, ADD_rru6, ADD_rrlimm>;
 316: defm : MultiPat<sub, SUB_rrr, SUB_rru6, SUB_rrlimm>;
 317: defm : MultiPat<or, OR_rrr, OR_rru6, OR_rrlimm>;
 318: defm : MultiPat<and, AND_rrr, AND_rru6, AND_rrlimm>;
 319: defm : MultiPat<xor, XOR_rrr, XOR_rru6, XOR_rrlimm>;
 320: defm : MultiPat<smax, MAX_rrr, MAX_rru6, MAX_rrlimm>;
 321: defm : MultiPat<smin, MIN_rrr, MIN_rru6, MIN_rrlimm>;
 322: defm : MultiPat<shl, ASL_rrr, ASL_rru6, ASL_rrlimm>;
 323: defm : MultiPat<srl, LSR_rrr, LSR_rru6, LSR_rrlimm>;
 324: defm : MultiPat<sra, ASR_rrr, ASR_rru6, ASR_rrlimm>;
```

- EN: This range defines declarative TableGen records such as MPYMU, SETEQ, ADC, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 MPYMU, SETEQ, ADC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 325-342

```tablegen
 325: defm : MultiPat<rotr, ROR_rrr, ROR_rru6, ROR_rrlimm>;
 326: defm : MultiPat<mul, MPY_rrr, MPY_rru6, MPY_rrlimm>;
 327: defm : MultiPat<mulhs, MPYM_rrr, MPYM_rru6, MPYM_rrlimm>;
 328: defm : MultiPat<mulhu, MPYMU_rrr, MPYMU_rru6, MPYMU_rrlimm>;
 329: 
 330: defm : MultiPat<addc, ADD_f_rrr, ADD_f_rru6, ADD_f_rrlimm>;
 331: defm : MultiPat<adde, ADC_f_rrr, ADC_f_rru6, ADC_f_rrlimm>;
 332: defm : MultiPat<subc, SUB_f_rrr, SUB_f_rru6, SUB_f_rrlimm>;
 333: defm : MultiPat<sube, SBC_f_rrr, SBC_f_rru6, SBC_f_rrlimm>;
 334: 
 335: // ---------------------------------------------------------------------------
 336: // Unary Instruction definitions.
 337: // ---------------------------------------------------------------------------
 338: // General unary instruction definitions.
 339: defm SEXB : ArcUnaryGEN4Inst<0b000101, "sexb">;
 340: defm SEXH : ArcUnaryGEN4Inst<0b000110, "sexh">;
 341: 
 342: // Extension unary instruction definitions.
```

- EN: This range defines declarative TableGen records such as SEXB, SEXH, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 SEXB, SEXH 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 343-360

```tablegen
 343: defm FFS : ArcUnaryEXT5Inst<0b010010, "ffs">;
 344: defm FLS : ArcUnaryEXT5Inst<0b010011, "fls">;
 345: 
 346: let Predicates=[HasNorm] in {
 347:   defm NORM  : ArcUnaryEXT5Inst<0b000001,"norm">;
 348:   defm NORMH : ArcUnaryEXT5Inst<0b001000,"normh">;
 349: }
 350: 
 351: // General Unary Instruction fragments.
 352: def : Pat<(sext_inreg i32:$a, i8), (SEXB_rr i32:$a)>;
 353: def : Pat<(sext_inreg i32:$a, i16), (SEXH_rr i32:$a)>;
 354: 
 355: // Comparison instruction definition
 356: let isCompare = 1, Defs = [STATUS32] in {
 357: defm CMP : ArcSpecialDOPInst<0b001100, "cmp", 1>;
 358: }
 359: 
 360: def cmp : PatFrag<(ops node:$op1, node:$op2), (ARCcmp $op1, $op2)>;
```

- EN: This range defines declarative TableGen records such as FFS, FLS, NORM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 FFS, FLS, NORM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 361-378

```tablegen
 361: defm : MultiPat<cmp, CMP_rr, CMP_ru6, CMP_rlimm>;
 362: 
 363: // ---------------------------------------------------------------------------
 364: // MOV instruction and variants (conditional mov).
 365: // ---------------------------------------------------------------------------
 366: let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1 in {
 367: def MOV_rs12 : F32_DOP_RS12<0b00100, 0b001010, 0,
 368:                  (outs GPR32:$B), (ins immS<12>:$S12),
 369:                  "mov\t$B, $S12",
 370:                  [(set GPR32:$B, immS<12>:$S12)]>;
 371: }
 372: 
 373: def MOV_rr : F32_DOP_RR<0b00100, 0b001010, 0,
 374:                 (outs GPR32:$B), (ins GPR32:$C),
 375:                 "mov\t$B, $C", []>;
 376: 
 377: def MOV_rlimm : F32_DOP_RLIMM<0b00100, 0b001010, 0,
 378:                       (outs GPR32:$B), (ins i32imm:$LImm),
```

- EN: This range defines declarative TableGen records such as MOV_rs12, MOV_rr, MOV_rlimm, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 MOV_rs12, MOV_rr, MOV_rlimm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 379-396

```tablegen
 379:                       "mov\t$B, $LImm", []>;
 380: 
 381: def MOV_ru6 : F32_DOP_RU6<0b00100, 0b001010, 0,
 382:                           (outs GPR32:$B), (ins immU6:$U6),
 383:                           "mov\t$B, $U6", []>;
 384: 
 385: def MOV_f_ru6 : F32_DOP_RU6<0b00100, 0b001010, 1,
 386:                           (outs GPR32:$B), (ins u6:$U6),
 387:                           "mov.f\t$B, $U6", []> {
 388:   let isAsCheapAsAMove=1;
 389:   let Defs = [STATUS32];
 390: }
 391: 
 392: def cmov : PatFrag<(ops node:$op1, node:$op2, node:$cc),
 393:                    (ARCcmov $op1, $op2, $cc)>;
 394: let Uses = [STATUS32], isAsCheapAsAMove = 1, isPredicable=1,
 395: 	  isReMaterializable = 0, Constraints = "$B = $B2" in {
 396:   def MOV_cc : F32_DOP_CC_RR<0b00100, 0b001010, 0,
```

- EN: This range defines declarative TableGen records such as MOV_ru6, MOV_f_ru6, cmov, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MOV_ru6, MOV_f_ru6, cmov 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 397-414

```tablegen
 397:                  (outs GPR32:$B), (ins GPR32:$C, GPR32:$B2, CCOp:$cc),
 398:                  "mov.$cc\t$B, $C",
 399:                  [(set GPR32:$B, (cmov i32:$C, i32:$B2, timm:$cc))]>;
 400: 
 401:   def MOV_cc_ru6 : F32_DOP_CC_RU6<0b00100, 0b001010, 0,
 402:                  (outs GPR32:$B), (ins u6:$C, CCOp:$cc, GPR32:$B2),
 403:                  "mov.$cc\t$B, $C", []>;
 404: 
 405:   def MOV_cc_f_ru6 : F32_DOP_CC_RU6<0b00100, 0b001010, 1,
 406:                     (outs GPR32:$B), (ins u6:$C, CCOp:$cc, GPR32:$B2),
 407:                     "mov.$cc.f\t$B, $C", []> {
 408:     let Defs = [STATUS32];
 409:   }
 410: }
 411: 
 412: def : Pat<(ARCGAWrapper tglobaladdr:$addr),
 413:            (MOV_rlimm tglobaladdr:$addr)>;
 414: 
```

- EN: This range defines declarative TableGen records such as MOV_cc_ru6, MOV_cc_f_ru6, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 MOV_cc_ru6, MOV_cc_f_ru6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 415-432

```tablegen
 415: def : Pat<(ARCGAWrapper tjumptable:$addr),
 416:            (MOV_rlimm tjumptable:$addr)>;
 417: 
 418: 
 419: // ---------------------------------------------------------------------------
 420: // Control flow instructions (branch, return, calls, etc).
 421: // ---------------------------------------------------------------------------
 422: 
 423: // Branch instructions
 424: let isBranch = 1, isTerminator = 1 in {
 425: 
 426:   // Unconditional branch.
 427:   let isBarrier = 1 in
 428:   def BR : F32_BR0_UCOND_FAR<(outs), (ins btargetS25:$S25),
 429:                              "b\t$S25", [(br bb:$S25)]>;
 430: 
 431:   let Uses=[STATUS32] in
 432:   // Conditional branch.
```

- EN: This range defines declarative TableGen records such as BR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 433-450

```tablegen
 433:   def Bcc : F32_BR0_COND<(outs), (ins btargetS21:$S21, ccond:$cc),
 434:                          "b$cc\t$S21", []>;
 435: 
 436:   // Compare and branch (limited range).
 437:   def BRcc_rr  : F32_BR1_BCC<(outs),
 438:                              (ins btargetS9:$S9, GPR32:$B, GPR32:$C, brccond:$cc),
 439:                              "br$cc\t$B, $C, $S9", 0, []>;
 440:   def BRcc_ru6 : F32_BR1_BCC<(outs),
 441:                              (ins btargetS9:$S9, GPR32:$B, immU6:$C, brccond:$cc),
 442:                              "br$cc\t$B, $C, $S9", 1, []>;
 443: 
 444:   // Pseudo compare and branch.
 445:   // After register allocation, this can expand into either a limited range
 446:   // Compare and branch (BRcc), or into CMP + Bcc.
 447:   // At worst, this expands into 2 4-byte instructions.
 448:   def BRcc_rr_p : PseudoInstARC<(outs),
 449:                                 (ins btarget:$T, GPR32:$B, GPR32:$C, ccond:$cc),
 450:                                 "pbr$cc\t$B, $C, $T",
```

- EN: This range defines declarative TableGen records such as Bcc, BRcc_rr, BRcc_ru6, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Bcc, BRcc_rr, BRcc_ru6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 451-468

```tablegen
 451:                                 [(ARCbrcc bb:$T, i32:$B, i32:$C, imm32:$cc)]>
 452:                                 { let Size = 8; }
 453: 
 454:   def BRcc_ru6_p : PseudoInstARC<(outs),
 455:                                  (ins btarget:$T, GPR32:$B, i32imm:$C, ccond:$cc),
 456:                                  "pbr$cc\t$B, $C, $T",
 457:                                  [(ARCbrcc bb:$T, i32:$B, immU6:$C, imm32:$cc)]>
 458:                                  { let Size = 8; }
 459: } // let isBranch, isTerminator
 460: 
 461: // Unconditional Jump.
 462: let isBranch = 1, isTerminator = 1, isBarrier = 1 in {
 463:   // Indirect.
 464:   let isIndirectBranch = 1 in
 465:   def J :  F32_DOP_RR<0b00100, 0b100000, 0,
 466:                       (outs), (ins GPR32:$C),
 467:                       "j\t[$C]", [(brind i32:$C)]>;
 468: 
```

- EN: This range defines declarative TableGen records such as BRcc_ru6_p, J, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BRcc_ru6_p, J 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 469-486

```tablegen
 469:   // Direct.
 470:   def J_LImm : F32_DOP_RLIMM<0b00100, 0b100000, 0,
 471:                              (outs), (ins i32imm:$LImm),
 472:                              "j\t$LImm", []>;
 473: }
 474: 
 475: // Call instructions.
 476: let isCall = 1, isBarrier = 1, Defs = [BLINK], Uses = [SP] in {
 477:   // Direct unconditional call.
 478:   def BL : F32_BR1_BL_UCOND_FAR<(outs), (ins calltargetS25:$S25),
 479:                       "bl\t$S25", [(ARCBranchLink tglobaladdr:$S25)]>;
 480: 
 481:   // Indirect unconditional call.
 482:   let isIndirectBranch = 1 in
 483:   def JL : F32_DOP_RR<0b00100, 0b100010, 0, (outs), (ins GPR32:$C),
 484:                      "jl\t[$C]", [(ARCJumpLink i32:$C)]>;
 485: 
 486:   // Direct unconditional call.
```

- EN: This range defines declarative TableGen records such as J_LImm, BL, JL, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 J_LImm, BL, JL 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 487-504

```tablegen
 487:   def JL_LImm : F32_DOP_RLIMM<0b00100, 0b100010, 0, (outs), (ins i32imm:$LImm),
 488:                               "jl\t$LImm", []>;
 489: } // let isCall, isBarrier, Defs, Uses
 490: 
 491: // Pattern to generate BL instruction.
 492: def : Pat<(ARCBranchLink texternalsym:$dst), (BL texternalsym:$dst)>;
 493: 
 494: // Return from call.
 495: let isReturn = 1, isTerminator = 1, isBarrier = 1  in
 496: // This is a specialized 2-byte instruction that doesn't generalize
 497: // to any larger 2-byte class, so go ahead and define it here.
 498: def J_S_BLINK : InstARC<2, (outs), (ins), "j_s\t[%blink]", [(ret)]> {
 499:   let Inst{15-0} = 0b0111111011100000;
 500: }
 501: 
 502: //----------------------------------------------------------------------------
 503: // Compact stack-based operations.
 504: //----------------------------------------------------------------------------
```

- EN: This range defines declarative TableGen records such as JL_LImm, J_S_BLINK, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 JL_LImm, J_S_BLINK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 505-522

```tablegen
 505: 
 506: // 2-byte push/pop blink instructions commonly used for prolog/epilog
 507: // generation.  These 2 instructions are actually specialized 2-byte
 508: // format instructions that aren't generalized to a larger 2-byte
 509: // class, so we might as well have them here.
 510: let Uses = [BLINK], Defs = [SP] in
 511: def PUSH_S_BLINK : F16_SP_OPS_buconst<0b111, "push_s">;
 512: 
 513: let Defs = [BLINK, SP] in
 514: def POP_S_BLINK : F16_SP_OPS_buconst<0b110, "pop_s">;
 515: 
 516: def PUSH_S_r : F16_SP_OPS_uconst<0b110,
 517:   (outs), (ins GPR32Reduced:$b3), "push_s">;
 518: def POP_S_r : F16_SP_OPS_uconst<0b111,
 519:   (outs GPR32Reduced:$b3), (ins), "pop_s">;
 520: 
 521: def SP_SUB_SP_S : F16_SP_OPS_bconst<0b001, "sub_s">;
 522: def SP_ADD_SP_S : F16_SP_OPS_bconst<0b000, "add_s">;
```

- EN: This range defines declarative TableGen records such as PUSH_S_BLINK, POP_S_BLINK, PUSH_S_r, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 PUSH_S_BLINK, POP_S_BLINK, PUSH_S_r 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 523-540

```tablegen
 523: def SP_ADD_S : F16_SP_OPS_u7_aligned<0b100,
 524:                 (outs GPR32Reduced:$b3), (ins immU<7>:$u7),
 525:                 "add_s\t$b3, %sp, $u7">;
 526: 
 527: def SP_LD_S : F16_SP_LD<0b000, "ld_s">;
 528: def SP_LDB_S : F16_SP_LD<0b001, "ldb_s">;
 529: def SP_ST_S : F16_SP_ST<0b010, "st_s">;
 530: def SP_STB_S : F16_SP_ST<0b011, "stb_s">;
 531: 
 532: def LEAVE_S : F16_SP_OPS<0b110,
 533:   (outs), (ins immU<7>:$u7), "leave_s\t$u7"> {
 534: 
 535:   bits<7> u7;
 536: 
 537:   let fieldB = u7{6-4};
 538:   let fieldU{4-1} = u7{3-0};
 539:   let fieldU{0} = 0b0;
 540: }
```

- EN: This range defines declarative TableGen records such as SP_ADD_S, SP_LD_S, SP_LDB_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SP_ADD_S, SP_LD_S, SP_LDB_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 541-558

```tablegen
 541: 
 542: def ENTER_S : F16_SP_OPS<0b111,
 543:   (outs), (ins immU<6>:$u6), "enter_s\t$u6"> {
 544: 
 545:   bits<6> u6;
 546: 
 547:   let fieldB{2} = 0;
 548:   let fieldB{1-0} = u6{5-4};
 549:   let fieldU{4-1} = u6{3-0};
 550:   let fieldU{0} = 0b0;
 551: }
 552: 
 553: //----------------------------------------------------------------------------
 554: // Compact Move/Load instructions.
 555: //----------------------------------------------------------------------------
 556: class COMPACT_MOV_S :
 557:   F16_COMPACT<0b0, (outs GPR32:$g), (ins GPR32:$h),
 558:           "mov_s\t$g, $h"> {
```

- EN: This range defines declarative TableGen records such as ENTER_S, COMPACT_MOV_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ENTER_S, COMPACT_MOV_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 559-576

```tablegen
 559:   let DecoderMethod = "DecodeMoveHRegInstruction";
 560: }
 561: 
 562: def COMPACT_MOV_S_limm : COMPACT_MOV_S {
 563:   bits<32> LImm;
 564:   let Inst{47-16} = LImm;
 565: 
 566:   bits<5> LImmReg = 0b11110;
 567:   let Inst{7-5} = LImmReg{2-0};
 568:   let Inst{1-0} = LImmReg{4-3};
 569: 
 570:   let Size = 6;
 571: }
 572: 
 573: def COMPACT_MOV_S_hreg : COMPACT_MOV_S;
 574: 
 575: def COMPACT_LD_S :
 576:   F16_COMPACT<0b1, (outs GPR32:$r), (ins GPR32:$h, immU<5>:$u5),
```

- EN: This range defines declarative TableGen records such as COMPACT_MOV_S_limm, COMPACT_MOV_S_hreg, COMPACT_LD_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 COMPACT_MOV_S_limm, COMPACT_MOV_S_hreg, COMPACT_LD_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 577-594

```tablegen
 577:           "ld_s\t$r, [$h, $u5]"> {
 578:   bits<5> u5;
 579:   bits<2> r;
 580: 
 581:   let Inst{10} = u5{4};
 582:   let Inst{9-8} = r;
 583:   let Inst{4-3} = u5{3-2};
 584:   let u5{1-0} = 0b00;
 585: }
 586: 
 587: //----------------------------------------------------------------------------
 588: // Compact Load/Add/Sub.
 589: //----------------------------------------------------------------------------
 590: def LD_S_AS_rrr : F16_LD_SUB<0b0, "ld_s.as\t$a, [$b, $c]">;
 591: def SUB_S_rrr : F16_LD_SUB<0b1, "sub_s\t$a, $b, $c">;
 592: def ADD_S_rru6 : F16_ADD;
 593: 
 594: //----------------------------------------------------------------------------
```

- EN: This range defines declarative TableGen records such as LD_S_AS_rrr, SUB_S_rrr, ADD_S_rru6, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LD_S_AS_rrr, SUB_S_rrr, ADD_S_rru6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 595-612

```tablegen
 595: // Compact Load/Store.
 596: //----------------------------------------------------------------------------
 597: def LD_S_s11 : F16_LD_ST_s11<0b0, "ld_s\t%r1, [%gp, $s11]">;
 598: def ST_S_s11 : F16_LD_ST_s11<0b1, "st_s\t%r0, [%gp, $s11]">;
 599: def LDI_S_u7 : F16_LDI_u7;
 600: 
 601: //----------------------------------------------------------------------------
 602: // Indexed Jump or Execute.
 603: //----------------------------------------------------------------------------
 604: def JLI_S : F16_JLI_EI<0, "jli_s">;
 605: def EI_S : F16_JLI_EI<1, "ei_s">;
 606: 
 607: //----------------------------------------------------------------------------
 608: // Load/Add Register-Register.
 609: //----------------------------------------------------------------------------
 610: def LD_S_rrr : F16_LD_ADD_RR<0b00, "ld_s\t$a, [$b, $c]">;
 611: def LDB_S_rrr : F16_LD_ADD_RR<0b01, "ldb_s\t$a, [$b, $c]">;
 612: def LDH_S_rrr : F16_LD_ADD_RR<0b10, "ldh_s\t$a, [$b, $c]">;
```

- EN: This range defines declarative TableGen records such as LD_S_s11, ST_S_s11, LDI_S_u7, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 LD_S_s11, ST_S_s11, LDI_S_u7 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 613-630

```tablegen
 613: def ADD_S_rrr : F16_LD_ADD_RR<0b11, "add_s\t$a, $b, $c">;
 614: 
 615: //----------------------------------------------------------------------------
 616: // Load/Add GP-Relative.
 617: //----------------------------------------------------------------------------
 618: def GP_LD_S : F16_GP_LD_ADD<0b00, (ins immS<11>:$s),
 619:   "ld_s\t%r0, [%gp, $s]"> {
 620: 
 621:   bits<11> s;
 622:   let Inst{8-0} = s{10-2};
 623:   let s{1-0} = 0b00;
 624: }
 625: 
 626: def GP_LDB_S : F16_GP_LD_ADD<0b01, (ins immS<9>:$s),
 627:   "ldb_s\t%r0, [%gp, $s]"> {
 628: 
 629:   bits<9> s;
 630:   let Inst{8-0} = s{8-0};
```

- EN: This range defines declarative TableGen records such as ADD_S_rrr, GP_LD_S, GP_LDB_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADD_S_rrr, GP_LD_S, GP_LDB_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 631-648

```tablegen
 631: }
 632: 
 633: def GP_LDH_S : F16_GP_LD_ADD<0b10, (ins immS<10>:$s),
 634:   "ldh_s\t%r0, [%gp, $s]"> {
 635: 
 636:   bits<10> s;
 637:   let Inst{8-0} = s{9-1};
 638:   let s{0} = 0b0;
 639: }
 640: 
 641: def GP_ADD_S : F16_GP_LD_ADD<0b11, (ins immS<11>:$s),
 642:   "add_s\t%r0, %gp, $s"> {
 643: 
 644:   bits<11> s;
 645:   let Inst{8-0} = s{10-2};
 646:   let s{1-0} = 0b00;
 647: }
 648: 
```

- EN: This range defines declarative TableGen records such as GP_LDH_S, GP_ADD_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 GP_LDH_S, GP_ADD_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 649-666

```tablegen
 649: //----------------------------------------------------------------------------
 650: // Load PCL-Relative.
 651: //----------------------------------------------------------------------------
 652: def PCL_LD : InstARC<2, (outs GPR32:$b), (ins immU<10>:$u10),
 653:  "ld_s\t$b, [%pcl, $u10]", []> {
 654: 
 655:   bits<3> b;
 656:   bits<10> u10;
 657: 
 658:   let Inst{15-11} = 0b11010;
 659:   let Inst{10-8} = b;
 660:   let Inst{7-0} = u10{9-2};
 661:   let u10{1-0} = 0b00;
 662: }
 663: 
 664: let isBranch = 1 in {
 665:   //----------------------------------------------------------------------------
 666:   // Branch on Compare Register with Zero.
```

- EN: This range defines declarative TableGen records such as PCL_LD, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 PCL_LD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 667-684

```tablegen
 667:   //----------------------------------------------------------------------------
 668:   def BREQ_S : F16_BCC_REG<0b0, "breq_s">;
 669:   def BRNE_S : F16_BCC_REG<0b1, "brne_s">;
 670: 
 671:   //----------------------------------------------------------------------------
 672:   // Branch Conditionally.
 673:   //----------------------------------------------------------------------------
 674:   let isBarrier = 1 in
 675:   def B_S : F16_BCC_s10<0b00, "b_s">;
 676: 
 677:   def BEQ_S : F16_BCC_s10<0b01, "beq_s">;
 678:   def BNE_S : F16_BCC_s10<0b10, "bne_s">;
 679:   def BGT_S : F16_BCC_s7<0b000, "bgt_s">;
 680:   def BGE_S : F16_BCC_s7<0b001, "bge_s">;
 681:   def BLT_S : F16_BCC_s7<0b010, "blt_s">;
 682:   def BLE_S : F16_BCC_s7<0b011, "ble_s">;
 683:   def BHI_S : F16_BCC_s7<0b100, "bhi_s">;
 684:   def BHS_S : F16_BCC_s7<0b101, "bhs_s">;
```

- EN: This range defines declarative TableGen records such as BREQ_S, BRNE_S, B_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BREQ_S, BRNE_S, B_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 685-702

```tablegen
 685:   def BLO_S : F16_BCC_s7<0b110, "blo_s">;
 686:   def BLS_S : F16_BCC_s7<0b111, "bls_s">;
 687: } // let isBranch
 688: 
 689: def BL_S :
 690:   InstARC<2, (outs), (ins btargetS13:$s13), "bl_s\t$s13", []> {
 691: 
 692:   let Inst{15-11} = 0b11111;
 693: 
 694:   bits<13> s13;
 695:   let Inst{10-0} = s13{12-2};
 696:   let s13{1-0} = 0b00;
 697: 
 698:   let isCall = 1;
 699:   let isBarrier = 1;
 700: }
 701: 
 702: //----------------------------------------------------------------------------
```

- EN: This range defines declarative TableGen records such as BLO_S, BLS_S, BL_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BLO_S, BLS_S, BL_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 703-720

```tablegen
 703: // Add/Sub/Shift Register-Immediate.
 704: //----------------------------------------------------------------------------
 705: def ADD_S_ru3 : F16_ADD_IMM<0b00,"add_s">;
 706: def SUB_S_ru3 : F16_ADD_IMM<0b01,"sub_s">;
 707: def ASL_S_ru3 : F16_ADD_IMM<0b10,"asl_s">;
 708: def ASR_S_ru3 : F16_ADD_IMM<0b11,"asr_s">;
 709: 
 710: //----------------------------------------------------------------------------
 711: // Shift/Subtract/Bit Immediate.
 712: //----------------------------------------------------------------------------
 713: def ASL_S_ru5 : F16_SH_SUB_BIT_DST<0b000,"asl_s">;
 714: def LSR_S_ru5 : F16_SH_SUB_BIT_DST<0b001,"lsr_s">;
 715: def ASR_S_ru5 : F16_SH_SUB_BIT_DST<0b010,"asr_s">;
 716: def SUB_S_ru5 : F16_SH_SUB_BIT_DST<0b011,"sub_s">;
 717: def BSET_S_ru5 : F16_SH_SUB_BIT_DST<0b100,"bset_s">;
 718: def BCLR_S_ru5 : F16_SH_SUB_BIT_DST<0b101,"bclr_s">;
 719: def BMSK_S_ru5 : F16_SH_SUB_BIT_DST<0b110,"bmsk_s">;
 720: def BTST_S_ru5 : F16_SH_SUB_BIT<0b111, "btst_s\t$b, $u5">;
```

- EN: This range defines declarative TableGen records such as ADD_S_ru3, SUB_S_ru3, ASL_S_ru3, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ADD_S_ru3, SUB_S_ru3, ASL_S_ru3 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 721-738

```tablegen
 721: 
 722: //----------------------------------------------------------------------------
 723: // Dual Register Operations.
 724: //----------------------------------------------------------------------------
 725: def ADD_S_rlimm :
 726:   F16_OP_HREG_LIMM<0b000, (outs GPR32:$b_s3), (ins i32imm:$LImm),
 727:           !strconcat("add_s", "\t$b_s3, $b_s3, $LImm")>;
 728: 
 729: def ADD_S_rr :
 730:   F16_OP_HREG<0b000, (outs GPR32:$b_s3), (ins GPR32:$h),
 731:           !strconcat("add_s", "\t$b_s3, $b_s3, $h")>;
 732: 
 733: def ADD_S_rs3 :
 734:   F16_OP_HREG<0b001, (outs GPR32:$h), (ins immC<3>:$b_s3),
 735:           !strconcat("add_s", "\t$h, $h, $b_s3")>;
 736: 
 737: def ADD_S_limms3 :
 738:   F16_OP_HREG_LIMM<0b001, (outs), (ins immC<3>:$b_s3, i32imm:$LImm),
```

- EN: This range defines declarative TableGen records such as ADD_S_rlimm, ADD_S_rr, ADD_S_rs3, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ADD_S_rlimm, ADD_S_rr, ADD_S_rs3 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 739-756

```tablegen
 739:           !strconcat("add_s", "\t0, $LImm, $b_s3")>;
 740: 
 741: def MOV_S_NE_rlimm :
 742:   F16_OP_HREG_LIMM<0b111, (outs GPR32:$b_s3), (ins i32imm:$LImm),
 743:           !strconcat("mov_s.ne", "\t$b_s3, $LImm")>;
 744: 
 745: def MOV_S_NE_rr :
 746:   F16_OP_HREG<0b111,(outs GPR32:$b_s3), (ins GPR32:$h),
 747:           !strconcat("mov_s.ne", "\t$b_s3, $h")>;
 748: 
 749: def MOV_S_rs3 :
 750:   F16_OP_HREG<0b011, (outs GPR32:$h), (ins immC<3>:$b_s3),
 751:           !strconcat("mov_s", "\t$h, $b_s3")>;
 752: 
 753: def MOV_S_s3 :
 754:   F16_OP_HREG30<0b011, (outs), (ins immC<3>:$b_s3),
 755:           !strconcat("mov_s", "\t0, $b_s3")>;
 756: 
```

- EN: This range defines declarative TableGen records such as MOV_S_NE_rlimm, MOV_S_NE_rr, MOV_S_rs3, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 MOV_S_NE_rlimm, MOV_S_NE_rr, MOV_S_rs3 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 757-774

```tablegen
 757: def CMP_S_rlimm :
 758:   F16_OP_HREG_LIMM<0b100, (outs GPR32:$b_s3), (ins i32imm:$LImm),
 759:           !strconcat("cmp_s", "\t$b_s3, $LImm")>;
 760: 
 761: def CMP_S_rr :
 762:   F16_OP_HREG<0b100, (outs GPR32:$b_s3), (ins GPR32:$h),
 763:           !strconcat("cmp_s", "\t$b_s3, $h")>;
 764: 
 765: def CMP_S_rs3 :
 766:   F16_OP_HREG<0b101, (outs GPR32:$h), (ins immC<3>:$b_s3),
 767:           !strconcat("cmp_s", "\t$h, $b_s3")>;
 768: 
 769: def CMP_S_limms3 :
 770:   F16_OP_HREG_LIMM<0b101, (outs), (ins immC<3>:$b_s3, i32imm:$LImm),
 771:           !strconcat("cmp_s", "\t$LImm, $b_s3")>;
 772: 
 773: //----------------------------------------------------------------------------
 774: // Compact MOV/ADD/CMP Immediate instructions.
```

- EN: This range defines declarative TableGen records such as CMP_S_rlimm, CMP_S_rr, CMP_S_rs3, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CMP_S_rlimm, CMP_S_rr, CMP_S_rs3 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 775-792

```tablegen
 775: //----------------------------------------------------------------------------
 776: def MOV_S_u8 :
 777:   F16_OP_IMM<0b11011, (outs GPR32:$b), (ins immU<8>:$u8),
 778:           !strconcat("mov_s", "\t$b, $u8")> {
 779:   bits<8> u8;
 780:   let Inst{7-0} = u8;
 781: }
 782: 
 783: def ADD_S_u7 :
 784:   F16_OP_U7<0b0, !strconcat("add_s", "\t$b, $b, $u7")>;
 785: 
 786: def CMP_S_u7 :
 787:   F16_OP_U7<0b1, !strconcat("cmp_s", "\t$b, $u7")>;
 788: 
 789: //----------------------------------------------------------------------------
 790: // Compact Load/Store instructions with offset.
 791: //----------------------------------------------------------------------------
 792: def LD_S_OFF :
```

- EN: This range defines declarative TableGen records such as MOV_S_u8, ADD_S_u7, CMP_S_u7, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MOV_S_u8, ADD_S_u7, CMP_S_u7 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 793-810

```tablegen
 793:   F16_LD_ST_WORD_OFF<0x10, (outs GPR32:$c), (ins GPR32:$b, immU<7>:$off),
 794:   "ld_s">;
 795: 
 796: def LDB_S_OFF :
 797:   F16_LD_ST_BYTE_OFF<0x11, (outs GPR32:$c), (ins GPR32:$b, immU<5>:$off),
 798:   "ldb_s">;
 799: 
 800: class F16_LDH_OFF<bits<5> opc, string asmstr> :
 801:   F16_LD_ST_HALF_OFF<opc, (outs GPR32:$c), (ins GPR32:$b, immU<6>:$off),
 802:   asmstr>;
 803: 
 804: def LDH_S_OFF : F16_LDH_OFF<0x12, "ldh_s">;
 805: def LDH_S_X_OFF : F16_LDH_OFF<0x13, "ldh_s.x">;
 806: 
 807: def ST_S_OFF :
 808:   F16_LD_ST_WORD_OFF<0x14, (outs), (ins GPR32:$c, GPR32:$b, immU<7>:$off),
 809:   "st_s">;
 810: 
```

- EN: This range defines declarative TableGen records such as LDB_S_OFF, F16_LDH_OFF, LDH_S_OFF, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 LDB_S_OFF, F16_LDH_OFF, LDH_S_OFF 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 811-828

```tablegen
 811: def STB_S_OFF :
 812:   F16_LD_ST_BYTE_OFF<0x15, (outs), (ins GPR32:$c, GPR32:$b, immU<5>:$off),
 813:   "stb_s">;
 814: 
 815: def STH_S_OFF :
 816:   F16_LD_ST_HALF_OFF<0x16, (outs), (ins GPR32:$c, GPR32:$b, immU<6>:$off),
 817:   "sth_s">;
 818: 
 819: //----------------------------------------------------------------------------
 820: // General compact instructions.
 821: //----------------------------------------------------------------------------
 822: def GEN_SUB_S : F16_GEN_DOP<0x02, "sub_s">;
 823: def GEN_AND_S : F16_GEN_DOP<0x04, "and_s">;
 824: def GEN_OR_S : F16_GEN_DOP<0x05, "or_s">;
 825: def GEN_BIC_S : F16_GEN_DOP<0x06, "bic_s">;
 826: def GEN_XOR_S : F16_GEN_DOP<0x07, "xor_s">;
 827: def GEN_MPYW_S : F16_GEN_DOP<0x09, "mpyw_s">;
 828: def GEN_MPYUW_S : F16_GEN_DOP<0x0a, "mpyuw_s">;
```

- EN: This range defines declarative TableGen records such as STB_S_OFF, STH_S_OFF, GEN_SUB_S, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 STB_S_OFF, STH_S_OFF, GEN_SUB_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 829-846

```tablegen
 829: def GEN_TST_S : F16_GEN_DOP_NODST<0x0b, "tst_s">;
 830: def GEN_MPY_S : F16_GEN_DOP<0x0c, "mpy_s">;
 831: def GEN_SEXB_S : F16_GEN_DOP_SINGLESRC<0x0d, "sexb_s">;
 832: def GEN_SEXH_S : F16_GEN_DOP_SINGLESRC<0x0e, "sexh_s">;
 833: def GEN_EXTB_S : F16_GEN_DOP_SINGLESRC<0x0f, "extb_s">;
 834: def GEN_EXTH_S : F16_GEN_DOP_SINGLESRC<0x10, "exth_s">;
 835: def GEN_ABS_S : F16_GEN_DOP_SINGLESRC<0x11, "abs_s">;
 836: def GEN_NOT_S : F16_GEN_DOP_SINGLESRC<0x12, "not_s">;
 837: def GEN_NEG_S : F16_GEN_DOP_SINGLESRC<0x13, "neg_s">;
 838: def GEN_ADD1_S : F16_GEN_DOP<0x14, "add1_s">;
 839: def GEN_ADD2_S : F16_GEN_DOP<0x15, "add2_s">;
 840: def GEN_ADD3_S : F16_GEN_DOP<0x16, "add3_s">;
 841: def GEN_ASL_S : F16_GEN_DOP<0x18, "asl_s">;
 842: def GEN_LSR_S : F16_GEN_DOP<0x19, "lsr_s">;
 843: def GEN_ASR_S : F16_GEN_DOP<0x1a, "asr_s">;
 844: def GEN_AS1L_S : F16_GEN_DOP_SINGLESRC<0x1b, "asl_s">;
 845: def GEN_AS1R_S : F16_GEN_DOP_SINGLESRC<0x1c, "asr_s">;
 846: def GEN_LS1R_S : F16_GEN_DOP_SINGLESRC<0x1d, "lsr_s">;
```

- EN: This range defines declarative TableGen records such as GEN_TST_S, GEN_MPY_S, GEN_SEXB_S, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 GEN_TST_S, GEN_MPY_S, GEN_SEXB_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 847-864

```tablegen
 847: def GEN_TRAP_S : F16_GEN_DOP_BASE<0x1e, (outs), (ins immU6:$u6),
 848:   "trap_s\t$u6"> {
 849: 
 850:   bits<6> u6;
 851:   let b = u6{5-3};
 852:   let c = u6{2-0};
 853: }
 854: 
 855: def GEN_BRK_S : F16_GEN_DOP_BASE<0x1f, (outs), (ins),
 856:   "brk_s"> {
 857: 
 858:   let b = 0b111;
 859:   let c = 0b111;
 860: }
 861: 
 862: let isBarrier = 1 in {
 863:   let isBranch = 1 in {
 864:     def GEN_J_S : F16_GEN_SOP<0x0, "j_s\t[$b]">;
```

- EN: This range defines declarative TableGen records such as GEN_TRAP_S, GEN_BRK_S, GEN_J_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 GEN_TRAP_S, GEN_BRK_S, GEN_J_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 865-882

```tablegen
 865:     def GEN_J_S_D : F16_GEN_SOP<0x1, "j_s.d\t[$b]">;
 866:   } // let isBranch
 867: 
 868:   let isCall = 1 in {
 869:     def GEN_JL_S : F16_GEN_SOP<0x2, "jl_s\t[$b]">;
 870:     def GEN_JL_S_D : F16_GEN_SOP<0x3, "jl_s.d\t[$b]">;
 871:   } // let isCall
 872: } // let isBarrier
 873: 
 874: def GEN_SUB_S_NE : F16_GEN_SOP<0x6, "sub_s.ne\t$b, $b, $b">;
 875: 
 876: def GEN_NOP_S : F16_GEN_ZOP<0x0, "nop_s">;
 877: def GEN_UNIMP_S : F16_GEN_ZOP<0x1, "unimp_s">;
 878: def GEN_SWI_S : F16_GEN_ZOP<0x2, "swi_s">;
 879: 
 880: let isReturn = 1, isTerminator = 1 in {
 881:   def GEN_JEQ_S : F16_GEN_ZOP<0x4, "jeq_s\t[%blink]">;
 882:   def GEN_JNE_S : F16_GEN_ZOP<0x5, "jne_s\t[%blink]">;
```

- EN: This range defines declarative TableGen records such as GEN_J_S_D, GEN_JL_S, GEN_JL_S_D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 GEN_J_S_D, GEN_JL_S, GEN_JL_S_D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 883-900

```tablegen
 883:   let isBarrier = 1 in {
 884:     //def GEN_J_S_BLINK : F16_GEN_ZOP<0x6, "j_s\t[%blink]">;
 885:     def GEN_J_S_D_BLINK : F16_GEN_ZOP<0x7, "j_s.d\t[%blink]">;
 886:   } // let isBarrier
 887: } // let isReturn, isTerminator
 888: 
 889: //----------------------------------------------------------------------------
 890: // Load/Store instructions.
 891: //----------------------------------------------------------------------------
 892: 
 893: // Filter  class for load/store mappings
 894: class ArcLdStRel;
 895: 
 896: // Load instruction variants:
 897: // Control bits: x, aa, di, zz
 898: // x - sign extend.
 899: // aa - incrementing mode. (N/A for LIMM).
 900: // di - uncached.
```

- EN: This range defines declarative TableGen records such as GEN_J_S_D_BLINK, ArcLdStRel, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 GEN_J_S_D_BLINK, ArcLdStRel 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 901-918

```tablegen
 901: // zz - data size.
 902: multiclass ArcLdInst<DataSizeMode zz, ExtMode x, CacheMode di, string asmop> {
 903:   let mayLoad = 1, ZZ = zz, X = x, DI = di in {
 904:     def _rs9: F32_LD_ADDR<x.Value, NoAM.Value, di.Value, zz.Value,
 905: 			   (outs GPR32:$A), (ins MEMrs9:$addr),
 906: 			   !strconcat(asmop, "\t$A, [$addr]"), []>, ArcLdStRel;
 907: 
 908:     def _limm: F32_LD_LIMM<x.Value, di.Value, zz.Value,
 909: 			   (outs GPR32:$A), (ins MEMii:$addr),
 910: 			   !strconcat(asmop, "\t$A, [$addr]"), []>, ArcLdStRel;
 911: 
 912:     def _rlimm: F32_LD_RLIMM<x.Value, NoAM.Value, di.Value, zz.Value,
 913: 			     (outs GPR32:$A), (ins MEMrlimm:$addr),
 914: 			     !strconcat(asmop, "\t$A, [$addr]"), []>, ArcLdStRel;
 915: 
 916:     foreach aa = [PreIncAM, PostIncAM] in {
 917:       def aa.InstSuffix#_rs9: F32_LD_RS9<x.Value, aa.Value, di.Value, zz.Value,
 918: 					  (outs GPR32:$A, GPR32:$addrout),
```

- EN: This range defines declarative TableGen records such as ArcLdInst, _rs9, _limm, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ArcLdInst, _rs9, _limm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 919-936

```tablegen
 919: 					  (ins GPR32:$B, immS<9>:$S9),
 920: 					  asmop#aa.AsmSuffix#"\t$A, [$B,$S9]", []>, ArcLdStRel
 921: 			       { let Constraints = "$addrout = $B"; let AA = aa; }
 922:     }
 923:   }
 924: }
 925: 
 926: foreach di = [NoCC, UncachedCC] in {
 927:   defm LD#di.InstSuffix : ArcLdInst<WordSM, NoEM, di, "ld"#di.AsmSuffix>;
 928:   foreach zz = [ByteSM, HalfSM] in {
 929:     foreach x = [NoEM, SignedEM] in {
 930:       defm LD#zz.InstSuffix#x.InstSuffix#di.InstSuffix : ArcLdInst<zz, x, di, "ld"#zz.AsmSuffix#x.AsmSuffix#di.AsmSuffix>;
 931:     }
 932:   }
 933: }
 934: 
 935: // Load instruction patterns.
 936: // 32-bit loads.
```

- EN: This range defines declarative TableGen records such as LD, LD, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 LD, LD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 937-954

```tablegen
 937: def : Pat<(load AddrModeS9:$addr), (LD_rs9 AddrModeS9:$addr)>;
 938: def : Pat<(load AddrModeImm:$addr), (LD_limm AddrModeImm:$addr)>;
 939: def : Pat<(load AddrModeFar:$addr), (LD_rs9 AddrModeFar:$addr)>;
 940: 
 941: // 16-bit loads
 942: def : Pat<(zextloadi16 AddrModeS9:$addr), (LDH_rs9 AddrModeS9:$addr)>;
 943: def : Pat<(extloadi16 AddrModeS9:$addr), (LDH_rs9 AddrModeS9:$addr)>;
 944: def : Pat<(zextloadi16 AddrModeImm:$addr), (LDH_limm AddrModeImm:$addr)>;
 945: def : Pat<(extloadi16 AddrModeImm:$addr), (LDH_limm AddrModeImm:$addr)>;
 946: def : Pat<(zextloadi16 AddrModeFar:$addr), (LDH_rlimm AddrModeFar:$addr)>;
 947: def : Pat<(extloadi16 AddrModeFar:$addr), (LDH_rlimm AddrModeFar:$addr)>;
 948: def : Pat<(sextloadi16 AddrModeImm:$addr),(LDH_X_limm AddrModeImm:$addr)>;
 949: def : Pat<(sextloadi16 AddrModeFar:$addr),(LDH_X_rlimm AddrModeFar:$addr)>;
 950: def : Pat<(sextloadi16 AddrModeS9:$addr),(LDH_X_rs9 AddrModeS9:$addr)>;
 951: 
 952: // 8-bit loads.
 953: def : Pat<(zextloadi8 AddrModeS9:$addr), (LDB_rs9 AddrModeS9:$addr)>;
 954: def : Pat<(extloadi8 AddrModeS9:$addr), (LDB_rs9 AddrModeS9:$addr)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 955-972

```tablegen
 955: def : Pat<(zextloadi8 AddrModeImm:$addr), (LDB_limm AddrModeImm:$addr)>;
 956: def : Pat<(extloadi8 AddrModeImm:$addr), (LDB_limm AddrModeImm:$addr)>;
 957: def : Pat<(zextloadi8 AddrModeFar:$addr), (LDB_rlimm AddrModeFar:$addr)>;
 958: def : Pat<(extloadi8 AddrModeFar:$addr), (LDB_rlimm AddrModeFar:$addr)>;
 959: def : Pat<(zextloadi1 AddrModeS9:$addr), (LDB_rs9 AddrModeS9:$addr)>;
 960: def : Pat<(extloadi1 AddrModeS9:$addr), (LDB_rs9 AddrModeS9:$addr)>;
 961: def : Pat<(zextloadi1 AddrModeImm:$addr), (LDB_limm AddrModeImm:$addr)>;
 962: def : Pat<(extloadi1 AddrModeImm:$addr), (LDB_limm AddrModeImm:$addr)>;
 963: def : Pat<(zextloadi1 AddrModeFar:$addr), (LDB_rlimm AddrModeFar:$addr)>;
 964: def : Pat<(extloadi1 AddrModeFar:$addr), (LDB_rlimm AddrModeFar:$addr)>;
 965: def : Pat<(sextloadi8 AddrModeImm:$addr),(LDB_X_limm AddrModeImm:$addr)>;
 966: def : Pat<(sextloadi8 AddrModeFar:$addr),(LDB_X_rlimm AddrModeFar:$addr)>;
 967: def : Pat<(sextloadi8 AddrModeS9:$addr),(LDB_X_rs9 AddrModeS9:$addr)>;
 968: 
 969: 
 970: // Store instruction variants:
 971: // Control bits: aa, di, zz
 972: // aa - incrementing mode. (N/A for LIMM).
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 973-990

```tablegen
 973: // di - uncached.
 974: // zz - data size.
 975: multiclass ArcStInst<DataSizeMode zz, CacheMode di, string asmop> {
 976:   let mayStore = 1, ZZ = zz, DI = di in {
 977:     def _rs9: F32_ST_ADDR<NoAM.Value, di.Value, zz.Value,
 978: 			   (outs), (ins GPR32:$C, MEMrs9:$addr),
 979: 			   !strconcat(asmop, "\t$C, [$addr]"), []>, ArcLdStRel;
 980: 
 981:     def _limm: F32_ST_LIMM<di.Value, zz.Value,
 982: 			   (outs), (ins GPR32:$C, MEMii:$addr),
 983: 			   !strconcat(asmop, "\t$C, [$addr]"), []>, ArcLdStRel;
 984: 
 985: 
 986:     foreach aa = [PreIncAM, PostIncAM] in {
 987:       def aa.InstSuffix#_rs9: F32_ST_RS9<aa.Value, di.Value, zz.Value,
 988: 					  (outs GPR32:$addrout),
 989: 					  (ins GPR32:$C, GPR32:$B, immS<9>:$S9),
 990: 					  asmop#aa.AsmSuffix#"\t$C, [$B,$S9]", []>, ArcLdStRel
```

- EN: This range defines declarative TableGen records such as ArcStInst, _rs9, _limm, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ArcStInst, _rs9, _limm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 991-1008

```tablegen
 991: 			       { let Constraints = "$addrout = $B"; let AA = aa; }
 992:     }
 993:   }
 994: }
 995: 
 996: foreach di = [NoCC, UncachedCC] in {
 997:   foreach zz = [ByteSM, HalfSM, WordSM] in {
 998:       defm ST#zz.InstSuffix#di.InstSuffix : ArcStInst<zz, di, "st"#zz.AsmSuffix#di.AsmSuffix>;
 999:   }
1000: }
1001: 
1002: // Store instruction patterns.
1003: // 32-bit stores
1004: def : Pat<(store i32:$C, AddrModeS9:$addr),
1005:           (ST_rs9 i32:$C, AddrModeS9:$addr)>;
1006: def : Pat<(store i32:$C, AddrModeImm:$addr),
1007:           (ST_limm i32:$C, AddrModeImm:$addr)>;
1008: 
```

- EN: This range defines declarative TableGen records such as ST, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 ST 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1009-1026

```tablegen
1009: // 16-bit stores
1010: def : Pat<(truncstorei16 i32:$C, AddrModeS9:$addr),
1011:           (STH_rs9 i32:$C, AddrModeS9:$addr)>;
1012: def : Pat<(truncstorei16 i32:$C, AddrModeImm:$addr),
1013:           (STH_limm i32:$C, AddrModeImm:$addr)>;
1014: 
1015: // 8-bit stores
1016: def : Pat<(truncstorei8 i32:$C, AddrModeS9:$addr),
1017:           (STB_rs9 i32:$C, AddrModeS9:$addr)>;
1018: def : Pat<(truncstorei8 i32:$C, AddrModeImm:$addr),
1019:           (STB_limm i32:$C, AddrModeImm:$addr)>;
1020: 
1021: def getPostIncOpcode : InstrMapping {
1022:   let FilterClass = "ArcLdStRel";
1023:   let RowFields = [ "BaseOpcode", "ZZ", "DI", "X"];
1024:   let ColFields = [ "AA" ];
1025:   let KeyCol = [ "NoAM" ];
1026:   let ValueCols = [["PostIncAM"]];
```

- EN: This range defines declarative TableGen records such as getPostIncOpcode, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 getPostIncOpcode 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1027-1027

```tablegen
1027: }
```

- EN: This range continues the implementation of the backend component described by ARCInstrInfo.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- TableGen instruction records / TableGen 指令记录
- Encoding and patterns / 编码与匹配模式
- CPU feature modelling / CPU 特性建模
- Pseudo-instruction handling / 伪指令处理
- Pattern matching / 模式匹配

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCInstrFormats.td`
- LLVM subsystems / LLVM 子系统: TableGen
- Local companions / 本地配套文件: `ARCInstrInfo.h`, `ARCInstrInfo.cpp`
