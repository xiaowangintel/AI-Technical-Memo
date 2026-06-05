# LoongArchInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchInstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 LoongArch 后端元数据；具体而言，它定义或实现目标指令信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //== LoongArchInstrInfo.td - Target Description for LoongArch -*- tablegen -*-//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the LoongArch instructions in TableGen format.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: //===----------------------------------------------------------------------===//
  14: // LoongArch specific DAG Nodes.
  15: //===----------------------------------------------------------------------===//
  16: 
  17: // Target-independent type requirements, but with target-specific formats.
  18: def SDT_CallSeqStart : SDCallSeqStart<[SDTCisVT<0, i32>,
  19:                                        SDTCisVT<1, i32>]>;
  20: def SDT_CallSeqEnd : SDCallSeqEnd<[SDTCisVT<0, i32>,
  21:                                    SDTCisVT<1, i32>]>;
  22: 
  23: // Target-dependent type requirements.
  24: def SDT_LoongArchCall : SDTypeProfile<0, -1, [SDTCisVT<0, GRLenVT>]>;
  25: def SDT_LoongArchIntBinOpW : SDTypeProfile<1, 2, [
  26:   SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>, SDTCisVT<0, i64>
  27: ]>;
  28: 
  29: def SDT_LoongArchSelectCC : SDTypeProfile<1, 5, [SDTCisSameAs<1, 2>,
  30:                                                  SDTCisVT<3, OtherVT>,
  31:                                                  SDTCisSameAs<0, 4>,
  32:                                                  SDTCisSameAs<4, 5>]>;
  33: 
  34: def SDT_LoongArchBrCC : SDTypeProfile<0, 4, [SDTCisSameAs<0, 1>,
  35:                                              SDTCisVT<2, OtherVT>,
  36:                                              SDTCisVT<3, OtherVT>]>;
  37: 
  38: def SDT_LoongArchBStrIns: SDTypeProfile<1, 4, [
  39:   SDTCisInt<0>, SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>, SDTCisInt<3>,
  40:   SDTCisSameAs<3, 4>
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `SDT_CallSeqStart`, `SDT_CallSeqEnd`, `SDT_LoongArchCall`, `SDT_LoongArchIntBinOpW`, `SDT_LoongArchSelectCC`, `SDT_LoongArchBrCC`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `SDT_CallSeqStart`, `SDT_CallSeqEnd`, `SDT_LoongArchCall`, `SDT_LoongArchIntBinOpW`, `SDT_LoongArchSelectCC`, `SDT_LoongArchBrCC` 等 TableGen 记录。

### Lines 41-80 / 第 41-80 行
```tablegen
  41: ]>;
  42: 
  43: def SDT_LoongArchBStrPick: SDTypeProfile<1, 3, [
  44:   SDTCisInt<0>, SDTCisSameAs<0, 1>, SDTCisInt<2>, SDTCisSameAs<2, 3>
  45: ]>;
  46: 
  47: // "VI" means no output and an integer input.
  48: def SDT_LoongArchVI : SDTypeProfile<0, 1, [SDTCisVT<0, GRLenVT>]>;
  49: 
  50: def SDT_LoongArchCsrrd : SDTypeProfile<1, 1, [SDTCisInt<0>,
  51:                                               SDTCisVT<1, GRLenVT>]>;
  52: def SDT_LoongArchCsrwr : SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisSameAs<0, 1>,
  53:                                               SDTCisVT<2, GRLenVT>]>;
  54: def SDT_LoongArchCsrxchg : SDTypeProfile<1, 3, [SDTCisInt<0>,
  55:                                                 SDTCisSameAs<0, 1>,
  56:                                                 SDTCisSameAs<0, 2>,
  57:                                                 SDTCisVT<3, GRLenVT>]>;
  58: def SDT_LoongArchIocsrwr : SDTypeProfile<0, 2, [SDTCisInt<0>,
  59:                                                 SDTCisSameAs<0, 1>]>;
  60: def SDT_LoongArchMovgr2fcsr : SDTypeProfile<0, 2, [SDTCisVT<0, GRLenVT>,
  61:                                                    SDTCisSameAs<0, 1>]>;
  62: def SDT_LoongArchMovfcsr2gr : SDTypeProfile<1, 1, [SDTCisVT<0, GRLenVT>,
  63:                                                    SDTCisSameAs<0, 1>]>;
  64: 
  65: def SDT_LoongArchBuildPairF64 : SDTypeProfile<1, 2, [SDTCisVT<0, f64>,
  66:                                                      SDTCisVT<1, i32>,
  67:                                                      SDTCisSameAs<1, 2>]>;
  68: def SDT_LoongArchSplitPairF64 : SDTypeProfile<2, 1, [SDTCisVT<0, i32>,
  69:                                                      SDTCisVT<1, i32>,
  70:                                                      SDTCisVT<2, f64>]>;
  71: 
  72: // TODO: Add LoongArch specific DAG Nodes
  73: // Target-independent nodes, but with target-specific formats.
  74: def callseq_start : SDNode<"ISD::CALLSEQ_START", SDT_CallSeqStart,
  75:                            [SDNPHasChain, SDNPOutGlue]>;
  76: def callseq_end : SDNode<"ISD::CALLSEQ_END", SDT_CallSeqEnd,
  77:                          [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
  78: 
  79: // Target-dependent nodes.
  80: def loongarch_call : SDNode<"LoongArchISD::CALL", SDT_LoongArchCall,
```
- **EN**: This block declares or refines TableGen records such as `SDT_LoongArchBStrPick:`, `SDT_LoongArchVI`, `SDT_LoongArchCsrrd`, `SDT_LoongArchCsrwr`, `SDT_LoongArchCsrxchg`, `SDT_LoongArchIocsrwr`.
- **CN**: 该代码块声明或细化了 `SDT_LoongArchBStrPick:`, `SDT_LoongArchVI`, `SDT_LoongArchCsrrd`, `SDT_LoongArchCsrwr`, `SDT_LoongArchCsrxchg`, `SDT_LoongArchIocsrwr` 等 TableGen 记录。

### Lines 81-120 / 第 81-120 行
```tablegen
  81:                             [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
  82:                              SDNPVariadic]>;
  83: def loongarch_ret : SDNode<"LoongArchISD::RET", SDTNone,
  84:                            [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
  85: def loongarch_tail : SDNode<"LoongArchISD::TAIL", SDT_LoongArchCall,
  86:                             [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
  87:                              SDNPVariadic]>;
  88: def loongarch_call_medium : SDNode<"LoongArchISD::CALL_MEDIUM", SDT_LoongArchCall,
  89:                                    [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
  90:                                     SDNPVariadic]>;
  91: def loongarch_tail_medium : SDNode<"LoongArchISD::TAIL_MEDIUM", SDT_LoongArchCall,
  92:                                    [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
  93:                                     SDNPVariadic]>;
  94: def loongarch_call_large : SDNode<"LoongArchISD::CALL_LARGE", SDT_LoongArchCall,
  95:                                   [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
  96:                                    SDNPVariadic]>;
  97: def loongarch_tail_large : SDNode<"LoongArchISD::TAIL_LARGE", SDT_LoongArchCall,
  98:                                   [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
  99:                                    SDNPVariadic]>;
 100: def loongarch_selectcc : SDNode<"LoongArchISD::SELECT_CC", SDT_LoongArchSelectCC>;
 101: def loongarch_brcc : SDNode<"LoongArchISD::BR_CC", SDT_LoongArchBrCC,
 102:                             [SDNPHasChain]>;
 103: 
 104: // 32-bit shifts, directly matching the semantics of the named LoongArch
 105: // instructions.
 106: def loongarch_sll_w : SDNode<"LoongArchISD::SLL_W", SDT_LoongArchIntBinOpW>;
 107: def loongarch_sra_w : SDNode<"LoongArchISD::SRA_W", SDT_LoongArchIntBinOpW>;
 108: def loongarch_srl_w : SDNode<"LoongArchISD::SRL_W", SDT_LoongArchIntBinOpW>;
 109: 
 110: def loongarch_rotr_w : SDNode<"LoongArchISD::ROTR_W", SDT_LoongArchIntBinOpW>;
 111: 
 112: // unsigned 32-bit integer division
 113: def loongarch_div_w : SDNode<"LoongArchISD::DIV_W", SDT_LoongArchIntBinOpW>;
 114: def loongarch_div_wu : SDNode<"LoongArchISD::DIV_WU", SDT_LoongArchIntBinOpW>;
 115: def loongarch_mod_w : SDNode<"LoongArchISD::MOD_W", SDT_LoongArchIntBinOpW>;
 116: def loongarch_mod_wu : SDNode<"LoongArchISD::MOD_WU", SDT_LoongArchIntBinOpW>;
 117: 
 118: // CRC check operations
 119: def loongarch_crc_w_b_w
 120:     : SDNode<"LoongArchISD::CRC_W_B_W", SDT_LoongArchIntBinOpW, [SDNPHasChain]>;
```
- **EN**: This block declares or refines TableGen records such as `loongarch_ret`, `loongarch_tail`, `loongarch_call_medium`, `loongarch_tail_medium`, `loongarch_call_large`, `loongarch_tail_large`.
- **CN**: 该代码块声明或细化了 `loongarch_ret`, `loongarch_tail`, `loongarch_call_medium`, `loongarch_tail_medium`, `loongarch_call_large`, `loongarch_tail_large` 等 TableGen 记录。

### Lines 121-160 / 第 121-160 行
```tablegen
 121: def loongarch_crc_w_h_w
 122:     : SDNode<"LoongArchISD::CRC_W_H_W", SDT_LoongArchIntBinOpW, [SDNPHasChain]>;
 123: def loongarch_crc_w_w_w
 124:     : SDNode<"LoongArchISD::CRC_W_W_W", SDT_LoongArchIntBinOpW, [SDNPHasChain]>;
 125: def loongarch_crc_w_d_w
 126:     : SDNode<"LoongArchISD::CRC_W_D_W", SDT_LoongArchIntBinOpW, [SDNPHasChain]>;
 127: def loongarch_crcc_w_b_w : SDNode<"LoongArchISD::CRCC_W_B_W",
 128:                                   SDT_LoongArchIntBinOpW, [SDNPHasChain]>;
 129: def loongarch_crcc_w_h_w : SDNode<"LoongArchISD::CRCC_W_H_W",
 130:                                   SDT_LoongArchIntBinOpW, [SDNPHasChain]>;
 131: def loongarch_crcc_w_w_w : SDNode<"LoongArchISD::CRCC_W_W_W",
 132:                                   SDT_LoongArchIntBinOpW, [SDNPHasChain]>;
 133: def loongarch_crcc_w_d_w : SDNode<"LoongArchISD::CRCC_W_D_W",
 134:                                   SDT_LoongArchIntBinOpW, [SDNPHasChain]>;
 135: 
 136: def loongarch_bstrins
 137:     : SDNode<"LoongArchISD::BSTRINS", SDT_LoongArchBStrIns>;
 138: def loongarch_bstrpick
 139:     : SDNode<"LoongArchISD::BSTRPICK", SDT_LoongArchBStrPick>;
 140: 
 141: // Byte-swapping and bit-reversal
 142: def loongarch_revb_2h : SDNode<"LoongArchISD::REVB_2H", SDTUnaryOp>;
 143: def loongarch_revb_2w : SDNode<"LoongArchISD::REVB_2W", SDTUnaryOp>;
 144: def loongarch_bitrev_4b : SDNode<"LoongArchISD::BITREV_4B", SDTUnaryOp>;
 145: def loongarch_bitrev_8b : SDNode<"LoongArchISD::BITREV_8B", SDTUnaryOp>;
 146: def loongarch_bitrev_w : SDNode<"LoongArchISD::BITREV_W", SDTUnaryOp>;
 147: 
 148: // Bit counting operations
 149: def loongarch_clzw : SDNode<"LoongArchISD::CLZ_W", SDTIntBitCountUnaryOp>;
 150: def loongarch_ctzw : SDNode<"LoongArchISD::CTZ_W", SDTIntBitCountUnaryOp>;
 151: 
 152: def loongarch_dbar : SDNode<"LoongArchISD::DBAR", SDT_LoongArchVI,
 153:                              [SDNPHasChain, SDNPSideEffect]>;
 154: def loongarch_ibar : SDNode<"LoongArchISD::IBAR", SDT_LoongArchVI,
 155:                              [SDNPHasChain, SDNPSideEffect]>;
 156: def loongarch_break : SDNode<"LoongArchISD::BREAK", SDT_LoongArchVI,
 157:                               [SDNPHasChain, SDNPSideEffect]>;
 158: 
 159: // FPR<->GPR transfer operations
 160: def loongarch_movfcsr2gr : SDNode<"LoongArchISD::MOVFCSR2GR",
```
- **EN**: This block declares or refines TableGen records such as `loongarch_crc_w_h_w`, `loongarch_crc_w_w_w`, `loongarch_crc_w_d_w`, `loongarch_crcc_w_b_w`, `loongarch_crcc_w_h_w`, `loongarch_crcc_w_w_w`.
- **CN**: 该代码块声明或细化了 `loongarch_crc_w_h_w`, `loongarch_crc_w_w_w`, `loongarch_crc_w_d_w`, `loongarch_crcc_w_b_w`, `loongarch_crcc_w_h_w`, `loongarch_crcc_w_w_w` 等 TableGen 记录。

### Lines 161-200 / 第 161-200 行
```tablegen
 161:                                   SDT_LoongArchMovfcsr2gr, [SDNPHasChain]>;
 162: def loongarch_movgr2fcsr : SDNode<"LoongArchISD::MOVGR2FCSR",
 163:                                   SDT_LoongArchMovgr2fcsr,
 164:                                   [SDNPHasChain, SDNPSideEffect]>;
 165: 
 166: def loongarch_syscall : SDNode<"LoongArchISD::SYSCALL", SDT_LoongArchVI,
 167:                                 [SDNPHasChain, SDNPSideEffect]>;
 168: def loongarch_csrrd : SDNode<"LoongArchISD::CSRRD", SDT_LoongArchCsrrd,
 169:                               [SDNPHasChain, SDNPSideEffect]>;
 170: 
 171: // Write new value to CSR and return old value.
 172: // Operand 0: A chain pointer.
 173: // Operand 1: The new value to write.
 174: // Operand 2: The address of the required CSR.
 175: // Result 0: The old value of the CSR.
 176: // Result 1: The new chain pointer.
 177: def loongarch_csrwr : SDNode<"LoongArchISD::CSRWR", SDT_LoongArchCsrwr,
 178:                               [SDNPHasChain, SDNPSideEffect]>;
 179: 
 180: // Similar to CSRWR but with a write mask.
 181: // Operand 0: A chain pointer.
 182: // Operand 1: The new value to write.
 183: // Operand 2: The write mask.
 184: // Operand 3: The address of the required CSR.
 185: // Result 0: The old value of the CSR.
 186: // Result 1: The new chain pointer.
 187: def loongarch_csrxchg : SDNode<"LoongArchISD::CSRXCHG",
 188:                                 SDT_LoongArchCsrxchg,
 189:                                 [SDNPHasChain, SDNPSideEffect]>;
 190: 
 191: // IOCSR access operations
 192: def loongarch_iocsrrd_b : SDNode<"LoongArchISD::IOCSRRD_B", SDTUnaryOp,
 193:                                   [SDNPHasChain, SDNPSideEffect]>;
 194: def loongarch_iocsrrd_h : SDNode<"LoongArchISD::IOCSRRD_H", SDTUnaryOp,
 195:                                   [SDNPHasChain, SDNPSideEffect]>;
 196: def loongarch_iocsrrd_w : SDNode<"LoongArchISD::IOCSRRD_W", SDTUnaryOp,
 197:                                   [SDNPHasChain, SDNPSideEffect]>;
 198: def loongarch_iocsrrd_d : SDNode<"LoongArchISD::IOCSRRD_D", SDTUnaryOp,
 199:                                   [SDNPHasChain, SDNPSideEffect]>;
 200: def loongarch_iocsrwr_b : SDNode<"LoongArchISD::IOCSRWR_B",
```
- **EN**: This block declares or refines TableGen records such as `loongarch_movgr2fcsr`, `loongarch_syscall`, `loongarch_csrrd`, `loongarch_csrwr`, `loongarch_csrxchg`, `loongarch_iocsrrd_b`.
- **CN**: 该代码块声明或细化了 `loongarch_movgr2fcsr`, `loongarch_syscall`, `loongarch_csrrd`, `loongarch_csrwr`, `loongarch_csrxchg`, `loongarch_iocsrrd_b` 等 TableGen 记录。

### Lines 201-240 / 第 201-240 行
```tablegen
 201:                                   SDT_LoongArchIocsrwr,
 202:                                   [SDNPHasChain, SDNPSideEffect]>;
 203: def loongarch_iocsrwr_h : SDNode<"LoongArchISD::IOCSRWR_H",
 204:                                   SDT_LoongArchIocsrwr,
 205:                                   [SDNPHasChain, SDNPSideEffect]>;
 206: def loongarch_iocsrwr_w : SDNode<"LoongArchISD::IOCSRWR_W",
 207:                                   SDT_LoongArchIocsrwr,
 208:                                   [SDNPHasChain, SDNPSideEffect]>;
 209: def loongarch_iocsrwr_d : SDNode<"LoongArchISD::IOCSRWR_D",
 210:                                   SDT_LoongArchIocsrwr,
 211:                                   [SDNPHasChain, SDNPSideEffect]>;
 212: 
 213: // Read CPU configuration information operation
 214: def loongarch_cpucfg : SDNode<"LoongArchISD::CPUCFG", SDTUnaryOp,
 215:                                [SDNPHasChain]>;
 216: 
 217: // Build and split F64 pair
 218: def loongarch_build_pair_f64 : SDNode<"LoongArchISD::BUILD_PAIR_F64",
 219:                                       SDT_LoongArchBuildPairF64>;
 220: def loongarch_split_pair_f64 : SDNode<"LoongArchISD::SPLIT_PAIR_F64",
 221:                                       SDT_LoongArchSplitPairF64>;
 222: 
 223: def to_fclass_mask: SDNodeXForm<timm, [{
 224:   uint64_t Check = N->getZExtValue();
 225:   unsigned Mask = 0;
 226:   if (Check & fcSNan)
 227:     Mask |= LoongArch::FClassMaskSignalingNaN;
 228:   if (Check & fcQNan)
 229:     Mask |= LoongArch::FClassMaskQuietNaN;
 230:   if (Check & fcPosInf)
 231:     Mask |= LoongArch::FClassMaskPositiveInfinity;
 232:   if (Check & fcNegInf)
 233:     Mask |= LoongArch::FClassMaskNegativeInfinity;
 234:   if (Check & fcPosNormal)
 235:     Mask |= LoongArch::FClassMaskPositiveNormal;
 236:   if (Check & fcNegNormal)
 237:     Mask |= LoongArch::FClassMaskNegativeNormal;
 238:   if (Check & fcPosSubnormal)
 239:     Mask |= LoongArch::FClassMaskPositiveSubnormal;
 240:   if (Check & fcNegSubnormal)
```
- **EN**: This block declares or refines TableGen records such as `loongarch_iocsrwr_h`, `loongarch_iocsrwr_w`, `loongarch_iocsrwr_d`, `loongarch_cpucfg`, `loongarch_build_pair_f64`, `loongarch_split_pair_f64`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `loongarch_iocsrwr_h`, `loongarch_iocsrwr_w`, `loongarch_iocsrwr_d`, `loongarch_cpucfg`, `loongarch_build_pair_f64`, `loongarch_split_pair_f64` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 241-280 / 第 241-280 行
```tablegen
 241:     Mask |= LoongArch::FClassMaskNegativeSubnormal;
 242:   if (Check & fcPosZero)
 243:     Mask |= LoongArch::FClassMaskPositiveZero;
 244:   if (Check & fcNegZero)
 245:     Mask |= LoongArch::FClassMaskNegativeZero;
 246:   return CurDAG->getTargetConstant(Mask, SDLoc(N), Subtarget->getGRLenVT());
 247: }]>;
 248: 
 249: //===----------------------------------------------------------------------===//
 250: // Operand and SDNode transformation definitions.
 251: //===----------------------------------------------------------------------===//
 252: 
 253: class ImmAsmOperand<string prefix, int width, string suffix>
 254:     : AsmOperandClass {
 255:   let Name = prefix # "Imm" # width # suffix;
 256:   let DiagnosticType = !strconcat("Invalid", Name);
 257:   let RenderMethod = "addImmOperands";
 258: }
 259: 
 260: class SImmAsmOperand<int width, string suffix = "">
 261:     : ImmAsmOperand<"S", width, suffix> {
 262: }
 263: 
 264: class UImmAsmOperand<int width, string suffix = "">
 265:     : ImmAsmOperand<"U", width, suffix> {
 266: }
 267: 
 268: // A parse method for "$r*" or "$r*, 0", where the 0 is be silently ignored.
 269: // Only used for "AM*" instructions, in order to be compatible with GAS.
 270: def AtomicMemAsmOperand : AsmOperandClass {
 271:   let Name = "AtomicMemAsmOperand";
 272:   let RenderMethod = "addRegOperands";
 273:   let PredicateMethod = "isGPR";
 274:   let ParserMethod = "parseAtomicMemOp";
 275: }
 276: 
 277: def GPRMemAtomic : RegisterOperand<GPR> {
 278:   let ParserMatchClass = AtomicMemAsmOperand;
 279:   let PrintMethod = "printAtomicMemOp";
 280: }
```
- **EN**: This block declares or refines TableGen records such as `ImmAsmOperand`, `SImmAsmOperand`, `UImmAsmOperand`, `AtomicMemAsmOperand`, `GPRMemAtomic`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `ImmAsmOperand`, `SImmAsmOperand`, `UImmAsmOperand`, `AtomicMemAsmOperand`, `GPRMemAtomic` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-320 / 第 281-320 行
```tablegen
 281: 
 282: // A parameterized register class alternative to i32imm/i64imm from Target.td.
 283: def grlenimm : Operand<GRLenVT>;
 284: def imm32 : Operand<GRLenVT> {
 285:   let ParserMatchClass = ImmAsmOperand<"", 32, "">;
 286: }
 287: def imm64 : Operand<i64> {
 288:   let ParserMatchClass = ImmAsmOperand<"", 64, "">;
 289: }
 290: 
 291: def uimm1 : Operand<GRLenVT>, ImmLeaf<GRLenVT, [{return isUInt<1>(Imm);}]>{
 292:   let ParserMatchClass = UImmAsmOperand<1>;
 293: }
 294: 
 295: def uimm2 : Operand<GRLenVT>, ImmLeaf<GRLenVT, [{return isUInt<2>(Imm);}]> {
 296:   let ParserMatchClass = UImmAsmOperand<2>;
 297: }
 298: 
 299: def uimm2_plus1 : Operand<GRLenVT>,
 300:     ImmLeaf<GRLenVT, [{return isUInt<2>(Imm - 1);}]> {
 301:   let ParserMatchClass = UImmAsmOperand<2, "plus1">;
 302:   let EncoderMethod = "getImmOpValueSub1";
 303:   let DecoderMethod = "decodeUImmOperand<2, 1>";
 304: }
 305: 
 306: def uimm3 : Operand<GRLenVT>, ImmLeaf<GRLenVT, [{return isUInt<3>(Imm);}]> {
 307:   let ParserMatchClass = UImmAsmOperand<3>;
 308: }
 309: 
 310: def uimm4 : Operand<GRLenVT>, ImmLeaf<GRLenVT, [{return isUInt<4>(Imm);}]> {
 311:   let ParserMatchClass = UImmAsmOperand<4>;
 312: }
 313: 
 314: def uimm5 : Operand<GRLenVT>, ImmLeaf<GRLenVT, [{return isUInt<5>(Imm);}]> {
 315:   let ParserMatchClass = UImmAsmOperand<5>;
 316: }
 317: 
 318: def uimm6 : Operand<GRLenVT>, ImmLeaf<GRLenVT, [{return isUInt<6>(Imm);}]> {
 319:   let ParserMatchClass = UImmAsmOperand<6>;
 320: }
```
- **EN**: This block declares or refines TableGen records such as `grlenimm`, `imm32`, `imm64`, `uimm1`, `uimm2`, `uimm2_plus1`.
- **CN**: 该代码块声明或细化了 `grlenimm`, `imm32`, `imm64`, `uimm1`, `uimm2`, `uimm2_plus1` 等 TableGen 记录。

### Lines 321-360 / 第 321-360 行
```tablegen
 321: 
 322: def uimm7 : Operand<GRLenVT> {
 323:   let ParserMatchClass = UImmAsmOperand<7>;
 324: }
 325: 
 326: def uimm8 : Operand<GRLenVT>, ImmLeaf<GRLenVT, [{return isUInt<8>(Imm);}]> {
 327:   let ParserMatchClass = UImmAsmOperand<8>;
 328: }
 329: 
 330: class UImm12Operand : Operand<GRLenVT>,
 331:                       ImmLeaf <GRLenVT, [{return isUInt<12>(Imm);}]> {
 332:   let DecoderMethod = "decodeUImmOperand<12>";
 333: }
 334: 
 335: def uimm12 : UImm12Operand {
 336:   let ParserMatchClass = UImmAsmOperand<12>;
 337: }
 338: 
 339: def uimm12_ori : UImm12Operand {
 340:   let ParserMatchClass = UImmAsmOperand<12, "ori">;
 341: }
 342: 
 343: def uimm14 : Operand<GRLenVT>,
 344:              ImmLeaf <GRLenVT, [{return isUInt<14>(Imm);}]> {
 345:   let ParserMatchClass = UImmAsmOperand<14>;
 346: }
 347: 
 348: def uimm15 : Operand<GRLenVT>,
 349:              ImmLeaf <GRLenVT, [{return isUInt<15>(Imm);}]> {
 350:   let ParserMatchClass = UImmAsmOperand<15>;
 351: }
 352: 
 353: def simm5 : Operand<GRLenVT> {
 354:   let ParserMatchClass = SImmAsmOperand<5>;
 355:   let DecoderMethod = "decodeSImmOperand<5>";
 356: }
 357: 
 358: def simm8 : Operand<GRLenVT>,
 359:                  ImmLeaf<GRLenVT, [{return isInt<8>(Imm);}]> {
 360:   let ParserMatchClass = SImmAsmOperand<8>;
```
- **EN**: This block declares or refines TableGen records such as `uimm7`, `uimm8`, `UImm12Operand`, `uimm12`, `uimm12_ori`, `uimm14`.
- **CN**: 该代码块声明或细化了 `uimm7`, `uimm8`, `UImm12Operand`, `uimm12`, `uimm12_ori`, `uimm14` 等 TableGen 记录。

### Lines 361-400 / 第 361-400 行
```tablegen
 361:   let DecoderMethod = "decodeSImmOperand<8>";
 362: }
 363: 
 364: def simm8_lsl1 : Operand<GRLenVT>,
 365:                  ImmLeaf<GRLenVT, [{return isShiftedInt<8,1>(Imm);}]> {
 366:   let ParserMatchClass = SImmAsmOperand<8, "lsl1">;
 367:   let EncoderMethod = "getImmOpValueAsr<1>";
 368:   let DecoderMethod = "decodeSImmOperand<8, 1>";
 369: }
 370: 
 371: def simm8_lsl2 : Operand<GRLenVT>,
 372:                  ImmLeaf<GRLenVT, [{return isShiftedInt<8,2>(Imm);}]> {
 373:   let ParserMatchClass = SImmAsmOperand<8, "lsl2">;
 374:   let EncoderMethod = "getImmOpValueAsr<2>";
 375:   let DecoderMethod = "decodeSImmOperand<8, 2>";
 376: }
 377: 
 378: def simm8_lsl3 : Operand<GRLenVT>,
 379:                  ImmLeaf<GRLenVT, [{return isShiftedInt<8,3>(Imm);}]> {
 380:   let ParserMatchClass = SImmAsmOperand<8, "lsl3">;
 381:   let EncoderMethod = "getImmOpValueAsr<3>";
 382:   let DecoderMethod = "decodeSImmOperand<8, 3>";
 383: }
 384: 
 385: def simm9_lsl3 : Operand<GRLenVT>,
 386:                  ImmLeaf<GRLenVT, [{return isShiftedInt<9,3>(Imm);}]> {
 387:   let ParserMatchClass = SImmAsmOperand<9, "lsl3">;
 388:   let EncoderMethod = "getImmOpValueAsr<3>";
 389:   let DecoderMethod = "decodeSImmOperand<9, 3>";
 390: }
 391: 
 392: def simm10 : Operand<GRLenVT> {
 393:   let ParserMatchClass = SImmAsmOperand<10>;
 394: }
 395: 
 396: def simm10_lsl2 : Operand<GRLenVT>,
 397:                   ImmLeaf<GRLenVT, [{return isShiftedInt<10,2>(Imm);}]> {
 398:   let ParserMatchClass = SImmAsmOperand<10, "lsl2">;
 399:   let EncoderMethod = "getImmOpValueAsr<2>";
 400:   let DecoderMethod = "decodeSImmOperand<10, 2>";
```
- **EN**: This block declares or refines TableGen records such as `simm8_lsl1`, `simm8_lsl2`, `simm8_lsl3`, `simm9_lsl3`, `simm10`, `simm10_lsl2`.
- **CN**: 该代码块声明或细化了 `simm8_lsl1`, `simm8_lsl2`, `simm8_lsl3`, `simm9_lsl3`, `simm10`, `simm10_lsl2` 等 TableGen 记录。

### Lines 401-440 / 第 401-440 行
```tablegen
 401: }
 402: 
 403: def simm11_lsl1 : Operand<GRLenVT>,
 404:                   ImmLeaf<GRLenVT, [{return isShiftedInt<11,1>(Imm);}]> {
 405:   let ParserMatchClass = SImmAsmOperand<11, "lsl1">;
 406:   let EncoderMethod = "getImmOpValueAsr<1>";
 407:   let DecoderMethod = "decodeSImmOperand<11, 1>";
 408: }
 409: 
 410: class SImm12Operand : Operand<GRLenVT>,
 411:                       ImmLeaf <GRLenVT, [{return isInt<12>(Imm);}]> {
 412:   let DecoderMethod = "decodeSImmOperand<12>";
 413: }
 414: 
 415: def simm12 : SImm12Operand {
 416:   let ParserMatchClass = SImmAsmOperand<12>;
 417: }
 418: 
 419: def simm12_addlike : SImm12Operand {
 420:   let ParserMatchClass = SImmAsmOperand<12, "addlike">;
 421: }
 422: 
 423: def simm12_lu52id : SImm12Operand {
 424:   let ParserMatchClass = SImmAsmOperand<12, "lu52id">;
 425: }
 426: 
 427: def simm13 : Operand<GRLenVT> {
 428:   let ParserMatchClass = SImmAsmOperand<13>;
 429:   let DecoderMethod = "decodeSImmOperand<13>";
 430: }
 431: 
 432: def simm14_lsl2 : Operand<GRLenVT>,
 433:     ImmLeaf<GRLenVT, [{return isShiftedInt<14,2>(Imm);}]> {
 434:   let ParserMatchClass = SImmAsmOperand<14, "lsl2">;
 435:   let EncoderMethod = "getImmOpValueAsr<2>";
 436:   let DecoderMethod = "decodeSImmOperand<14, 2>";
 437: }
 438: 
 439: def simm16 : Operand<GRLenVT> {
 440:   let ParserMatchClass = SImmAsmOperand<16>;
```
- **EN**: This block declares or refines TableGen records such as `simm11_lsl1`, `SImm12Operand`, `simm12`, `simm12_addlike`, `simm12_lu52id`, `simm13`.
- **CN**: 该代码块声明或细化了 `simm11_lsl1`, `SImm12Operand`, `simm12`, `simm12_addlike`, `simm12_lu52id`, `simm13` 等 TableGen 记录。

### Lines 441-480 / 第 441-480 行
```tablegen
 441:   let DecoderMethod = "decodeSImmOperand<16>";
 442: }
 443: 
 444: def simm16_lsl2 : Operand<GRLenVT>,
 445:     ImmLeaf<GRLenVT, [{return isInt<16>(Imm>>2);}]> {
 446:   let ParserMatchClass = SImmAsmOperand<16, "lsl2">;
 447:   let EncoderMethod = "getImmOpValueAsr<2>";
 448:   let DecoderMethod = "decodeSImmOperand<16, 2>";
 449: }
 450: 
 451: def simm16_lsl2_br : Operand<OtherVT> {
 452:   let ParserMatchClass = SImmAsmOperand<16, "lsl2">;
 453:   let EncoderMethod = "getImmOpValueAsr<2>";
 454:   let DecoderMethod = "decodeSImmOperand<16, 2>";
 455: }
 456: 
 457: class SImm20Operand : Operand<GRLenVT> {
 458:   let DecoderMethod = "decodeSImmOperand<20>";
 459: }
 460: 
 461: def simm20 : SImm20Operand {
 462:   let ParserMatchClass = SImmAsmOperand<20>;
 463: }
 464: 
 465: def simm20_pcalau12i : SImm20Operand {
 466:   let ParserMatchClass = SImmAsmOperand<20, "pcalau12i">;
 467: }
 468: 
 469: def simm20_lu12iw : SImm20Operand {
 470:   let ParserMatchClass = SImmAsmOperand<20, "lu12iw">;
 471: }
 472: 
 473: def simm20_lu32id : SImm20Operand {
 474:   let ParserMatchClass = SImmAsmOperand<20, "lu32id">;
 475: }
 476: 
 477: def simm20_pcaddu12i : SImm20Operand {
 478:   let ParserMatchClass = SImmAsmOperand<20, "pcaddu12i">;
 479: }
 480: 
```
- **EN**: This block declares or refines TableGen records such as `simm16_lsl2`, `simm16_lsl2_br`, `SImm20Operand`, `simm20`, `simm20_pcalau12i`, `simm20_lu12iw`.
- **CN**: 该代码块声明或细化了 `simm16_lsl2`, `simm16_lsl2_br`, `SImm20Operand`, `simm20`, `simm20_pcalau12i`, `simm20_lu12iw` 等 TableGen 记录。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: def simm20_pcaddu18i : SImm20Operand {
 482:   let ParserMatchClass = SImmAsmOperand<20, "pcaddu18i">;
 483: }
 484: 
 485: def simm20_pcaddi : SImm20Operand {
 486:   let ParserMatchClass = SImmAsmOperand<20, "pcaddi">;
 487: }
 488: 
 489: def simm21_lsl2 : Operand<OtherVT> {
 490:   let ParserMatchClass = SImmAsmOperand<21, "lsl2">;
 491:   let EncoderMethod = "getImmOpValueAsr<2>";
 492:   let DecoderMethod = "decodeSImmOperand<21, 2>";
 493: }
 494: 
 495: def SImm26OperandB: AsmOperandClass {
 496:   let Name = "SImm26OperandB";
 497:   let PredicateMethod = "isSImm26Operand";
 498:   let RenderMethod = "addImmOperands";
 499:   let DiagnosticType = "InvalidSImm26Operand";
 500:   let ParserMethod = "parseImmediate";
 501: }
 502: 
 503: // A symbol or an imm used in B/PseudoBR.
 504: def simm26_b : Operand<OtherVT> {
 505:   let ParserMatchClass = SImm26OperandB;
 506:   let EncoderMethod = "getImmOpValueAsr<2>";
 507:   let DecoderMethod = "decodeSImmOperand<26, 2>";
 508: }
 509: 
 510: def SImm26OperandBL: AsmOperandClass {
 511:   let Name = "SImm26OperandBL";
 512:   let PredicateMethod = "isSImm26Operand";
 513:   let RenderMethod = "addImmOperands";
 514:   let DiagnosticType = "InvalidSImm26Operand";
 515:   let ParserMethod = "parseSImm26Operand";
 516: }
 517: 
 518: // A symbol or an imm used in BL/PseudoCALL_SMALL/PseudoTAIL_SMALL.
 519: def simm26_symbol : Operand<GRLenVT> {
 520:   let ParserMatchClass = SImm26OperandBL;
```
- **EN**: This block declares or refines TableGen records such as `simm20_pcaddu18i`, `simm20_pcaddi`, `simm21_lsl2`, `SImm26OperandB:`, `simm26_b`, `SImm26OperandBL:`.
- **CN**: 该代码块声明或细化了 `simm20_pcaddu18i`, `simm20_pcaddi`, `simm21_lsl2`, `SImm26OperandB:`, `simm26_b`, `SImm26OperandBL:` 等 TableGen 记录。

### Lines 521-560 / 第 521-560 行
```tablegen
 521:   let EncoderMethod = "getImmOpValueAsr<2>";
 522:   let DecoderMethod = "decodeSImmOperand<26, 2>";
 523: }
 524: 
 525: // A 32-bit signed immediate with the lowest 16 bits zeroed, suitable for
 526: // direct use with `addu16i.d`.
 527: def simm16_lsl16 : Operand<GRLenVT>,
 528:     ImmLeaf<GRLenVT, [{return isShiftedInt<16, 16>(Imm);}]>;
 529: 
 530: // A 32-bit signed immediate expressible with a pair of `addu16i.d + addi` for
 531: // use in additions.
 532: def simm32_hi16_lo12: Operand<GRLenVT>, ImmLeaf<GRLenVT, [{
 533:   return !isInt<12>(Imm) && isShiftedInt<16, 16>(Imm - SignExtend64<12>(Imm));
 534: }]>;
 535: 
 536: def BareSymbol : AsmOperandClass {
 537:   let Name = "BareSymbol";
 538:   let RenderMethod = "addImmOperands";
 539:   let DiagnosticType = "InvalidBareSymbol";
 540:   let ParserMethod = "parseImmediate";
 541: }
 542: 
 543: // A bare symbol used in "PseudoLA_*" instructions.
 544: def bare_symbol : Operand<GRLenVT> {
 545:   let ParserMatchClass = BareSymbol;
 546: }
 547: 
 548: def TPRelAddSymbol : AsmOperandClass {
 549:   let Name = "TPRelAddSymbol";
 550:   let RenderMethod = "addImmOperands";
 551:   let DiagnosticType = "InvalidTPRelAddSymbol";
 552:   let ParserMethod = "parseOperandWithModifier";
 553: }
 554: 
 555: // A bare symbol with the %le_add_r variant.
 556: def tprel_add_symbol : Operand<GRLenVT> {
 557:   let ParserMatchClass = TPRelAddSymbol;
 558: }
 559: 
 560: 
```
- **EN**: This block declares or refines TableGen records such as `simm16_lsl16`, `simm32_hi16_lo12:`, `BareSymbol`, `bare_symbol`, `TPRelAddSymbol`, `tprel_add_symbol`.
- **CN**: 该代码块声明或细化了 `simm16_lsl16`, `simm32_hi16_lo12:`, `BareSymbol`, `bare_symbol`, `TPRelAddSymbol`, `tprel_add_symbol` 等 TableGen 记录。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: // Standalone (codegen-only) immleaf patterns.
 562: 
 563: // A 12-bit signed immediate plus one where the imm range will be [-2047, 2048].
 564: def simm12_plus1 : ImmLeaf<GRLenVT,
 565:   [{return (isInt<12>(Imm) && Imm != -2048) || Imm == 2048;}]>;
 566: 
 567: // Return the negation of an immediate value.
 568: def NegImm : SDNodeXForm<imm, [{
 569:   return CurDAG->getSignedTargetConstant(-N->getSExtValue(), SDLoc(N),
 570:                                          N->getValueType(0));
 571: }]>;
 572: 
 573: // FP immediate patterns.
 574: def fpimm0    : PatLeaf<(fpimm), [{return N->isExactlyValue(+0.0);}]>;
 575: def fpimm0neg : PatLeaf<(fpimm), [{return N->isExactlyValue(-0.0);}]>;
 576: def fpimm1    : PatLeaf<(fpimm), [{return N->isExactlyValue(+1.0);}]>;
 577: 
 578: // Return an immediate subtracted from 32.
 579: def ImmSubFrom32 : SDNodeXForm<imm, [{
 580:   return CurDAG->getTargetConstant(32 - N->getZExtValue(), SDLoc(N),
 581:                                    N->getValueType(0));
 582: }]>;
 583: 
 584: // Return the lowest 12 bits of the signed immediate.
 585: def LO12: SDNodeXForm<imm, [{
 586:   return CurDAG->getSignedTargetConstant(SignExtend64<12>(N->getSExtValue()),
 587:                                          SDLoc(N), N->getValueType(0));
 588: }]>;
 589: 
 590: // Return the higher 16 bits of the signed immediate.
 591: def HI16 : SDNodeXForm<imm, [{
 592:   return CurDAG->getSignedTargetConstant(N->getSExtValue() >> 16, SDLoc(N),
 593:                                          N->getValueType(0));
 594: }]>;
 595: 
 596: // Return the higher 16 bits of the signed immediate, adjusted for use within an
 597: // `addu16i.d + addi` pair.
 598: def HI16ForAddu16idAddiPair: SDNodeXForm<imm, [{
 599:   auto Imm = N->getSExtValue();
 600:   return CurDAG->getSignedTargetConstant((Imm - SignExtend64<12>(Imm)) >> 16,
```
- **EN**: This block declares or refines TableGen records such as `simm12_plus1`, `NegImm`, `fpimm0`, `fpimm0neg`, `fpimm1`, `ImmSubFrom32`.
- **CN**: 该代码块声明或细化了 `simm12_plus1`, `NegImm`, `fpimm0`, `fpimm0neg`, `fpimm1`, `ImmSubFrom32` 等 TableGen 记录。

### Lines 601-640 / 第 601-640 行
```tablegen
 601:                                           SDLoc(N), N->getValueType(0));
 602: }]>;
 603: 
 604: def BaseAddr : ComplexPattern<iPTR, 1, "SelectBaseAddr">;
 605: def AddrConstant : ComplexPattern<iPTR, 2, "SelectAddrConstant">;
 606: def NonFIBaseAddr : ComplexPattern<iPTR, 1, "selectNonFIBaseAddr">;
 607: def AddrRegImm : ComplexPattern<iPTR, 2, "SelectAddrRegImm12">;
 608: 
 609: def fma_nsz : PatFrag<(ops node:$fj, node:$fk, node:$fa),
 610:                       (fma node:$fj, node:$fk, node:$fa), [{
 611:   return N->getFlags().hasNoSignedZeros();
 612: }]>;
 613: 
 614: // Check if (add r, imm) can be optimized to (ADDI (ADDI r, imm0), imm1),
 615: // in which imm = imm0 + imm1, and both imm0 & imm1 are simm12.
 616: def AddiPair : PatLeaf<(imm), [{
 617:   if (!N->hasOneUse())
 618:     return false;
 619:   // The immediate operand must be in range [-4096,-2049] or [2048,4094].
 620:   int64_t Imm = N->getSExtValue();
 621:   return (-4096 <= Imm && Imm <= -2049) || (2048 <= Imm && Imm <= 4094);
 622: }]>;
 623: 
 624: // Return -2048 if immediate is negative or 2047 if positive.
 625: def AddiPairImmLarge : SDNodeXForm<imm, [{
 626:   int64_t Imm = N->getSExtValue() < 0 ? -2048 : 2047;
 627:   return CurDAG->getSignedTargetConstant(Imm, SDLoc(N),
 628:                                          N->getValueType(0));
 629: }]>;
 630: 
 631: // Return imm - (imm < 0 ? -2048 : 2047).
 632: def AddiPairImmSmall : SDNodeXForm<imm, [{
 633:   int64_t Imm = N->getSExtValue();
 634:   int64_t Adj = Imm < 0 ? -2048 : 2047;
 635:   return CurDAG->getSignedTargetConstant(Imm - Adj, SDLoc(N),
 636:                                          N->getValueType(0));
 637: }]>;
 638: 
 639: // Check if (mul r, imm) can be optimized to (SLLI (ALSL r, r, i0), i1),
 640: // in which imm = (1 + (1 << i0)) << i1.
```
- **EN**: This block declares or refines TableGen records such as `BaseAddr`, `AddrConstant`, `NonFIBaseAddr`, `AddrRegImm`, `fma_nsz`, `AddiPair`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `BaseAddr`, `AddrConstant`, `NonFIBaseAddr`, `AddrRegImm`, `fma_nsz`, `AddiPair` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 641-680 / 第 641-680 行
```tablegen
 641: def AlslSlliImm : PatLeaf<(imm), [{
 642:   if (!N->hasOneUse())
 643:     return false;
 644:   uint64_t Imm = N->getZExtValue();
 645:   unsigned I1 = llvm::countr_zero(Imm);
 646:   uint64_t Rem = Imm >> I1;
 647:   return Rem == 3 || Rem == 5 || Rem == 9 || Rem == 17;
 648: }]>;
 649: 
 650: def AlslSlliImmI1 : SDNodeXForm<imm, [{
 651:   uint64_t Imm = N->getZExtValue();
 652:   unsigned I1 = llvm::countr_zero(Imm);
 653:   return CurDAG->getTargetConstant(I1, SDLoc(N),
 654:                                    N->getValueType(0));
 655: }]>;
 656: 
 657: def AlslSlliImmI0 : SDNodeXForm<imm, [{
 658:   uint64_t Imm = N->getZExtValue();
 659:   unsigned I1 = llvm::countr_zero(Imm);
 660:   uint64_t I0;
 661:   switch (Imm >> I1) {
 662:   case 3:  I0 = 1; break;
 663:   case 5:  I0 = 2; break;
 664:   case 9:  I0 = 3; break;
 665:   default: I0 = 4; break;
 666:   }
 667:   return CurDAG->getTargetConstant(I0, SDLoc(N),
 668:                                    N->getValueType(0));
 669: }]>;
 670: 
 671: // Check if (and r, imm) can be optimized to (BSTRINS r, R0, msb, lsb),
 672: // in which imm = ~((2^^(msb-lsb+1) - 1) << lsb).
 673: def BstrinsImm : PatLeaf<(imm), [{
 674:   if (!N->hasOneUse())
 675:     return false;
 676:   uint64_t Imm = N->getZExtValue();
 677:   // andi can be used instead if Imm <= 0xfff.
 678:   if (Imm <= 0xfff)
 679:     return false;
 680:   unsigned MaskIdx, MaskLen;
```
- **EN**: This block declares or refines TableGen records such as `AlslSlliImm`, `AlslSlliImmI1`, `AlslSlliImmI0`, `BstrinsImm`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `AlslSlliImm`, `AlslSlliImmI1`, `AlslSlliImmI0`, `BstrinsImm` 等 TableGen 记录。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 681-720 / 第 681-720 行
```tablegen
 681:   return N->getValueType(0).getSizeInBits() == 32
 682:              ? llvm::isShiftedMask_32(~Imm, MaskIdx, MaskLen)
 683:              : llvm::isShiftedMask_64(~Imm, MaskIdx, MaskLen);
 684: }]>;
 685: 
 686: def BstrinsMsb: SDNodeXForm<imm, [{
 687:   uint64_t Imm = N->getZExtValue();
 688:   unsigned MaskIdx, MaskLen;
 689:   N->getValueType(0).getSizeInBits() == 32
 690:       ? llvm::isShiftedMask_32(~Imm, MaskIdx, MaskLen)
 691:       : llvm::isShiftedMask_64(~Imm, MaskIdx, MaskLen);
 692:   return CurDAG->getTargetConstant(MaskIdx + MaskLen - 1, SDLoc(N),
 693:                                    N->getValueType(0));
 694: }]>;
 695: 
 696: def BstrinsLsb: SDNodeXForm<imm, [{
 697:   uint64_t Imm = N->getZExtValue();
 698:   unsigned MaskIdx, MaskLen;
 699:   N->getValueType(0).getSizeInBits() == 32
 700:       ? llvm::isShiftedMask_32(~Imm, MaskIdx, MaskLen)
 701:       : llvm::isShiftedMask_64(~Imm, MaskIdx, MaskLen);
 702:   return CurDAG->getTargetConstant(MaskIdx, SDLoc(N), N->getValueType(0));
 703: }]>;
 704: 
 705: //===----------------------------------------------------------------------===//
 706: // Instruction Formats
 707: //===----------------------------------------------------------------------===//
 708: 
 709: include "LoongArchInstrFormats.td"
 710: include "LoongArchFloatInstrFormats.td"
 711: include "LoongArchLSXInstrFormats.td"
 712: include "LoongArchLASXInstrFormats.td"
 713: include "LoongArchLBTInstrFormats.td"
 714: 
 715: //===----------------------------------------------------------------------===//
 716: // Instruction Class Templates
 717: //===----------------------------------------------------------------------===//
 718: 
 719: let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
 720: class ALU_3R<bits<32> op>
```
- **EN**: It composes TableGen records by including `LoongArchInstrFormats.td`, `LoongArchFloatInstrFormats.td`, `LoongArchLSXInstrFormats.td`, `LoongArchLASXInstrFormats.td`, `LoongArchLBTInstrFormats.td` and reusing previously declared backend fragments. This block declares or refines TableGen records such as `BstrinsMsb:`, `BstrinsLsb:`, `ALU_3R`.
- **CN**: 它通过包含 `LoongArchInstrFormats.td`, `LoongArchFloatInstrFormats.td`, `LoongArchLSXInstrFormats.td`, `LoongArchLASXInstrFormats.td`, `LoongArchLBTInstrFormats.td` 组合 TableGen 记录，并复用此前声明的后端片段。 该代码块声明或细化了 `BstrinsMsb:`, `BstrinsLsb:`, `ALU_3R` 等 TableGen 记录。

### Lines 721-760 / 第 721-760 行
```tablegen
 721:     : Fmt3R<op, (outs GPR:$rd), (ins GPR:$rj, GPR:$rk), "$rd, $rj, $rk">;
 722: class ALU_2R<bits<32> op>
 723:     : Fmt2R<op, (outs GPR:$rd), (ins GPR:$rj), "$rd, $rj">;
 724: 
 725: class ALU_3RI2<bits<32> op, Operand ImmOpnd>
 726:     : Fmt3RI2<op, (outs GPR:$rd), (ins GPR:$rj, GPR:$rk, ImmOpnd:$imm2),
 727:               "$rd, $rj, $rk, $imm2">;
 728: class ALU_3RI3<bits<32> op, Operand ImmOpnd>
 729:     : Fmt3RI3<op, (outs GPR:$rd), (ins GPR:$rj, GPR:$rk, ImmOpnd:$imm3),
 730:               "$rd, $rj, $rk, $imm3">;
 731: class ALU_2RI5<bits<32> op, Operand ImmOpnd>
 732:     : Fmt2RI5<op, (outs GPR:$rd), (ins GPR:$rj, ImmOpnd:$imm5),
 733:               "$rd, $rj, $imm5">;
 734: class ALU_2RI6<bits<32> op, Operand ImmOpnd>
 735:     : Fmt2RI6<op, (outs GPR:$rd), (ins GPR:$rj, ImmOpnd:$imm6),
 736:               "$rd, $rj, $imm6">;
 737: class ALU_2RI12<bits<32> op, Operand ImmOpnd>
 738:     : Fmt2RI12<op, (outs GPR:$rd), (ins GPR:$rj, ImmOpnd:$imm12),
 739:                "$rd, $rj, $imm12">;
 740: class ALU_2RI16<bits<32> op, Operand ImmOpnd>
 741:     : Fmt2RI16<op, (outs GPR:$rd), (ins GPR:$rj, ImmOpnd:$imm16),
 742:                "$rd, $rj, $imm16">;
 743: class ALU_1RI20<bits<32> op, Operand ImmOpnd>
 744:     : Fmt1RI20<op, (outs GPR:$rd), (ins ImmOpnd:$imm20), "$rd, $imm20">;
 745: } // hasSideEffects = 0, mayLoad = 0, mayStore = 0
 746: 
 747: let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in
 748: class MISC_I15<bits<32> op>
 749:     : FmtI15<op, (outs), (ins uimm15:$imm15), "$imm15">;
 750: 
 751: let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in
 752: class RDTIME_2R<bits<32> op>
 753:     : Fmt2R<op, (outs GPR:$rd, GPR:$rj), (ins), "$rd, $rj">;
 754: 
 755: let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
 756: class BrCC_2RI16<bits<32> op>
 757:     : Fmt2RI16<op, (outs), (ins GPR:$rj, GPR:$rd, simm16_lsl2_br:$imm16),
 758:                "$rj, $rd, $imm16"> {
 759:   let isBranch = 1;
 760:   let isTerminator = 1;
```
- **EN**: This block declares or refines TableGen records such as `ALU_2R`, `ALU_3RI2`, `ALU_3RI3`, `ALU_2RI5`, `ALU_2RI6`, `ALU_2RI12`.
- **CN**: 该代码块声明或细化了 `ALU_2R`, `ALU_3RI2`, `ALU_3RI3`, `ALU_2RI5`, `ALU_2RI6`, `ALU_2RI12` 等 TableGen 记录。

### Lines 761-800 / 第 761-800 行
```tablegen
 761: }
 762: class BrCCZ_1RI21<bits<32> op>
 763:     : Fmt1RI21<op, (outs), (ins GPR:$rj, simm21_lsl2:$imm21),
 764:                "$rj, $imm21"> {
 765:   let isBranch = 1;
 766:   let isTerminator = 1;
 767: }
 768: class Br_I26<bits<32> op>
 769:     : FmtI26<op, (outs), (ins simm26_b:$imm26), "$imm26"> {
 770:   let isBranch = 1;
 771:   let isTerminator = 1;
 772:   let isBarrier = 1;
 773: }
 774: } // hasSideEffects = 0, mayLoad = 0, mayStore = 0
 775: 
 776: let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in {
 777: class LOAD_3R<bits<32> op>
 778:     : Fmt3R<op, (outs GPR:$rd), (ins GPR:$rj, GPR:$rk), "$rd, $rj, $rk">;
 779: class LOAD_2RI12<bits<32> op>
 780:     : Fmt2RI12<op, (outs GPR:$rd), (ins GPR:$rj, simm12_addlike:$imm12),
 781:                "$rd, $rj, $imm12">;
 782: class LOAD_2RI14<bits<32> op>
 783:     : Fmt2RI14<op, (outs GPR:$rd), (ins GPR:$rj, simm14_lsl2:$imm14),
 784:                "$rd, $rj, $imm14">;
 785: } // hasSideEffects = 0, mayLoad = 1, mayStore = 0
 786: 
 787: let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in {
 788: class STORE_3R<bits<32> op>
 789:     : Fmt3R<op, (outs), (ins GPR:$rd, GPR:$rj, GPR:$rk),
 790:             "$rd, $rj, $rk">;
 791: class STORE_2RI12<bits<32> op>
 792:     : Fmt2RI12<op, (outs), (ins GPR:$rd, GPR:$rj, simm12_addlike:$imm12),
 793:                "$rd, $rj, $imm12">;
 794: class STORE_2RI14<bits<32> op>
 795:     : Fmt2RI14<op, (outs), (ins GPR:$rd, GPR:$rj, simm14_lsl2:$imm14),
 796:                "$rd, $rj, $imm14">;
 797: } // hasSideEffects = 0, mayLoad = 0, mayStore = 1
 798: 
 799: let hasSideEffects = 0, mayLoad = 1, mayStore = 1,
 800:     IsSubjectToAMORdConstraint = 1 in {
```
- **EN**: This block declares or refines TableGen records such as `BrCCZ_1RI21`, `Br_I26`, `LOAD_3R`, `LOAD_2RI12`, `LOAD_2RI14`, `STORE_3R`.
- **CN**: 该代码块声明或细化了 `BrCCZ_1RI21`, `Br_I26`, `LOAD_3R`, `LOAD_2RI12`, `LOAD_2RI14`, `STORE_3R` 等 TableGen 记录。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: class AM_3R<bits<32> op>
 802:     : Fmt3R<op, (outs GPR:$rd), (ins GPR:$rk, GPRMemAtomic:$rj),
 803:             "$rd, $rk, $rj"> {
 804:   let Constraints = "@earlyclobber $rd";
 805: }
 806: 
 807: class AU_3R<bits<32> op> : AM_3R<op> {
 808:   let DecoderMethod = "DecodeAMOrUDInstruction";
 809: }
 810: 
 811: class AMCAS_3R<bits<32> op>
 812:     : Fmt3R<op, (outs GPR:$dst), (ins GPR:$rd, GPR:$rk, GPRMemAtomic:$rj),
 813:             "$rd, $rk, $rj"> {
 814:   let Constraints = "@earlyclobber $dst, $dst = $rd";
 815:   let IsAMCAS = 1;
 816: }
 817: } // hasSideEffects = 0, mayLoad = 1, mayStore = 1,
 818:   // IsSubjectToAMORdConstraint = 1
 819: 
 820: let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in {
 821: class LLBase<bits<32> op>
 822:     : Fmt2RI14<op, (outs GPR:$rd), (ins GPR:$rj, simm14_lsl2:$imm14),
 823:                "$rd, $rj, $imm14">;
 824: class LLBase_ACQ<bits<32> op>
 825:     : Fmt2R<op, (outs GPR:$rd), (ins GPR:$rj), "$rd, $rj">;
 826: }
 827: 
 828: let hasSideEffects = 0, mayLoad = 0, mayStore = 1, Constraints = "$rd = $dst" in {
 829: class SCBase<bits<32> op>
 830:     : Fmt2RI14<op, (outs GPR:$dst), (ins GPR:$rd, GPR:$rj, simm14_lsl2:$imm14),
 831:                "$rd, $rj, $imm14">;
 832: class SCBase_128<bits<32> op>
 833:     : Fmt3R<op, (outs GPR:$dst), (ins GPR:$rd, GPR:$rk, GPR:$rj),
 834:                "$rd, $rk, $rj">;
 835: class SCBase_REL<bits<32> op>
 836:     : Fmt2R<op, (outs GPR:$dst), (ins GPR:$rd, GPR:$rj), "$rd, $rj">;
 837: }
 838: 
 839: let hasSideEffects = 1 in
 840: class IOCSRRD<bits<32> op>
```
- **EN**: This block declares or refines TableGen records such as `AM_3R`, `AU_3R`, `AMCAS_3R`, `LLBase`, `LLBase_ACQ`, `SCBase`.
- **CN**: 该代码块声明或细化了 `AM_3R`, `AU_3R`, `AMCAS_3R`, `LLBase`, `LLBase_ACQ`, `SCBase` 等 TableGen 记录。

### Lines 841-880 / 第 841-880 行
```tablegen
 841:     : Fmt2R<op, (outs GPR:$rd), (ins GPR:$rj), "$rd, $rj">;
 842: 
 843: let hasSideEffects = 1 in
 844: class IOCSRWR<bits<32> op>
 845:     : Fmt2R<op, (outs), (ins GPR:$rd, GPR:$rj), "$rd, $rj">;
 846: 
 847: //===----------------------------------------------------------------------===//
 848: // Basic Integer Instructions
 849: //===----------------------------------------------------------------------===//
 850: 
 851: // Arithmetic Operation Instructions
 852: def ADD_W : ALU_3R<0x00100000>;
 853: def SUB_W : ALU_3R<0x00110000>;
 854: def ADDI_W : ALU_2RI12<0x02800000, simm12_addlike>;
 855: let isReMaterializable = 1 in {
 856: def LU12I_W : ALU_1RI20<0x14000000, simm20_lu12iw>;
 857: }
 858: def SLT  : ALU_3R<0x00120000>;
 859: def SLTU : ALU_3R<0x00128000>;
 860: def SLTI  : ALU_2RI12<0x02000000, simm12>;
 861: def SLTUI : ALU_2RI12<0x02400000, simm12>;
 862: def PCADDU12I : ALU_1RI20<0x1c000000, simm20_pcaddu12i>;
 863: def AND  : ALU_3R<0x00148000>;
 864: def OR   : ALU_3R<0x00150000>;
 865: def NOR  : ALU_3R<0x00140000>;
 866: def XOR  : ALU_3R<0x00158000>;
 867: def ANDI : ALU_2RI12<0x03400000, uimm12>;
 868: // See LoongArchInstrInfo::isAsCheapAsAMove for more details.
 869: let isReMaterializable = 1, isAsCheapAsAMove = 1 in {
 870: def ORI  : ALU_2RI12<0x03800000, uimm12_ori>;
 871: def XORI : ALU_2RI12<0x03c00000, uimm12>;
 872: }
 873: def MUL_W   : ALU_3R<0x001c0000>;
 874: def MULH_W  : ALU_3R<0x001c8000>;
 875: def MULH_WU : ALU_3R<0x001d0000>;
 876: let usesCustomInserter = true in {
 877: def DIV_W   : ALU_3R<0x00200000>;
 878: def MOD_W   : ALU_3R<0x00208000>;
 879: def DIV_WU  : ALU_3R<0x00210000>;
 880: def MOD_WU  : ALU_3R<0x00218000>;
```
- **EN**: This block declares or refines TableGen records such as `IOCSRWR`, `ADD_W`, `SUB_W`, `ADDI_W`, `LU12I_W`, `SLT`.
- **CN**: 该代码块声明或细化了 `IOCSRWR`, `ADD_W`, `SUB_W`, `ADDI_W`, `LU12I_W`, `SLT` 等 TableGen 记录。

### Lines 881-920 / 第 881-920 行
```tablegen
 881: } // usesCustomInserter = true
 882: 
 883: // Bit-shift Instructions
 884: def SLL_W  : ALU_3R<0x00170000>;
 885: def SRL_W  : ALU_3R<0x00178000>;
 886: def SRA_W  : ALU_3R<0x00180000>;
 887: 
 888: def SLLI_W  : ALU_2RI5<0x00408000, uimm5>;
 889: def SRLI_W  : ALU_2RI5<0x00448000, uimm5>;
 890: def SRAI_W  : ALU_2RI5<0x00488000, uimm5>;
 891: 
 892: // Branch Instructions
 893: def BEQ  : BrCC_2RI16<0x58000000>;
 894: def BNE  : BrCC_2RI16<0x5c000000>;
 895: def BLT  : BrCC_2RI16<0x60000000>;
 896: def BGE  : BrCC_2RI16<0x64000000>;
 897: def BLTU : BrCC_2RI16<0x68000000>;
 898: def BGEU : BrCC_2RI16<0x6c000000>;
 899: def B : Br_I26<0x50000000>;
 900: 
 901: let hasSideEffects = 0, mayLoad = 0, mayStore = 0, isCall = 1, Defs=[R1] in
 902: def BL : FmtI26<0x54000000, (outs), (ins simm26_symbol:$imm26), "$imm26">;
 903: let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
 904: def JIRL : Fmt2RI16<0x4c000000, (outs GPR:$rd),
 905:                     (ins GPR:$rj, simm16_lsl2:$imm16), "$rd, $rj, $imm16">;
 906: 
 907: // Common Memory Access Instructions
 908: def LD_B  : LOAD_2RI12<0x28000000>;
 909: def LD_H  : LOAD_2RI12<0x28400000>;
 910: def LD_W  : LOAD_2RI12<0x28800000>;
 911: def LD_BU : LOAD_2RI12<0x2a000000>;
 912: def LD_HU : LOAD_2RI12<0x2a400000>;
 913: def ST_B : STORE_2RI12<0x29000000>;
 914: def ST_H : STORE_2RI12<0x29400000>;
 915: def ST_W : STORE_2RI12<0x29800000>;
 916: let hasSideEffects = 0, mayLoad = 1, mayStore = 1 in
 917: def PRELD : FmtPRELD<(outs), (ins uimm5:$imm5, GPR:$rj, simm12:$imm12),
 918:                      "$imm5, $rj, $imm12">;
 919: 
 920: // Atomic Memory Access Instructions
```
- **EN**: This block declares or refines TableGen records such as `SLL_W`, `SRL_W`, `SRA_W`, `SLLI_W`, `SRLI_W`, `SRAI_W`.
- **CN**: 该代码块声明或细化了 `SLL_W`, `SRL_W`, `SRA_W`, `SLLI_W`, `SRLI_W`, `SRAI_W` 等 TableGen 记录。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: def LL_W : LLBase<0x20000000>;
 922: def SC_W : SCBase<0x21000000>;
 923: 
 924: // Barrier Instructions
 925: def DBAR : MISC_I15<0x38720000>;
 926: def IBAR : MISC_I15<0x38728000>;
 927: 
 928: // Other Miscellaneous Instructions
 929: def SYSCALL : MISC_I15<0x002b0000>;
 930: def BREAK   : MISC_I15<0x002a0000>;
 931: def RDTIMEL_W : RDTIME_2R<0x00006000>;
 932: def RDTIMEH_W : RDTIME_2R<0x00006400>;
 933: 
 934: let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in
 935: def UD : FmtUD<(outs), (ins uimm5:$imm5), "$imm5">;
 936: 
 937: // The CPUCFG instruction offers a reliable way to probing CPU features.
 938: // Although support is not guaranteed on LA32R, having compiler support
 939: // nevertheless enables applications to rely on its presence, potentially
 940: // via kernel emulation if not available natively.
 941: def CPUCFG : ALU_2R<0x00006c00>;
 942: 
 943: // Cache Maintenance Instructions
 944: def CACOP : FmtCACOP<(outs), (ins uimm5:$op, GPR:$rj, simm12:$imm12),
 945:                      "$op, $rj, $imm12">;
 946: 
 947: let Predicates = [Has32S] in {
 948: // Arithmetic Operation Instructions
 949: def ALSL_W : ALU_3RI2<0x00040000, uimm2_plus1>;
 950: def ANDN : ALU_3R<0x00168000>;
 951: def ORN  : ALU_3R<0x00160000>;
 952: def PCADDI    : ALU_1RI20<0x18000000, simm20_pcaddi>;
 953: def PCALAU12I : ALU_1RI20<0x1a000000, simm20_pcalau12i>;
 954: 
 955: // Bit-shift Instructions
 956: def ROTR_W : ALU_3R<0x001b0000>;
 957: def ROTRI_W : ALU_2RI5<0x004c8000, uimm5>;
 958: 
 959: // Bit-manipulation Instructions
 960: def EXT_W_B : ALU_2R<0x00005c00>;
```
- **EN**: This block declares or refines TableGen records such as `LL_W`, `SC_W`, `DBAR`, `IBAR`, `SYSCALL`, `BREAK`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `LL_W`, `SC_W`, `DBAR`, `IBAR`, `SYSCALL`, `BREAK` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: def EXT_W_H : ALU_2R<0x00005800>;
 962: def CLO_W   : ALU_2R<0x00001000>;
 963: def CLZ_W   : ALU_2R<0x00001400>;
 964: def CTO_W   : ALU_2R<0x00001800>;
 965: def CTZ_W   : ALU_2R<0x00001c00>;
 966: def BYTEPICK_W : ALU_3RI2<0x00080000, uimm2>;
 967: def REVB_2H   : ALU_2R<0x00003000>;
 968: def BITREV_4B : ALU_2R<0x00004800>;
 969: def BITREV_W  : ALU_2R<0x00005000>;
 970: let Constraints = "$rd = $dst" in {
 971: def BSTRINS_W  : FmtBSTR_W<0x00600000, (outs GPR:$dst),
 972:                            (ins GPR:$rd, GPR:$rj, uimm5:$msbw, uimm5:$lsbw),
 973:                            "$rd, $rj, $msbw, $lsbw">;
 974: }
 975: def BSTRPICK_W : FmtBSTR_W<0x00608000, (outs GPR:$rd),
 976:                            (ins GPR:$rj, uimm5:$msbw, uimm5:$lsbw),
 977:                            "$rd, $rj, $msbw, $lsbw">;
 978: def MASKEQZ : ALU_3R<0x00130000>;
 979: def MASKNEZ : ALU_3R<0x00138000>;
 980: 
 981: // Branch Instructions
 982: def BEQZ : BrCCZ_1RI21<0x40000000>;
 983: def BNEZ : BrCCZ_1RI21<0x44000000>;
 984: } // Predicates = [Has32S]
 985: 
 986: /// LA64 instructions
 987: 
 988: let Predicates = [IsLA64] in {
 989: 
 990: // Arithmetic Operation Instructions for 64-bits
 991: def ADD_D : ALU_3R<0x00108000>;
 992: def SUB_D : ALU_3R<0x00118000>;
 993: // ADDI_D isn't always rematerializable, but isReMaterializable will be used as
 994: // a hint which is verified in isReMaterializableImpl.
 995: // See LoongArchInstrInfo::isAsCheapAsAMove for more details.
 996: let isReMaterializable = 1, isAsCheapAsAMove = 1 in {
 997: def ADDI_D : ALU_2RI12<0x02c00000, simm12_addlike>;
 998: }
 999: def ADDU16I_D : ALU_2RI16<0x10000000, simm16>;
1000: def ALSL_WU : ALU_3RI2<0x00060000, uimm2_plus1>;
```
- **EN**: This block declares or refines TableGen records such as `EXT_W_H`, `CLO_W`, `CLZ_W`, `CTO_W`, `CTZ_W`, `BYTEPICK_W`.
- **CN**: 该代码块声明或细化了 `EXT_W_H`, `CLO_W`, `CLZ_W`, `CTO_W`, `CTZ_W`, `BYTEPICK_W` 等 TableGen 记录。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: def ALSL_D  : ALU_3RI2<0x002c0000, uimm2_plus1>;
1002: let Constraints = "$rd = $dst" in {
1003: let hasSideEffects = 0, mayLoad = 0, mayStore = 0,
1004:     isReMaterializable = 1 in
1005: def LU32I_D : Fmt1RI20<0x16000000, (outs GPR:$dst),
1006:                        (ins GPR:$rd, simm20_lu32id:$imm20),
1007:                        "$rd, $imm20">;
1008: }
1009: let isReMaterializable = 1 in {
1010: def LU52I_D : ALU_2RI12<0x03000000, simm12_lu52id>;
1011: }
1012: def PCADDU18I : ALU_1RI20<0x1e000000, simm20_pcaddu18i>;
1013: def MUL_D     : ALU_3R<0x001d8000>;
1014: def MULH_D    : ALU_3R<0x001e0000>;
1015: def MULH_DU   : ALU_3R<0x001e8000>;
1016: def MULW_D_W  : ALU_3R<0x001f0000>;
1017: def MULW_D_WU : ALU_3R<0x001f8000>;
1018: let usesCustomInserter = true in {
1019: def DIV_D     : ALU_3R<0x00220000>;
1020: def MOD_D     : ALU_3R<0x00228000>;
1021: def DIV_DU    : ALU_3R<0x00230000>;
1022: def MOD_DU    : ALU_3R<0x00238000>;
1023: } // usesCustomInserter = true
1024: 
1025: // Bit-shift Instructions for 64-bits
1026: def SLL_D  : ALU_3R<0x00188000>;
1027: def SRL_D  : ALU_3R<0x00190000>;
1028: def SRA_D  : ALU_3R<0x00198000>;
1029: def ROTR_D : ALU_3R<0x001b8000>;
1030: def SLLI_D  : ALU_2RI6<0x00410000, uimm6>;
1031: def SRLI_D  : ALU_2RI6<0x00450000, uimm6>;
1032: def SRAI_D  : ALU_2RI6<0x00490000, uimm6>;
1033: def ROTRI_D : ALU_2RI6<0x004d0000, uimm6>;
1034: 
1035: // Bit-manipulation Instructions for 64-bits
1036: def CLO_D : ALU_2R<0x00002000>;
1037: def CLZ_D : ALU_2R<0x00002400>;
1038: def CTO_D : ALU_2R<0x00002800>;
1039: def CTZ_D : ALU_2R<0x00002c00>;
1040: def BYTEPICK_D : ALU_3RI3<0x000c0000, uimm3>;
```
- **EN**: This block declares or refines TableGen records such as `ALSL_D`, `LU32I_D`, `LU52I_D`, `PCADDU18I`, `MUL_D`, `MULH_D`.
- **CN**: 该代码块声明或细化了 `ALSL_D`, `LU32I_D`, `LU52I_D`, `PCADDU18I`, `MUL_D`, `MULH_D` 等 TableGen 记录。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: def REVB_4H   : ALU_2R<0x00003400>;
1042: def REVB_2W   : ALU_2R<0x00003800>;
1043: def REVB_D    : ALU_2R<0x00003c00>;
1044: def REVH_2W   : ALU_2R<0x00004000>;
1045: def REVH_D    : ALU_2R<0x00004400>;
1046: def BITREV_8B : ALU_2R<0x00004c00>;
1047: def BITREV_D  : ALU_2R<0x00005400>;
1048: let Constraints = "$rd = $dst" in {
1049: def BSTRINS_D  : FmtBSTR_D<0x00800000, (outs GPR:$dst),
1050:                            (ins GPR:$rd, GPR:$rj, uimm6:$msbd, uimm6:$lsbd),
1051:                            "$rd, $rj, $msbd, $lsbd">;
1052: }
1053: def BSTRPICK_D : FmtBSTR_D<0x00c00000, (outs GPR:$rd),
1054:                            (ins GPR:$rj, uimm6:$msbd, uimm6:$lsbd),
1055:                            "$rd, $rj, $msbd, $lsbd">;
1056: 
1057: // Common Memory Access Instructions for 64-bits
1058: def LD_WU : LOAD_2RI12<0x2a800000>;
1059: def LD_D  : LOAD_2RI12<0x28c00000>;
1060: def ST_D : STORE_2RI12<0x29c00000>;
1061: def LDX_B  : LOAD_3R<0x38000000>;
1062: def LDX_H  : LOAD_3R<0x38040000>;
1063: def LDX_W  : LOAD_3R<0x38080000>;
1064: def LDX_D  : LOAD_3R<0x380c0000>;
1065: def LDX_BU : LOAD_3R<0x38200000>;
1066: def LDX_HU : LOAD_3R<0x38240000>;
1067: def LDX_WU : LOAD_3R<0x38280000>;
1068: def STX_B : STORE_3R<0x38100000>;
1069: def STX_H : STORE_3R<0x38140000>;
1070: def STX_W : STORE_3R<0x38180000>;
1071: def STX_D : STORE_3R<0x381c0000>;
1072: def LDPTR_W : LOAD_2RI14<0x24000000>;
1073: def LDPTR_D : LOAD_2RI14<0x26000000>;
1074: def STPTR_W : STORE_2RI14<0x25000000>;
1075: def STPTR_D : STORE_2RI14<0x27000000>;
1076: let hasSideEffects = 0, mayLoad = 1, mayStore = 1 in
1077: def PRELDX : FmtPRELDX<(outs), (ins uimm5:$imm5, GPR:$rj, GPR:$rk),
1078:                        "$imm5, $rj, $rk">;
1079: 
1080: // Bound Check Memory Access Instructions
```
- **EN**: This block declares or refines TableGen records such as `REVB_4H`, `REVB_2W`, `REVB_D`, `REVH_2W`, `REVH_D`, `BITREV_8B`.
- **CN**: 该代码块声明或细化了 `REVB_4H`, `REVB_2W`, `REVB_D`, `REVH_2W`, `REVH_D`, `BITREV_8B` 等 TableGen 记录。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: def LDGT_B : LOAD_3R<0x38780000>;
1082: def LDGT_H : LOAD_3R<0x38788000>;
1083: def LDGT_W : LOAD_3R<0x38790000>;
1084: def LDGT_D : LOAD_3R<0x38798000>;
1085: def LDLE_B : LOAD_3R<0x387a0000>;
1086: def LDLE_H : LOAD_3R<0x387a8000>;
1087: def LDLE_W : LOAD_3R<0x387b0000>;
1088: def LDLE_D : LOAD_3R<0x387b8000>;
1089: def STGT_B : STORE_3R<0x387c0000>;
1090: def STGT_H : STORE_3R<0x387c8000>;
1091: def STGT_W : STORE_3R<0x387d0000>;
1092: def STGT_D : STORE_3R<0x387d8000>;
1093: def STLE_B : STORE_3R<0x387e0000>;
1094: def STLE_H : STORE_3R<0x387e8000>;
1095: def STLE_W : STORE_3R<0x387f0000>;
1096: def STLE_D : STORE_3R<0x387f8000>;
1097: 
1098: // Atomic Memory Access Instructions for 64-bits
1099: def AMSWAP_B     : AM_3R<0x385c0000>;
1100: def AMSWAP_H     : AM_3R<0x385c8000>;
1101: def AMSWAP_W     : AU_3R<0x38600000>;
1102: def AMSWAP_D     : AM_3R<0x38608000>;
1103: def AMADD_B      : AM_3R<0x385d0000>;
1104: def AMADD_H      : AM_3R<0x385d8000>;
1105: def AMADD_W      : AM_3R<0x38610000>;
1106: def AMADD_D      : AM_3R<0x38618000>;
1107: def AMAND_W      : AM_3R<0x38620000>;
1108: def AMAND_D      : AM_3R<0x38628000>;
1109: def AMOR_W       : AM_3R<0x38630000>;
1110: def AMOR_D       : AM_3R<0x38638000>;
1111: def AMXOR_W      : AM_3R<0x38640000>;
1112: def AMXOR_D      : AM_3R<0x38648000>;
1113: def AMMAX_W      : AM_3R<0x38650000>;
1114: def AMMAX_D      : AM_3R<0x38658000>;
1115: def AMMIN_W      : AM_3R<0x38660000>;
1116: def AMMIN_D      : AM_3R<0x38668000>;
1117: def AMMAX_WU     : AM_3R<0x38670000>;
1118: def AMMAX_DU     : AM_3R<0x38678000>;
1119: def AMMIN_WU     : AM_3R<0x38680000>;
1120: def AMMIN_DU     : AM_3R<0x38688000>;
```
- **EN**: This block declares or refines TableGen records such as `LDGT_B`, `LDGT_H`, `LDGT_W`, `LDGT_D`, `LDLE_B`, `LDLE_H`.
- **CN**: 该代码块声明或细化了 `LDGT_B`, `LDGT_H`, `LDGT_W`, `LDGT_D`, `LDLE_B`, `LDLE_H` 等 TableGen 记录。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: def AMSWAP__DB_B : AM_3R<0x385e0000>;
1122: def AMSWAP__DB_H : AM_3R<0x385e8000>;
1123: def AMSWAP__DB_W : AM_3R<0x38690000>;
1124: def AMSWAP__DB_D : AM_3R<0x38698000>;
1125: def AMADD__DB_B  : AM_3R<0x385f0000>;
1126: def AMADD__DB_H  : AM_3R<0x385f8000>;
1127: def AMADD__DB_W  : AM_3R<0x386a0000>;
1128: def AMADD__DB_D  : AM_3R<0x386a8000>;
1129: def AMAND__DB_W  : AM_3R<0x386b0000>;
1130: def AMAND__DB_D  : AM_3R<0x386b8000>;
1131: def AMOR__DB_W   : AM_3R<0x386c0000>;
1132: def AMOR__DB_D   : AM_3R<0x386c8000>;
1133: def AMXOR__DB_W  : AM_3R<0x386d0000>;
1134: def AMXOR__DB_D  : AM_3R<0x386d8000>;
1135: def AMMAX__DB_W  : AM_3R<0x386e0000>;
1136: def AMMAX__DB_D  : AM_3R<0x386e8000>;
1137: def AMMIN__DB_W  : AM_3R<0x386f0000>;
1138: def AMMIN__DB_D  : AM_3R<0x386f8000>;
1139: def AMMAX__DB_WU : AM_3R<0x38700000>;
1140: def AMMAX__DB_DU : AM_3R<0x38708000>;
1141: def AMMIN__DB_WU : AM_3R<0x38710000>;
1142: def AMMIN__DB_DU : AM_3R<0x38718000>;
1143: def AMCAS_B      : AMCAS_3R<0x38580000>;
1144: def AMCAS_H      : AMCAS_3R<0x38588000>;
1145: def AMCAS_W      : AMCAS_3R<0x38590000>;
1146: def AMCAS_D      : AMCAS_3R<0x38598000>;
1147: def AMCAS__DB_B  : AMCAS_3R<0x385a0000>;
1148: def AMCAS__DB_H  : AMCAS_3R<0x385a8000>;
1149: def AMCAS__DB_W  : AMCAS_3R<0x385b0000>;
1150: def AMCAS__DB_D  : AMCAS_3R<0x385b8000>;
1151: def LL_D : LLBase<0x22000000>;
1152: def SC_D : SCBase<0x23000000>;
1153: def SC_Q : SCBase_128<0x38570000>;
1154: def LLACQ_W : LLBase_ACQ<0x38578000>;
1155: def SCREL_W : SCBase_REL<0x38578400>;
1156: def LLACQ_D : LLBase_ACQ<0x38578800>;
1157: def SCREL_D : SCBase_REL<0x38578C00>;
1158: 
1159: // CRC Check Instructions
1160: def CRC_W_B_W  : ALU_3R<0x00240000>;
```
- **EN**: This block declares or refines TableGen records such as `AMSWAP__DB_B`, `AMSWAP__DB_H`, `AMSWAP__DB_W`, `AMSWAP__DB_D`, `AMADD__DB_B`, `AMADD__DB_H`.
- **CN**: 该代码块声明或细化了 `AMSWAP__DB_B`, `AMSWAP__DB_H`, `AMSWAP__DB_W`, `AMSWAP__DB_D`, `AMADD__DB_B`, `AMADD__DB_H` 等 TableGen 记录。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161: def CRC_W_H_W  : ALU_3R<0x00248000>;
1162: def CRC_W_W_W  : ALU_3R<0x00250000>;
1163: def CRC_W_D_W  : ALU_3R<0x00258000>;
1164: def CRCC_W_B_W : ALU_3R<0x00260000>;
1165: def CRCC_W_H_W : ALU_3R<0x00268000>;
1166: def CRCC_W_W_W : ALU_3R<0x00270000>;
1167: def CRCC_W_D_W : ALU_3R<0x00278000>;
1168: 
1169: // Other Miscellaneous Instructions for 64-bits
1170: def ASRTLE_D : FmtASRT<0x00010000, (outs), (ins GPR:$rj, GPR:$rk),
1171:                        "$rj, $rk">;
1172: def ASRTGT_D : FmtASRT<0x00018000, (outs), (ins GPR:$rj, GPR:$rk),
1173:                        "$rj, $rk">;
1174: def RDTIME_D : RDTIME_2R<0x00006800>;
1175: } // Predicates = [IsLA64]
1176: 
1177: //===----------------------------------------------------------------------===//
1178: // Pseudo-instructions and codegen patterns
1179: //
1180: // Naming convention: For 'generic' pattern classes, we use the naming
1181: // convention PatTy1Ty2.
1182: //===----------------------------------------------------------------------===//
1183: 
1184: /// Generic pattern classes
1185: 
1186: def assertsexti32 : PatFrag<(ops node:$src), (assertsext node:$src), [{
1187:   return cast<VTSDNode>(N->getOperand(1))->getVT().bitsLE(MVT::i32);
1188: }]>;
1189: class PatGprGpr<SDPatternOperator OpNode, LAInst Inst>
1190:     : Pat<(OpNode GPR:$rj, GPR:$rk), (Inst GPR:$rj, GPR:$rk)>;
1191: class PatGprGpr_32<SDPatternOperator OpNode, LAInst Inst>
1192:     : Pat<(sext_inreg (OpNode (assertsexti32 GPR:$rj), (assertsexti32 GPR:$rk)), i32), (Inst GPR:$rj, GPR:$rk)>;
1193: class PatGpr<SDPatternOperator OpNode, LAInst Inst>
1194:     : Pat<(OpNode GPR:$rj), (Inst GPR:$rj)>;
1195: 
1196: class PatGprImm<SDPatternOperator OpNode, LAInst Inst, Operand ImmOpnd>
1197:     : Pat<(OpNode GPR:$rj, ImmOpnd:$imm),
1198:           (Inst GPR:$rj, ImmOpnd:$imm)>;
1199: class PatGprImm_32<SDPatternOperator OpNode, LAInst Inst, Operand ImmOpnd>
1200:     : Pat<(sext_inreg (OpNode GPR:$rj, ImmOpnd:$imm), i32),
```
- **EN**: This block declares or refines TableGen records such as `CRC_W_H_W`, `CRC_W_W_W`, `CRC_W_D_W`, `CRCC_W_B_W`, `CRCC_W_H_W`, `CRCC_W_W_W`.
- **CN**: 该代码块声明或细化了 `CRC_W_H_W`, `CRC_W_W_W`, `CRC_W_D_W`, `CRCC_W_B_W`, `CRCC_W_H_W`, `CRCC_W_W_W` 等 TableGen 记录。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201:           (Inst GPR:$rj, ImmOpnd:$imm)>;
1202: 
1203: /// Predicates
1204: def AddLike: PatFrags<(ops node:$A, node:$B),
1205:                       [(add node:$A, node:$B), (or node:$A, node:$B)], [{
1206:     return CurDAG->isBaseWithConstantOffset(SDValue(N, 0));
1207: }]>;
1208: 
1209: /// Simple arithmetic operations
1210: 
1211: // Match both a plain shift and one where the shift amount is masked (this is
1212: // typically introduced when the legalizer promotes the shift amount and
1213: // zero-extends it). For LoongArch, the mask is unnecessary as shifts in the
1214: // base ISA only read the least significant 5 bits (LA32) or 6 bits (LA64).
1215: def shiftMaskGRLen
1216:     : ComplexPattern<GRLenVT, 1, "selectShiftMaskGRLen", [], [], 0>;
1217: def shiftMask32 : ComplexPattern<i64, 1, "selectShiftMask32", [], [], 0>;
1218: 
1219: def sexti32 : ComplexPattern<i64, 1, "selectSExti32">;
1220: def zexti32 : ComplexPattern<i64, 1, "selectZExti32">;
1221: 
1222: class shiftop<SDPatternOperator operator>
1223:     : PatFrag<(ops node:$val, node:$count),
1224:               (operator node:$val, (GRLenVT (shiftMaskGRLen node:$count)))>;
1225: class shiftopw<SDPatternOperator operator>
1226:     : PatFrag<(ops node:$val, node:$count),
1227:               (operator node:$val, (i64 (shiftMask32 node:$count)))>;
1228: 
1229: def mul_const_oneuse : PatFrag<(ops node:$A, node:$B),
1230:                                (mul node:$A, node:$B), [{
1231:   if (auto *N1C = dyn_cast<ConstantSDNode>(N->getOperand(1)))
1232:     return N1C->hasOneUse();
1233:   return false;
1234: }]>;
1235: 
1236: let Predicates = [IsLA32] in {
1237: def : PatGprGpr<add, ADD_W>;
1238: def : PatGprImm<add, ADDI_W, simm12>;
1239: def : PatGprGpr<sub, SUB_W>;
1240: def : PatGprGpr<sdiv, DIV_W>;
```
- **EN**: This block declares or refines TableGen records such as `AddLike:`, `shiftMaskGRLen`, `shiftMask32`, `sexti32`, `zexti32`, `shiftop`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `AddLike:`, `shiftMaskGRLen`, `shiftMask32`, `sexti32`, `zexti32`, `shiftop` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241: def : PatGprGpr<udiv, DIV_WU>;
1242: def : PatGprGpr<srem, MOD_W>;
1243: def : PatGprGpr<urem, MOD_WU>;
1244: def : PatGprGpr<mul, MUL_W>;
1245: def : PatGprGpr<mulhs, MULH_W>;
1246: def : PatGprGpr<mulhu, MULH_WU>;
1247: } // Predicates = [IsLA32]
1248: 
1249: let Predicates = [IsLA32, Has32S] in {
1250: def : PatGprGpr<shiftop<rotr>, ROTR_W>;
1251: def : PatGprImm<rotr, ROTRI_W, uimm5>;
1252: 
1253: foreach Idx = 1...3 in {
1254:   defvar ShamtA = !mul(8, Idx);
1255:   defvar ShamtB = !mul(8, !sub(4, Idx));
1256:   def : Pat<(or (shl GPR:$rk, (i32 ShamtA)), (srl GPR:$rj, (i32 ShamtB))),
1257:             (BYTEPICK_W GPR:$rj, GPR:$rk, Idx)>;
1258: }
1259: } // Predicates = [IsLA32, Has32S]
1260: 
1261: let Predicates = [IsLA64] in {
1262: def : PatGprGpr<add, ADD_D>;
1263: def : PatGprImm<add, ADDI_D, simm12>;
1264: def : PatGprGpr<sub, SUB_D>;
1265: def : PatGprGpr<sdiv, DIV_D>;
1266: def : PatGprGpr_32<sdiv, DIV_W>;
1267: def : PatGprGpr<udiv, DIV_DU>;
1268: def : PatGprGpr<loongarch_div_w, DIV_W>;
1269: def : PatGprGpr<loongarch_div_wu, DIV_WU>;
1270: def : PatGprGpr<srem, MOD_D>;
1271: def : PatGprGpr_32<srem, MOD_W>;
1272: def : PatGprGpr<urem, MOD_DU>;
1273: def : PatGprGpr<loongarch_mod_w, MOD_W>;
1274: def : PatGprGpr<loongarch_mod_wu, MOD_WU>;
1275: def : PatGprGpr<shiftop<rotr>, ROTR_D>;
1276: def : PatGprGpr<shiftopw<loongarch_rotr_w>, ROTR_W>;
1277: def : PatGprImm<rotr, ROTRI_D, uimm6>;
1278: def : PatGprImm_32<rotr, ROTRI_W, uimm5>;
1279: def : PatGprImm<loongarch_rotr_w, ROTRI_W, uimm5>;
1280: // TODO: Select "_W[U]" instructions for i32xi32 if only lower 32 bits of the
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281: // product are used.
1282: def : PatGprGpr<mul, MUL_D>;
1283: def : PatGprGpr<mulhs, MULH_D>;
1284: def : PatGprGpr<mulhu, MULH_DU>;
1285: // Select MULW_D_W for calculating the full 64 bits product of i32xi32 signed
1286: // multiplication.
1287: def : Pat<(i64 (mul (sext_inreg GPR:$rj, i32), (sext_inreg GPR:$rk, i32))),
1288:           (MULW_D_W GPR:$rj, GPR:$rk)>;
1289: // Select MULW_D_WU for calculating the full 64 bits product of i32xi32
1290: // unsigned multiplication.
1291: def : Pat<(i64 (mul (loongarch_bstrpick GPR:$rj, (i64 31), (i64 0)),
1292:                     (loongarch_bstrpick GPR:$rk, (i64 31), (i64 0)))),
1293:           (MULW_D_WU GPR:$rj, GPR:$rk)>;
1294: 
1295: def : Pat<(add GPR:$rj, simm16_lsl16:$imm),
1296:           (ADDU16I_D GPR:$rj, (HI16 $imm))>;
1297: def : Pat<(add GPR:$rj, simm32_hi16_lo12:$imm),
1298:           (ADDI_D (ADDU16I_D GPR:$rj, (HI16ForAddu16idAddiPair $imm)),
1299:                   (LO12 $imm))>;
1300: def : Pat<(sext_inreg (add GPR:$rj, simm32_hi16_lo12:$imm), i32),
1301:           (ADDI_W (ADDU16I_D GPR:$rj, (HI16ForAddu16idAddiPair $imm)),
1302:                   (LO12 $imm))>;
1303: 
1304: let Predicates = [IsLA32] in {
1305: def : Pat<(add GPR:$rj, (AddiPair:$im)),
1306:           (ADDI_W (ADDI_W GPR:$rj, (AddiPairImmLarge AddiPair:$im)),
1307:                   (AddiPairImmSmall AddiPair:$im))>;
1308: } // Predicates = [IsLA32]
1309: 
1310: let Predicates = [IsLA64] in {
1311: def : Pat<(add GPR:$rj, (AddiPair:$im)),
1312:           (ADDI_D (ADDI_D GPR:$rj, (AddiPairImmLarge AddiPair:$im)),
1313:                   (AddiPairImmSmall AddiPair:$im))>;
1314: def : Pat<(sext_inreg (add GPR:$rj, (AddiPair:$im)), i32),
1315:           (ADDI_W (ADDI_W GPR:$rj, (AddiPairImmLarge AddiPair:$im)),
1316:                   (AddiPairImmSmall AddiPair:$im))>;
1317: } // Predicates = [IsLA64]
1318: 
1319: let Predicates = [IsLA32, Has32S] in {
1320: foreach Idx0 = 1...4 in {
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1321-1360 / 第 1321-1360 行
```tablegen
1321:   foreach Idx1 = 1...4 in {
1322:     defvar CImm = !add(1, !shl(!add(1, !shl(1, Idx0)), Idx1));
1323:     def : Pat<(mul_const_oneuse GPR:$r, (i32 CImm)),
1324:               (ALSL_W (ALSL_W GPR:$r, GPR:$r, (i32 Idx0)),
1325:                       GPR:$r, (i32 Idx1))>;
1326:   }
1327: }
1328: foreach Idx0 = 1...4 in {
1329:   foreach Idx1 = 1...4 in {
1330:     defvar Cb = !add(1, !shl(1, Idx0));
1331:     defvar CImm = !add(Cb, !shl(Cb, Idx1));
1332:     def : Pat<(mul_const_oneuse GPR:$r, (i32 CImm)),
1333:               (ALSL_W (ALSL_W GPR:$r, GPR:$r, (i32 Idx0)),
1334:                       (ALSL_W GPR:$r, GPR:$r, (i32 Idx0)), (i32 Idx1))>;
1335:   }
1336: }
1337: } // Predicates = [IsLA32, Has32S]
1338: 
1339: let Predicates = [IsLA64] in {
1340: foreach Idx0 = 1...4 in {
1341:   foreach Idx1 = 1...4 in {
1342:     defvar CImm = !add(1, !shl(!add(1, !shl(1, Idx0)), Idx1));
1343:     def : Pat<(sext_inreg (mul_const_oneuse GPR:$r, (i64 CImm)), i32),
1344:               (ALSL_W (ALSL_W GPR:$r, GPR:$r, (i64 Idx0)),
1345:                       GPR:$r, (i64 Idx1))>;
1346:     def : Pat<(mul_const_oneuse GPR:$r, (i64 CImm)),
1347:               (ALSL_D (ALSL_D GPR:$r, GPR:$r, (i64 Idx0)),
1348:                       GPR:$r, (i64 Idx1))>;
1349:   }
1350: }
1351: foreach Idx0 = 1...4 in {
1352:   foreach Idx1 = 1...4 in {
1353:     defvar Cb = !add(1, !shl(1, Idx0));
1354:     defvar CImm = !add(Cb, !shl(Cb, Idx1));
1355:     def : Pat<(sext_inreg (mul_const_oneuse GPR:$r, (i64 CImm)), i32),
1356:               (ALSL_W (ALSL_W GPR:$r, GPR:$r, (i64 Idx0)),
1357:                       (ALSL_W GPR:$r, GPR:$r, (i64 Idx0)), (i64 Idx1))>;
1358:     def : Pat<(mul_const_oneuse GPR:$r, (i64 CImm)),
1359:               (ALSL_D (ALSL_D GPR:$r, GPR:$r, (i64 Idx0)),
1360:                       (ALSL_D GPR:$r, GPR:$r, (i64 Idx0)), (i64 Idx1))>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1361-1400 / 第 1361-1400 行
```tablegen
1361:   }
1362: }
1363: } // Predicates = [IsLA64]
1364: 
1365: let Predicates = [IsLA32, Has32S] in {
1366: def : Pat<(mul GPR:$rj, (AlslSlliImm:$im)),
1367:           (SLLI_W (ALSL_W GPR:$rj, GPR:$rj, (AlslSlliImmI0 AlslSlliImm:$im)),
1368:                   (AlslSlliImmI1 AlslSlliImm:$im))>;
1369: } // Predicates = [IsLA32, Has32S]
1370: 
1371: let Predicates = [IsLA64] in {
1372: def : Pat<(sext_inreg (mul GPR:$rj, (AlslSlliImm:$im)), i32),
1373:           (SLLI_W (ALSL_W GPR:$rj, GPR:$rj, (AlslSlliImmI0 AlslSlliImm:$im)),
1374:                   (AlslSlliImmI1 AlslSlliImm:$im))>;
1375: def : Pat<(mul GPR:$rj, (AlslSlliImm:$im)),
1376:           (SLLI_D (ALSL_D GPR:$rj, GPR:$rj, (AlslSlliImmI0 AlslSlliImm:$im)),
1377:                   (AlslSlliImmI1 AlslSlliImm:$im))>;
1378: } // Predicates = [IsLA64]
1379: 
1380: foreach Idx = 1...7 in {
1381:   defvar ShamtA = !mul(8, Idx);
1382:   defvar ShamtB = !mul(8, !sub(8, Idx));
1383:   def : Pat<(or (shl GPR:$rk, (i64 ShamtA)), (srl GPR:$rj, (i64 ShamtB))),
1384:             (BYTEPICK_D GPR:$rj, GPR:$rk, Idx)>;
1385: }
1386: 
1387: foreach Idx = 1...3 in {
1388:   defvar ShamtA = !mul(8, Idx);
1389:   defvar ShamtB = !mul(8, !sub(4, Idx));
1390:   // NOTE: the srl node would already be transformed into a loongarch_bstrpick
1391:   // by the time this pattern gets to execute, hence the weird construction.
1392:   def : Pat<(sext_inreg (or (shl GPR:$rk, (i64 ShamtA)),
1393:                             (loongarch_bstrpick GPR:$rj, (i64 31),
1394:                                                          (i64 ShamtB))), i32),
1395:             (BYTEPICK_W GPR:$rj, GPR:$rk, Idx)>;
1396: }
1397: } // Predicates = [IsLA64]
1398: 
1399: def : PatGprGpr<and, AND>;
1400: def : PatGprImm<and, ANDI, uimm12>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1401-1440 / 第 1401-1440 行
```tablegen
1401: def : PatGprGpr<or, OR>;
1402: def : PatGprImm<or, ORI, uimm12>;
1403: def : PatGprGpr<xor, XOR>;
1404: def : PatGprImm<xor, XORI, uimm12>;
1405: def : Pat<(not GPR:$rj), (NOR GPR:$rj, R0)>;
1406: def : Pat<(not (or GPR:$rj, GPR:$rk)), (NOR GPR:$rj, GPR:$rk)>;
1407: def : Pat<(or GPR:$rj, (not GPR:$rk)), (ORN GPR:$rj, GPR:$rk)>;
1408: def : Pat<(and GPR:$rj, (not GPR:$rk)), (ANDN GPR:$rj, GPR:$rk)>;
1409: 
1410: let Predicates = [IsLA32, Has32S] in {
1411: def : Pat<(and GPR:$rj, BstrinsImm:$imm),
1412:           (BSTRINS_W GPR:$rj, R0, (BstrinsMsb BstrinsImm:$imm),
1413:                      (BstrinsLsb BstrinsImm:$imm))>;
1414: } // Predicates = [IsLA32, Has32S]
1415: 
1416: let Predicates = [IsLA64] in {
1417: def : Pat<(and GPR:$rj, BstrinsImm:$imm),
1418:           (BSTRINS_D GPR:$rj, R0, (BstrinsMsb BstrinsImm:$imm),
1419:                      (BstrinsLsb BstrinsImm:$imm))>;
1420: } // Predicates = [IsLA64]
1421: 
1422: /// Traps
1423: 
1424: // We lower `trap` to `ud 0`, which is an alias for `amswap.w $r0, $r1, $r0`.
1425: def PseudoUNIMP : Pseudo<(outs), (ins), [(trap)]>, PseudoInstExpansion<(UD 0)>;
1426: 
1427: // We lower `debugtrap` to `break 0`, as this is guaranteed to exist and work,
1428: // even for LA32 Primary. Also, because so far the ISA does not provide a
1429: // specific trap instruction/kind exclusively for alerting the debugger,
1430: // every other project uses the generic immediate of 0 for this.
1431: def : Pat<(debugtrap), (BREAK 0)>;
1432: 
1433: /// Bit counting operations
1434: 
1435: let Predicates = [IsLA64] in {
1436: def : PatGpr<ctlz, CLZ_D>;
1437: def : PatGpr<cttz, CTZ_D>;
1438: def : Pat<(ctlz (not GPR:$rj)), (CLO_D GPR:$rj)>;
1439: def : Pat<(cttz (not GPR:$rj)), (CTO_D GPR:$rj)>;
1440: def : Pat<(cttz (or (not GPR:$rj), 256)), (CTO_D (ANDI GPR:$rj, 255))>;
```
- **EN**: This block declares or refines TableGen records such as `PseudoUNIMP`.
- **CN**: 该代码块声明或细化了 `PseudoUNIMP` 等 TableGen 记录。

### Lines 1441-1480 / 第 1441-1480 行
```tablegen
1441: def : Pat<(cttz (or (not GPR:$rj), 65536)), (CTO_D (BSTRPICK_D GPR:$rj, 15, 0))>;
1442: def : PatGpr<loongarch_clzw, CLZ_W>;
1443: def : PatGpr<loongarch_ctzw, CTZ_W>;
1444: def : Pat<(loongarch_clzw (not GPR:$rj)), (CLO_W GPR:$rj)>;
1445: def : Pat<(loongarch_ctzw (not GPR:$rj)), (CTO_W GPR:$rj)>;
1446: } // Predicates = [IsLA64]
1447: 
1448: let Predicates = [IsLA32, Has32S] in {
1449: def : PatGpr<ctlz, CLZ_W>;
1450: def : PatGpr<cttz, CTZ_W>;
1451: def : Pat<(ctlz (not GPR:$rj)), (CLO_W GPR:$rj)>;
1452: def : Pat<(cttz (not GPR:$rj)), (CTO_W GPR:$rj)>;
1453: def : Pat<(cttz (or (not GPR:$rj), 256)), (CTO_W (ANDI GPR:$rj, 255))>;
1454: def : Pat<(cttz (or (not GPR:$rj), 65536)), (CTO_W (BSTRPICK_W GPR:$rj, 15, 0))>;
1455: } // Predicates = [IsLA32, Has32S]
1456: 
1457: /// FrameIndex calculations
1458: let Predicates = [IsLA32] in {
1459: def : Pat<(AddLike (i32 BaseAddr:$rj), simm12:$imm12),
1460:           (ADDI_W (i32 BaseAddr:$rj), simm12:$imm12)>;
1461: } // Predicates = [IsLA32]
1462: let Predicates = [IsLA64] in {
1463: def : Pat<(AddLike (i64 BaseAddr:$rj), simm12:$imm12),
1464:           (ADDI_D (i64 BaseAddr:$rj), simm12:$imm12)>;
1465: } // Predicates = [IsLA64]
1466: 
1467: /// Shifted addition
1468: let Predicates = [IsLA32, Has32S] in {
1469: def : Pat<(add GPR:$rk, (shl GPR:$rj, uimm2_plus1:$imm2)),
1470:           (ALSL_W GPR:$rj, GPR:$rk, uimm2_plus1:$imm2)>;
1471: } // Predicates = [IsLA32, Has32S]
1472: let Predicates = [IsLA64] in {
1473: def : Pat<(add GPR:$rk, (shl GPR:$rj, uimm2_plus1:$imm2)),
1474:           (ALSL_D GPR:$rj, GPR:$rk, uimm2_plus1:$imm2)>;
1475: def : Pat<(sext_inreg (add GPR:$rk, (shl GPR:$rj, uimm2_plus1:$imm2)), i32),
1476:           (ALSL_W GPR:$rj, GPR:$rk, uimm2_plus1:$imm2)>;
1477: def : Pat<(loongarch_bstrpick (add GPR:$rk, (shl GPR:$rj, uimm2_plus1:$imm2)),
1478:                               (i64 31), (i64 0)),
1479:           (ALSL_WU GPR:$rj, GPR:$rk, uimm2_plus1:$imm2)>;
1480: } // Predicates = [IsLA64]
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1481-1520 / 第 1481-1520 行
```tablegen
1481: 
1482: /// Shift
1483: 
1484: let Predicates = [IsLA32] in {
1485: def : PatGprGpr<shiftop<shl>, SLL_W>;
1486: def : PatGprGpr<shiftop<sra>, SRA_W>;
1487: def : PatGprGpr<shiftop<srl>, SRL_W>;
1488: def : PatGprImm<shl, SLLI_W, uimm5>;
1489: def : PatGprImm<sra, SRAI_W, uimm5>;
1490: def : PatGprImm<srl, SRLI_W, uimm5>;
1491: } // Predicates = [IsLA32]
1492: 
1493: let Predicates = [IsLA64] in {
1494: def : PatGprGpr<shiftopw<loongarch_sll_w>, SLL_W>;
1495: def : PatGprGpr<shiftopw<loongarch_sra_w>, SRA_W>;
1496: def : PatGprGpr<shiftopw<loongarch_srl_w>, SRL_W>;
1497: def : PatGprGpr<shiftop<shl>, SLL_D>;
1498: def : PatGprGpr<shiftop<sra>, SRA_D>;
1499: def : PatGprGpr<shiftop<srl>, SRL_D>;
1500: def : PatGprImm<shl, SLLI_D, uimm6>;
1501: def : PatGprImm<sra, SRAI_D, uimm6>;
1502: def : PatGprImm<srl, SRLI_D, uimm6>;
1503: } // Predicates = [IsLA64]
1504: 
1505: /// sext and zext
1506: 
1507: let Predicates = [Has32S] in {
1508: def : Pat<(sext_inreg GPR:$rj, i8), (EXT_W_B GPR:$rj)>;
1509: def : Pat<(sext_inreg GPR:$rj, i16), (EXT_W_H GPR:$rj)>;
1510: } // Predicates = [Has32S]
1511: 
1512: let Predicates = [IsLA64] in {
1513: def : Pat<(sext_inreg GPR:$rj, i32), (ADDI_W GPR:$rj, 0)>;
1514: } // Predicates = [IsLA64]
1515: 
1516: /// Setcc
1517: 
1518: def : PatGprGpr<setlt, SLT>;
1519: def : PatGprImm<setlt, SLTI, simm12>;
1520: def : PatGprGpr<setult, SLTU>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1521-1560 / 第 1521-1560 行
```tablegen
1521: def : PatGprImm<setult, SLTUI, simm12>;
1522: 
1523: // Define pattern expansions for setcc operations that aren't directly
1524: // handled by a LoongArch instruction.
1525: def : Pat<(seteq GPR:$rj, 0), (SLTUI GPR:$rj, 1)>;
1526: def : Pat<(seteq GPR:$rj, GPR:$rk), (SLTUI (XOR GPR:$rj, GPR:$rk), 1)>;
1527: let Predicates = [IsLA32] in {
1528: def : Pat<(seteq GPR:$rj, simm12_plus1:$imm12),
1529:           (SLTUI (ADDI_W GPR:$rj, (NegImm simm12_plus1:$imm12)), 1)>;
1530: } // Predicates = [IsLA32]
1531: let Predicates = [IsLA64] in {
1532: def : Pat<(seteq GPR:$rj, simm12_plus1:$imm12),
1533:           (SLTUI (ADDI_D GPR:$rj, (NegImm simm12_plus1:$imm12)), 1)>;
1534: } // Predicates = [IsLA64]
1535: def : Pat<(setne GPR:$rj, 0), (SLTU R0, GPR:$rj)>;
1536: def : Pat<(setne GPR:$rj, GPR:$rk), (SLTU R0, (XOR GPR:$rj, GPR:$rk))>;
1537: let Predicates = [IsLA32] in {
1538: def : Pat<(setne GPR:$rj, simm12_plus1:$imm12),
1539:           (SLTU R0, (ADDI_W GPR:$rj, (NegImm simm12_plus1:$imm12)))>;
1540: } // Predicates = [IsLA32]
1541: let Predicates = [IsLA64] in {
1542: def : Pat<(setne GPR:$rj, simm12_plus1:$imm12),
1543:           (SLTU R0, (ADDI_D GPR:$rj, (NegImm simm12_plus1:$imm12)))>;
1544: } // Predicates = [IsLA64]
1545: def : Pat<(setugt GPR:$rj, GPR:$rk), (SLTU GPR:$rk, GPR:$rj)>;
1546: def : Pat<(setuge GPR:$rj, GPR:$rk), (XORI (SLTU GPR:$rj, GPR:$rk), 1)>;
1547: def : Pat<(setule GPR:$rj, GPR:$rk), (XORI (SLTU GPR:$rk, GPR:$rj), 1)>;
1548: def : Pat<(setgt GPR:$rj, GPR:$rk), (SLT GPR:$rk, GPR:$rj)>;
1549: def : Pat<(setge GPR:$rj, GPR:$rk), (XORI (SLT GPR:$rj, GPR:$rk), 1)>;
1550: def : Pat<(setle GPR:$rj, GPR:$rk), (XORI (SLT GPR:$rk, GPR:$rj), 1)>;
1551: 
1552: /// Select
1553: 
1554: def IntCCtoBranchOpc : SDNodeXForm<loongarch_selectcc, [{
1555:   ISD::CondCode CC = cast<CondCodeSDNode>(N->getOperand(2))->get();
1556:   unsigned BrCC = getBranchOpcForIntCC(CC);
1557:   return CurDAG->getTargetConstant(BrCC, SDLoc(N), Subtarget->getGRLenVT());
1558: }]>;
1559: 
1560: def loongarch_selectcc_frag : PatFrag<(ops node:$lhs, node:$rhs, node:$cc,
```
- **EN**: This block declares or refines TableGen records such as `IntCCtoBranchOpc`, `loongarch_selectcc_frag`.
- **CN**: 该代码块声明或细化了 `IntCCtoBranchOpc`, `loongarch_selectcc_frag` 等 TableGen 记录。

### Lines 1561-1600 / 第 1561-1600 行
```tablegen
1561:                                            node:$truev, node:$falsev),
1562:                                       (loongarch_selectcc node:$lhs, node:$rhs,
1563:                                                           node:$cc, node:$truev,
1564:                                                           node:$falsev), [{}],
1565:                                       IntCCtoBranchOpc>;
1566: 
1567: multiclass SelectCC_GPR_rrirr<DAGOperand valty, ValueType vt> {
1568:   let usesCustomInserter = 1 in
1569:   def _Using_CC_GPR : Pseudo<(outs valty:$dst),
1570:                              (ins GPR:$lhs, GPR:$rhs, GPR:$cc,
1571:                               valty:$truev, valty:$falsev),
1572:                              [(set valty:$dst,
1573:                                (loongarch_selectcc_frag:$cc (GRLenVT GPR:$lhs), GPR:$rhs, cond,
1574:                                                             (vt valty:$truev), valty:$falsev))]>;
1575:   // Explicitly select 0 in the condition to R0. The register coalescer doesn't
1576:   // always do it.
1577:   def : Pat<(loongarch_selectcc_frag:$cc (GRLenVT GPR:$lhs), 0, cond, (vt valty:$truev),
1578:                                          valty:$falsev),
1579:             (!cast<Instruction>(NAME#"_Using_CC_GPR") GPR:$lhs, (GRLenVT R0),
1580:              (IntCCtoBranchOpc $cc), valty:$truev, valty:$falsev)>;
1581: }
1582: 
1583: defm Select_GPR : SelectCC_GPR_rrirr<GPR, GRLenVT>;
1584: 
1585: let Predicates = [Has32S] in {
1586: def : Pat<(select GPR:$cond, GPR:$t, 0), (MASKEQZ GPR:$t, GPR:$cond)>;
1587: def : Pat<(select GPR:$cond, 0, GPR:$f), (MASKNEZ GPR:$f, GPR:$cond)>;
1588: def : Pat<(select GPR:$cond, GPR:$t, GPR:$f),
1589:           (OR (MASKEQZ GPR:$t, GPR:$cond), (MASKNEZ GPR:$f, GPR:$cond))>;
1590: } // Predicates = [Has32S]
1591: 
1592: /// Branches and jumps
1593: 
1594: let Predicates = [Has32S] in {
1595: class BccZeroPat<CondCode Cond, LAInst Inst>
1596:     : Pat<(loongarch_brcc (GRLenVT GPR:$rj), 0, Cond, bb:$imm21),
1597:            (Inst GPR:$rj, bb:$imm21)>;
1598: 
1599: def : BccZeroPat<SETEQ, BEQZ>;
1600: def : BccZeroPat<SETNE, BNEZ>;
```
- **EN**: This block declares or refines TableGen records such as `SelectCC_GPR_rrirr`, `_Using_CC_GPR`, `Select_GPR`, `BccZeroPat`.
- **CN**: 该代码块声明或细化了 `SelectCC_GPR_rrirr`, `_Using_CC_GPR`, `Select_GPR`, `BccZeroPat` 等 TableGen 记录。

### Lines 1601-1640 / 第 1601-1640 行
```tablegen
1601: } // Predicates = [Has32S]
1602: 
1603: multiclass BccPat<CondCode Cond, LAInst Inst> {
1604:   def : Pat<(loongarch_brcc (GRLenVT GPR:$rj), GPR:$rd, Cond, bb:$imm16),
1605:             (Inst GPR:$rj, GPR:$rd, bb:$imm16)>;
1606:   // Explicitly select 0 to R0. The register coalescer doesn't always do it.
1607:   def : Pat<(loongarch_brcc (GRLenVT GPR:$rj), 0, Cond, bb:$imm16),
1608:             (Inst GPR:$rj, (GRLenVT R0), bb:$imm16)>;
1609: }
1610: 
1611: defm : BccPat<SETEQ, BEQ>;
1612: defm : BccPat<SETNE, BNE>;
1613: defm : BccPat<SETLT, BLT>;
1614: defm : BccPat<SETGE, BGE>;
1615: defm : BccPat<SETULT, BLTU>;
1616: defm : BccPat<SETUGE, BGEU>;
1617: 
1618: let isBarrier = 1, isBranch = 1, isTerminator = 1 in
1619: def PseudoBR : Pseudo<(outs), (ins simm26_b:$imm26), [(br bb:$imm26)]>,
1620:                PseudoInstExpansion<(B simm26_b:$imm26)>;
1621: 
1622: let isBarrier = 1, isBranch = 1, isIndirectBranch = 1, isTerminator = 1 in
1623: def PseudoBRIND : Pseudo<(outs), (ins GPRJR:$rj, simm16_lsl2:$imm16)>,
1624:                   PseudoInstExpansion<(JIRL R0, GPR:$rj, simm16_lsl2:$imm16)>;
1625: 
1626: def : Pat<(brind GPRJR:$rj), (PseudoBRIND GPRJR:$rj, 0)>;
1627: def : Pat<(brind (add GPRJR:$rj, simm16_lsl2:$imm16)),
1628:           (PseudoBRIND GPRJR:$rj, simm16_lsl2:$imm16)>;
1629: 
1630: // Function call with 'Small' code model.
1631: let isCall = 1, Defs = [R1] in
1632: def PseudoCALL_SMALL : Pseudo<(outs), (ins bare_symbol:$func)>;
1633: 
1634: def : Pat<(loongarch_call tglobaladdr:$func),
1635:           (PseudoCALL_SMALL tglobaladdr:$func)>;
1636: def : Pat<(loongarch_call texternalsym:$func),
1637:           (PseudoCALL_SMALL texternalsym:$func)>;
1638: 
1639: // Function call with 'Medium' code model.
1640: let isCall = 1, Defs = [R1, R20], Size = 8 in
```
- **EN**: This block declares or refines TableGen records such as `BccPat`, `PseudoBR`, `PseudoBRIND`, `PseudoCALL_SMALL`.
- **CN**: 该代码块声明或细化了 `BccPat`, `PseudoBR`, `PseudoBRIND`, `PseudoCALL_SMALL` 等 TableGen 记录。

### Lines 1641-1680 / 第 1641-1680 行
```tablegen
1641: def PseudoCALL_MEDIUM : Pseudo<(outs), (ins bare_symbol:$func)>;
1642: 
1643: def : Pat<(loongarch_call_medium tglobaladdr:$func),
1644:           (PseudoCALL_MEDIUM tglobaladdr:$func)>;
1645: def : Pat<(loongarch_call_medium texternalsym:$func),
1646:           (PseudoCALL_MEDIUM texternalsym:$func)>;
1647: 
1648: // Function call with 'Large' code model.
1649: let isCall = 1, Defs = [R1] in
1650: def PseudoCALL_LARGE: Pseudo<(outs), (ins bare_symbol:$func)>;
1651: 
1652: let Predicates = [IsLA64] in {
1653: def : Pat<(loongarch_call_large tglobaladdr:$func),
1654:           (PseudoCALL_LARGE tglobaladdr:$func)>;
1655: def : Pat<(loongarch_call_large texternalsym:$func),
1656:           (PseudoCALL_LARGE texternalsym:$func)>;
1657: } // Predicates = [IsLA64]
1658: 
1659: let isCall = 1, Defs = [R1] in
1660: def PseudoCALLIndirect : Pseudo<(outs), (ins GPR:$rj),
1661:                                 [(loongarch_call GPR:$rj)]>,
1662:                          PseudoInstExpansion<(JIRL R1, GPR:$rj, 0)>;
1663: def : Pat<(loongarch_call_medium GPR:$rj), (PseudoCALLIndirect GPR:$rj)>;
1664: let Predicates = [IsLA64] in
1665: def : Pat<(loongarch_call_large GPR:$rj), (PseudoCALLIndirect GPR:$rj)>;
1666: 
1667: let isCall = 1, hasSideEffects = 0, mayStore = 0, mayLoad = 0, Defs = [R1] in
1668: def PseudoJIRL_CALL : Pseudo<(outs), (ins GPR:$rj, simm16_lsl2:$imm16)>,
1669:                       PseudoInstExpansion<(JIRL R1, GPR:$rj,
1670:                                            simm16_lsl2:$imm16)>;
1671: 
1672: let isBarrier = 1, isReturn = 1, isTerminator = 1 in
1673: def PseudoRET : Pseudo<(outs), (ins), [(loongarch_ret)]>,
1674:                 PseudoInstExpansion<(JIRL R0, R1, 0)>;
1675: 
1676: // Tail call with 'Small' code model.
1677: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [R3] in
1678: def PseudoTAIL_SMALL : Pseudo<(outs), (ins bare_symbol:$dst)>;
1679: 
1680: def : Pat<(loongarch_tail (iPTR tglobaladdr:$dst)),
```
- **EN**: This block declares or refines TableGen records such as `PseudoCALL_MEDIUM`, `PseudoCALL_LARGE:`, `PseudoCALLIndirect`, `PseudoJIRL_CALL`, `PseudoRET`, `PseudoTAIL_SMALL`.
- **CN**: 该代码块声明或细化了 `PseudoCALL_MEDIUM`, `PseudoCALL_LARGE:`, `PseudoCALLIndirect`, `PseudoJIRL_CALL`, `PseudoRET`, `PseudoTAIL_SMALL` 等 TableGen 记录。

### Lines 1681-1720 / 第 1681-1720 行
```tablegen
1681:           (PseudoTAIL_SMALL tglobaladdr:$dst)>;
1682: def : Pat<(loongarch_tail (iPTR texternalsym:$dst)),
1683:           (PseudoTAIL_SMALL texternalsym:$dst)>;
1684: 
1685: // Tail call with 'Medium' code model.
1686: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1,
1687:     Uses = [R3], Defs = [R20], Size = 8 in
1688: def PseudoTAIL_MEDIUM : Pseudo<(outs), (ins bare_symbol:$dst)>;
1689: 
1690: def : Pat<(loongarch_tail_medium (iPTR tglobaladdr:$dst)),
1691:           (PseudoTAIL_MEDIUM tglobaladdr:$dst)>;
1692: def : Pat<(loongarch_tail_medium (iPTR texternalsym:$dst)),
1693:           (PseudoTAIL_MEDIUM texternalsym:$dst)>;
1694: 
1695: // Tail call with 'Large' code model.
1696: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [R3] in
1697: def PseudoTAIL_LARGE : Pseudo<(outs), (ins bare_symbol:$dst)>;
1698: 
1699: let Predicates = [IsLA64] in {
1700: def : Pat<(loongarch_tail_large (iPTR tglobaladdr:$dst)),
1701:           (PseudoTAIL_LARGE tglobaladdr:$dst)>;
1702: def : Pat<(loongarch_tail_large (iPTR texternalsym:$dst)),
1703:           (PseudoTAIL_LARGE texternalsym:$dst)>;
1704: } // Predicates = [IsLA64]
1705: 
1706: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [R3] in
1707: def PseudoTAILIndirect : Pseudo<(outs), (ins GPRT:$rj),
1708:                                 [(loongarch_tail GPRT:$rj)]>,
1709:                          PseudoInstExpansion<(JIRL R0, GPR:$rj, 0)>;
1710: def : Pat<(loongarch_tail_medium GPR:$rj), (PseudoTAILIndirect GPR:$rj)>;
1711: let Predicates = [IsLA64] in
1712: def : Pat<(loongarch_tail_large GPR:$rj), (PseudoTAILIndirect GPR:$rj)>;
1713: 
1714: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1,
1715:     hasSideEffects = 0, mayStore = 0, mayLoad = 0, Uses = [R3] in
1716: def PseudoB_TAIL : Pseudo<(outs), (ins simm26_b:$imm26)>,
1717:                    PseudoInstExpansion<(B simm26_b:$imm26)>;
1718: 
1719: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1,
1720:     hasSideEffects = 0, mayStore = 0, mayLoad = 0, Uses = [R3] in
```
- **EN**: This block declares or refines TableGen records such as `PseudoTAIL_MEDIUM`, `PseudoTAIL_LARGE`, `PseudoTAILIndirect`, `PseudoB_TAIL`.
- **CN**: 该代码块声明或细化了 `PseudoTAIL_MEDIUM`, `PseudoTAIL_LARGE`, `PseudoTAILIndirect`, `PseudoB_TAIL` 等 TableGen 记录。

### Lines 1721-1760 / 第 1721-1760 行
```tablegen
1721: def PseudoJIRL_TAIL : Pseudo<(outs), (ins GPR:$rj, simm16_lsl2:$imm16)>,
1722:                       PseudoInstExpansion<(JIRL R0, GPR:$rj,
1723:                                            simm16_lsl2:$imm16)>;
1724: 
1725: /// call/tail macro instructions
1726: let isCall = 1, isBarrier = 1, isCodeGenOnly = 0, isAsmParserOnly = 1,
1727:     Defs = [R1], hasSideEffects = 0, mayStore = 0, mayLoad = 0 in
1728: def PseudoCALL : Pseudo<(outs), (ins bare_symbol:$dst), [], "call", "$dst">;
1729: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [R3],
1730:     isCodeGenOnly = 0, isAsmParserOnly = 1, hasSideEffects = 0,
1731:     mayStore = 0, mayLoad = 0 in
1732: def PseudoTAIL : Pseudo<(outs), (ins GPR:$tmp, bare_symbol:$dst), [],
1733:                         "tail", "$tmp, $dst">;
1734: 
1735: /// call30/tail30 macro instructions
1736: let isCall = 1, isBarrier = 1, isCodeGenOnly = 0, isAsmParserOnly = 1,
1737:     Defs = [R1], hasSideEffects = 0, mayStore = 0, mayLoad = 0 in
1738: def PseudoCALL30 : Pseudo<(outs), (ins bare_symbol:$dst), [], "call30", "$dst">;
1739: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [R3],
1740:     isCodeGenOnly = 0, isAsmParserOnly = 1, hasSideEffects = 0,
1741:     mayStore = 0, mayLoad = 0 in
1742: def PseudoTAIL30 : Pseudo<(outs), (ins GPR:$tmp, bare_symbol:$dst), [],
1743:                           "tail30", "$tmp, $dst">;
1744: 
1745: /// call36/taill36 macro instructions
1746: let isCall = 1, isBarrier = 1, isCodeGenOnly = 0, isAsmParserOnly = 1,
1747:     Defs = [R1], hasSideEffects = 0, mayStore = 0, mayLoad = 0 in
1748: def PseudoCALL36 : Pseudo<(outs), (ins bare_symbol:$dst), [],
1749:                           "call36", "$dst">,
1750:                    Requires<[IsLA64]>;
1751: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [R3],
1752:     isCodeGenOnly = 0, isAsmParserOnly = 1, hasSideEffects = 0,
1753:     mayStore = 0, mayLoad = 0 in
1754: def PseudoTAIL36 : Pseudo<(outs), (ins GPR:$tmp, bare_symbol:$dst), [],
1755:                           "tail36", "$tmp, $dst">,
1756:                    Requires<[IsLA64]>;
1757: 
1758: // This is a special case of the ADD_W/D instruction used to facilitate the use
1759: // of a fourth operand to emit a relocation on a symbol relating to this
1760: // instruction. The relocation does not affect any bits of the instruction itself
```
- **EN**: This block declares or refines TableGen records such as `PseudoJIRL_TAIL`, `PseudoCALL`, `PseudoTAIL`, `PseudoCALL30`, `PseudoTAIL30`, `PseudoCALL36`.
- **CN**: 该代码块声明或细化了 `PseudoJIRL_TAIL`, `PseudoCALL`, `PseudoTAIL`, `PseudoCALL30`, `PseudoTAIL30`, `PseudoCALL36` 等 TableGen 记录。

### Lines 1761-1800 / 第 1761-1800 行
```tablegen
1761: // but is used as a hint to the linker.
1762: let hasSideEffects = 0, mayLoad = 0, mayStore = 0, isCodeGenOnly = 0 in {
1763: def PseudoAddTPRel_W : Pseudo<(outs GPR:$rd),
1764:                               (ins GPR:$rj, GPR:$rk, tprel_add_symbol:$sym), [],
1765:                               "add.w", "$rd, $rj, $rk, $sym">,
1766:                               Requires<[IsLA32]>;
1767: def PseudoAddTPRel_D : Pseudo<(outs GPR:$rd),
1768:                               (ins GPR:$rj, GPR:$rk, tprel_add_symbol:$sym), [],
1769:                               "add.d", "$rd, $rj, $rk, $sym">,
1770:                               Requires<[IsLA64]>;
1771: }
1772: 
1773: /// Load address (la*) macro instructions.
1774: 
1775: // Define isCodeGenOnly = 0 to expose them to tablegened assembly parser.
1776: let hasSideEffects = 0, mayLoad = 0, mayStore = 0, isCodeGenOnly = 0,
1777:     isAsmParserOnly = 1 in {
1778: def PseudoLA_ABS : Pseudo<(outs GPR:$dst), (ins bare_symbol:$src), [],
1779:                           "la.abs", "$dst, $src">;
1780: def PseudoLA_ABS_LARGE : Pseudo<(outs GPR:$dst),
1781:                                 (ins GPR:$tmp, bare_symbol:$src), [],
1782:                                 "la.abs", "$dst, $src">;
1783: def PseudoLA_PCREL : Pseudo<(outs GPR:$dst), (ins bare_symbol:$src), [],
1784:                             "la.pcrel", "$dst, $src">;
1785: def PseudoLA_TLS_LD : Pseudo<(outs GPR:$dst), (ins bare_symbol:$src), [],
1786:                              "la.tls.ld", "$dst, $src">;
1787: def PseudoLA_TLS_GD : Pseudo<(outs GPR:$dst), (ins bare_symbol:$src), [],
1788:                              "la.tls.gd", "$dst, $src">;
1789: def PseudoLA_PCREL_LARGE : Pseudo<(outs GPR:$dst),
1790:                                   (ins GPR:$tmp, bare_symbol:$src), [],
1791:                                   "la.pcrel", "$dst, $tmp, $src">,
1792:                            Requires<[IsLA64]>;
1793: def PseudoLA_TLS_LE : Pseudo<(outs GPR:$dst), (ins bare_symbol:$src), [],
1794:                              "la.tls.le", "$dst, $src">;
1795: def PseudoLA_TLS_LD_LARGE : Pseudo<(outs GPR:$dst),
1796:                                    (ins GPR:$tmp, bare_symbol:$src), [],
1797:                                    "la.tls.ld", "$dst, $tmp, $src">,
1798:                             Requires<[IsLA64]>;
1799: def PseudoLA_TLS_GD_LARGE : Pseudo<(outs GPR:$dst),
1800:                                    (ins GPR:$tmp, bare_symbol:$src), [],
```
- **EN**: This block declares or refines TableGen records such as `PseudoAddTPRel_W`, `PseudoAddTPRel_D`, `PseudoLA_ABS`, `PseudoLA_ABS_LARGE`, `PseudoLA_PCREL`, `PseudoLA_TLS_LD`.
- **CN**: 该代码块声明或细化了 `PseudoAddTPRel_W`, `PseudoAddTPRel_D`, `PseudoLA_ABS`, `PseudoLA_ABS_LARGE`, `PseudoLA_PCREL`, `PseudoLA_TLS_LD` 等 TableGen 记录。

### Lines 1801-1840 / 第 1801-1840 行
```tablegen
1801:                                    "la.tls.gd", "$dst, $tmp, $src">,
1802:                             Requires<[IsLA64]>;
1803: }
1804: let hasSideEffects = 0, mayLoad = 1, mayStore = 0, isCodeGenOnly = 0,
1805:     isAsmParserOnly = 1 in {
1806: def PseudoLA_GOT : Pseudo<(outs GPR:$dst), (ins bare_symbol:$src), [],
1807:                           "la.got", "$dst, $src">;
1808: def PseudoLA_TLS_IE : Pseudo<(outs GPR:$dst), (ins bare_symbol:$src), [],
1809:                              "la.tls.ie", "$dst, $src">;
1810: def PseudoLA_GOT_LARGE : Pseudo<(outs GPR:$dst),
1811:                                 (ins GPR:$tmp, bare_symbol:$src), [],
1812:                                 "la.got", "$dst, $tmp, $src">,
1813:                          Requires<[IsLA64]>;
1814: def PseudoLA_TLS_IE_LARGE : Pseudo<(outs GPR:$dst),
1815:                                    (ins GPR:$tmp, bare_symbol:$src), [],
1816:                                    "la.tls.ie", "$dst, $tmp, $src">,
1817:                             Requires<[IsLA64]>;
1818: }
1819: 
1820: // Used for expand PseudoLA_TLS_DESC_* instructions.
1821: let isCall = 1, isBarrier = 1, hasSideEffects = 0, mayStore = 0, mayLoad = 0,
1822:     Defs = [R4], Uses = [R4] in
1823: def PseudoDESC_CALL : Pseudo<(outs GPR:$rd), (ins GPR:$rj, simm16_lsl2:$imm16)>,
1824:                       PseudoInstExpansion<(JIRL GPR:$rd, GPR:$rj,
1825:                                            simm16_lsl2:$imm16)>;
1826: // TLSDESC
1827: let hasSideEffects = 0, mayLoad = 1, mayStore = 0, isCodeGenOnly = 0,
1828:     isAsmParserOnly = 1, Defs = [R1] in
1829: def PseudoLA_TLS_DESC : Pseudo<(outs GPR:$dst), (ins bare_symbol:$src), [],
1830:                                "la.tls.desc", "$dst, $src">;
1831: let isCall = 1, isBarrier = 1, hasSideEffects = 0, mayStore = 0, mayLoad = 0,
1832:     isCodeGenOnly = 0, isAsmParserOnly = 1, Defs = [R1, R4] in
1833: def PseudoLA_TLS_DESC_LARGE : Pseudo<(outs GPR:$dst),
1834:                                      (ins GPR:$tmp, bare_symbol:$src), [],
1835:                                      "la.tls.desc", "$dst, $tmp, $src">,
1836:                               Requires<[IsLA64]>;
1837: 
1838: // Load address inst alias: "la", "la.global" and "la.local".
1839: // Default:
1840: //     la = la.global = la.got
```
- **EN**: This block declares or refines TableGen records such as `PseudoLA_GOT`, `PseudoLA_TLS_IE`, `PseudoLA_GOT_LARGE`, `PseudoLA_TLS_IE_LARGE`, `PseudoDESC_CALL`, `PseudoLA_TLS_DESC`.
- **CN**: 该代码块声明或细化了 `PseudoLA_GOT`, `PseudoLA_TLS_IE`, `PseudoLA_GOT_LARGE`, `PseudoLA_TLS_IE_LARGE`, `PseudoDESC_CALL`, `PseudoLA_TLS_DESC` 等 TableGen 记录。

### Lines 1841-1880 / 第 1841-1880 行
```tablegen
1841: //     la.local = la.pcrel
1842: // With feature "+la-global-with-pcrel":
1843: //     la = la.global = la.pcrel
1844: // With feature "+la-global-with-abs":
1845: //     la = la.global = la.abs
1846: // With feature "+la-local-with-abs":
1847: //     la.local = la.abs
1848: // With features "+la-global-with-pcrel,+la-global-with-abs"(disorder):
1849: //     la = la.global = la.pcrel
1850: // Note: To keep consistent with gnu-as behavior, the "la" can only have one
1851: //       register operand.
1852: def : InstAlias<"la $dst, $src", (PseudoLA_GOT GPR:$dst, bare_symbol:$src)>;
1853: def : InstAlias<"la.global $dst, $src",
1854:                 (PseudoLA_GOT GPR:$dst, bare_symbol:$src)>;
1855: def : InstAlias<"la.global $dst, $tmp, $src",
1856:                 (PseudoLA_GOT_LARGE GPR:$dst, GPR:$tmp, bare_symbol:$src)>;
1857: def : InstAlias<"la.local $dst, $src",
1858:                 (PseudoLA_PCREL GPR:$dst, bare_symbol:$src)>;
1859: def : InstAlias<"la.local $dst, $tmp, $src",
1860:                 (PseudoLA_PCREL_LARGE GPR:$dst, GPR:$tmp, bare_symbol:$src)>;
1861: 
1862: // Note: Keep HasLaGlobalWithPcrel before HasLaGlobalWithAbs to ensure
1863: // "la-global-with-pcrel" takes effect when bose "la-global-with-pcrel" and
1864: // "la-global-with-abs" are enabled.
1865: let Predicates = [HasLaGlobalWithPcrel] in {
1866: def : InstAlias<"la $dst, $src", (PseudoLA_PCREL GPR:$dst, bare_symbol:$src)>;
1867: def : InstAlias<"la.global $dst, $src",
1868:                 (PseudoLA_PCREL GPR:$dst, bare_symbol:$src)>;
1869: def : InstAlias<"la.global $dst, $tmp, $src",
1870:                 (PseudoLA_PCREL_LARGE GPR:$dst, GPR:$tmp, bare_symbol:$src)>;
1871: } // Predicates = [HasLaGlobalWithPcrel]
1872: 
1873: let Predicates = [HasLaGlobalWithAbs] in {
1874: def : InstAlias<"la $dst, $src", (PseudoLA_ABS GPR:$dst, bare_symbol:$src)>;
1875: def : InstAlias<"la.global $dst, $src",
1876:                 (PseudoLA_ABS GPR:$dst, bare_symbol:$src)>;
1877: def : InstAlias<"la.global $dst, $tmp, $src",
1878:                 (PseudoLA_ABS_LARGE GPR:$dst, GPR:$tmp, bare_symbol:$src)>;
1879: } // Predicates = [HasLaGlobalWithAbs]
1880: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1881-1920 / 第 1881-1920 行
```tablegen
1881: let Predicates = [HasLaLocalWithAbs] in {
1882: def : InstAlias<"la.local $dst, $src",
1883:                 (PseudoLA_ABS GPR:$dst, bare_symbol:$src)>;
1884: def : InstAlias<"la.local $dst, $tmp, $src",
1885:                 (PseudoLA_ABS_LARGE GPR:$dst, GPR:$tmp, bare_symbol:$src)>;
1886: } // Predicates = [HasLaLocalWithAbs]
1887: 
1888: /// BSTRINS and BSTRPICK
1889: 
1890: let Predicates = [IsLA32, Has32S] in {
1891: def : Pat<(loongarch_bstrins GPR:$rd, GPR:$rj, uimm5:$msbd, uimm5:$lsbd),
1892:           (BSTRINS_W GPR:$rd, GPR:$rj, uimm5:$msbd, uimm5:$lsbd)>;
1893: def : Pat<(loongarch_bstrpick GPR:$rj, uimm5:$msbd, uimm5:$lsbd),
1894:           (BSTRPICK_W GPR:$rj, uimm5:$msbd, uimm5:$lsbd)>;
1895: } // Predicates = [IsLA32, Has32S]
1896: 
1897: let Predicates = [IsLA64] in {
1898: def : Pat<(loongarch_bstrins GPR:$rd, GPR:$rj, uimm6:$msbd, uimm6:$lsbd),
1899:           (BSTRINS_D GPR:$rd, GPR:$rj, uimm6:$msbd, uimm6:$lsbd)>;
1900: def : Pat<(loongarch_bstrpick GPR:$rj, uimm6:$msbd, uimm6:$lsbd),
1901:           (BSTRPICK_D GPR:$rj, uimm6:$msbd, uimm6:$lsbd)>;
1902: } // Predicates = [IsLA64]
1903: 
1904: /// Byte-swapping and bit-reversal
1905: 
1906: def : Pat<(loongarch_revb_2h GPR:$rj), (REVB_2H GPR:$rj)>;
1907: def : Pat<(loongarch_bitrev_4b GPR:$rj), (BITREV_4B GPR:$rj)>;
1908: 
1909: let Predicates = [IsLA32, Has32S] in {
1910: def : Pat<(bswap GPR:$rj), (ROTRI_W (REVB_2H GPR:$rj), 16)>;
1911: def : Pat<(bitreverse GPR:$rj), (BITREV_W GPR:$rj)>;
1912: def : Pat<(bswap (bitreverse GPR:$rj)), (BITREV_4B GPR:$rj)>;
1913: def : Pat<(bitreverse (bswap GPR:$rj)), (BITREV_4B GPR:$rj)>;
1914: } // Predicates = [IsLA32, Has32S]
1915: 
1916: let Predicates = [IsLA64] in {
1917: def : Pat<(loongarch_revb_2w GPR:$rj), (REVB_2W GPR:$rj)>;
1918: def : Pat<(bswap GPR:$rj), (REVB_D GPR:$rj)>;
1919: def : Pat<(loongarch_bitrev_8b GPR:$rj), (BITREV_8B GPR:$rj)>;
1920: def : Pat<(loongarch_bitrev_w GPR:$rj), (BITREV_W GPR:$rj)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1921-1960 / 第 1921-1960 行
```tablegen
1921: def : Pat<(bitreverse GPR:$rj), (BITREV_D GPR:$rj)>;
1922: def : Pat<(bswap (bitreverse GPR:$rj)), (BITREV_8B GPR:$rj)>;
1923: def : Pat<(bitreverse (bswap GPR:$rj)), (BITREV_8B GPR:$rj)>;
1924: } // Predicates = [IsLA64]
1925: 
1926: /// Loads
1927: 
1928: multiclass LdPat<PatFrags LoadOp, LAInst Inst, ValueType vt = GRLenVT> {
1929:   def : Pat<(vt (LoadOp BaseAddr:$rj)), (Inst BaseAddr:$rj, 0)>;
1930:   def : Pat<(vt (LoadOp (AddrConstant GPR:$rj, simm12:$imm12))),
1931:             (Inst GPR:$rj, simm12:$imm12)>;
1932:   def : Pat<(vt (LoadOp (AddLike BaseAddr:$rj, simm12:$imm12))),
1933:             (Inst BaseAddr:$rj, simm12:$imm12)>;
1934: }
1935: 
1936: defm : LdPat<sextloadi8, LD_B>;
1937: defm : LdPat<extloadi8, LD_B>;
1938: defm : LdPat<sextloadi16, LD_H>;
1939: defm : LdPat<extloadi16, LD_H>;
1940: defm : LdPat<load, LD_W>, Requires<[IsLA32]>;
1941: defm : LdPat<zextloadi8, LD_BU>;
1942: defm : LdPat<zextloadi16, LD_HU>;
1943: let Predicates = [IsLA64] in {
1944: defm : LdPat<sextloadi32, LD_W, i64>;
1945: defm : LdPat<extloadi32, LD_W, i64>;
1946: defm : LdPat<zextloadi32, LD_WU, i64>;
1947: defm : LdPat<load, LD_D, i64>;
1948: } // Predicates = [IsLA64]
1949: 
1950: // LA64 register-register-addressed loads
1951: let Predicates = [IsLA64] in {
1952: class RegRegLdPat<PatFrag LoadOp, LAInst Inst, ValueType vt>
1953:   : Pat<(vt (LoadOp (add NonFIBaseAddr:$rj, GPR:$rk))),
1954:         (Inst NonFIBaseAddr:$rj, GPR:$rk)>;
1955: 
1956: def : RegRegLdPat<extloadi8, LDX_B, i64>;
1957: def : RegRegLdPat<sextloadi8, LDX_B, i64>;
1958: def : RegRegLdPat<zextloadi8, LDX_BU, i64>;
1959: def : RegRegLdPat<extloadi16, LDX_H, i64>;
1960: def : RegRegLdPat<sextloadi16, LDX_H, i64>;
```
- **EN**: This block declares or refines TableGen records such as `LdPat`, `RegRegLdPat`.
- **CN**: 该代码块声明或细化了 `LdPat`, `RegRegLdPat` 等 TableGen 记录。

### Lines 1961-2000 / 第 1961-2000 行
```tablegen
1961: def : RegRegLdPat<zextloadi16, LDX_HU, i64>;
1962: def : RegRegLdPat<extloadi32, LDX_W, i64>;
1963: def : RegRegLdPat<sextloadi32, LDX_W, i64>;
1964: def : RegRegLdPat<zextloadi32, LDX_WU, i64>;
1965: def : RegRegLdPat<load, LDX_D, i64>;
1966: } // Predicates = [IsLA64]
1967: 
1968: /// Stores
1969: 
1970: multiclass StPat<PatFrag StoreOp, LAInst Inst, RegisterClass StTy,
1971:                  ValueType vt> {
1972:   def : Pat<(StoreOp (vt StTy:$rd), BaseAddr:$rj),
1973:             (Inst StTy:$rd, BaseAddr:$rj, 0)>;
1974:   def : Pat<(StoreOp (vt StTy:$rs2), (AddrConstant GPR:$rj, simm12:$imm12)),
1975:             (Inst StTy:$rs2, GPR:$rj, simm12:$imm12)>;
1976:   def : Pat<(StoreOp (vt StTy:$rd), (AddLike BaseAddr:$rj, simm12:$imm12)),
1977:             (Inst StTy:$rd, BaseAddr:$rj, simm12:$imm12)>;
1978: }
1979: 
1980: defm : StPat<truncstorei8, ST_B, GPR, GRLenVT>;
1981: defm : StPat<truncstorei16, ST_H, GPR, GRLenVT>;
1982: defm : StPat<store, ST_W, GPR, i32>, Requires<[IsLA32]>;
1983: let Predicates = [IsLA64] in {
1984: defm : StPat<truncstorei32, ST_W, GPR, i64>;
1985: defm : StPat<store, ST_D, GPR, i64>;
1986: } // Predicates = [IsLA64]
1987: 
1988: let Predicates = [IsLA64] in {
1989: def : Pat<(i64 (sextloadi32 (AddLike BaseAddr:$rj, simm14_lsl2:$imm14))),
1990:           (LDPTR_W BaseAddr:$rj, simm14_lsl2:$imm14)>;
1991: def : Pat<(i64 (load (AddLike BaseAddr:$rj, simm14_lsl2:$imm14))),
1992:           (LDPTR_D BaseAddr:$rj, simm14_lsl2:$imm14)>;
1993: def : Pat<(truncstorei32 (i64 GPR:$rd),
1994:                          (AddLike BaseAddr:$rj, simm14_lsl2:$imm14)),
1995:           (STPTR_W GPR:$rd, BaseAddr:$rj, simm14_lsl2:$imm14)>;
1996: def : Pat<(store (i64 GPR:$rd), (AddLike BaseAddr:$rj, simm14_lsl2:$imm14)),
1997:           (STPTR_D GPR:$rd, BaseAddr:$rj, simm14_lsl2:$imm14)>;
1998: } // Predicates = [IsLA64]
1999: 
2000: // LA64 register-register-addressed stores
```
- **EN**: This block declares or refines TableGen records such as `StPat`.
- **CN**: 该代码块声明或细化了 `StPat` 等 TableGen 记录。

### Lines 2001-2040 / 第 2001-2040 行
```tablegen
2001: let Predicates = [IsLA64] in {
2002: class RegRegStPat<PatFrag StoreOp, LAInst Inst, RegisterClass StTy,
2003:                   ValueType vt>
2004:   : Pat<(StoreOp (vt StTy:$rd), (add NonFIBaseAddr:$rj, GPR:$rk)),
2005:         (Inst StTy:$rd, NonFIBaseAddr:$rj, GPR:$rk)>;
2006: 
2007: def : RegRegStPat<truncstorei8, STX_B, GPR, i64>;
2008: def : RegRegStPat<truncstorei16, STX_H, GPR, i64>;
2009: def : RegRegStPat<truncstorei32, STX_W, GPR, i64>;
2010: def : RegRegStPat<store, STX_D, GPR, i64>;
2011: } // Predicates = [IsLA64]
2012: 
2013: /// Atomic loads and stores
2014: 
2015: // DBAR hint encoding for LA664 and later micro-architectures, paraphrased from
2016: // the Linux patch revealing it [1]:
2017: //
2018: // - Bit 4: kind of constraint (0: completion, 1: ordering)
2019: // - Bit 3: barrier for previous read (0: true, 1: false)
2020: // - Bit 2: barrier for previous write (0: true, 1: false)
2021: // - Bit 1: barrier for succeeding read (0: true, 1: false)
2022: // - Bit 0: barrier for succeeding write (0: true, 1: false)
2023: //
2024: // Hint 0x700: barrier for "read after read" from the same address, which is
2025: // e.g. needed by LL-SC loops on older models. (DBAR 0x700 behaves the same as
2026: // nop if such reordering is disabled on supporting newer models.)
2027: //
2028: // [1]: https://lore.kernel.org/loongarch/20230516124536.535343-1-chenhuacai@loongson.cn/
2029: //
2030: // Implementations without support for the finer-granularity hints simply treat
2031: // all as the full barrier (DBAR 0), so we can unconditionally start emiting the
2032: // more precise hints right away.
2033: 
2034: def : Pat<(atomic_fence 4, timm), (DBAR 0b10100)>; // acquire
2035: def : Pat<(atomic_fence 5, timm), (DBAR 0b10010)>; // release
2036: def : Pat<(atomic_fence 6, timm), (DBAR 0b10000)>; // acqrel
2037: def : Pat<(atomic_fence 7, timm), (DBAR 0b10000)>; // seqcst
2038: 
2039: defm : LdPat<atomic_load_asext_8, LD_B>;
2040: defm : LdPat<atomic_load_asext_16, LD_H>;
```
- **EN**: This block declares or refines TableGen records such as `RegRegStPat`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `RegRegStPat` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2041-2080 / 第 2041-2080 行
```tablegen
2041: defm : LdPat<atomic_load_nonext_32, LD_W>, Requires<[IsLA32]>;
2042: defm : LdPat<atomic_load_asext_32, LD_W>, Requires<[IsLA64]>;
2043: 
2044: class release_seqcst_store<PatFrag base>
2045:     : PatFrag<(ops node:$val, node:$ptr), (base node:$val, node:$ptr), [{
2046:   AtomicOrdering Ordering = cast<AtomicSDNode>(N)->getSuccessOrdering();
2047:   return isReleaseOrStronger(Ordering);
2048: }]>;
2049: 
2050: class unordered_monotonic_store<PatFrag base>
2051:     : PatFrag<(ops node:$val, node:$ptr), (base node:$val, node:$ptr), [{
2052:   AtomicOrdering Ordering = cast<AtomicSDNode>(N)->getSuccessOrdering();
2053:   return !isReleaseOrStronger(Ordering);
2054: }]>;
2055: 
2056: def atomic_store_release_seqcst_32 : release_seqcst_store<atomic_store_32>;
2057: def atomic_store_release_seqcst_64 : release_seqcst_store<atomic_store_64>;
2058: def atomic_store_unordered_monotonic_32
2059:     : unordered_monotonic_store<atomic_store_32>;
2060: def atomic_store_unordered_monotonic_64
2061:     : unordered_monotonic_store<atomic_store_64>;
2062: 
2063: defm : StPat<atomic_store_8, ST_B, GPR, GRLenVT>;
2064: defm : StPat<atomic_store_16, ST_H, GPR, GRLenVT>;
2065: defm : StPat<atomic_store_unordered_monotonic_32, ST_W, GPR, i32>,
2066:                    Requires<[IsLA32]>;
2067: 
2068: def PseudoAtomicStoreW
2069:   : Pseudo<(outs GPR:$dst), (ins GPR:$rk, GPR:$rj)>,
2070:            PseudoInstExpansion<(AMSWAP__DB_W R0, GPR:$rk, GPRMemAtomic:$rj)>;
2071: 
2072: def : Pat<(atomic_store_release_seqcst_32 GPR:$rj, GPR:$rk),
2073:           (PseudoAtomicStoreW GPR:$rj, GPR:$rk)>;
2074: 
2075: let Predicates = [IsLA64] in {
2076: def PseudoAtomicStoreD
2077:   : Pseudo<(outs GPR:$dst), (ins GPR:$rk, GPR:$rj)>,
2078:            PseudoInstExpansion<(AMSWAP__DB_D R0, GPR:$rk, GPRMemAtomic:$rj)>;
2079: 
2080: def : Pat<(atomic_store_release_seqcst_64 GPR:$rj, GPR:$rk),
```
- **EN**: This block declares or refines TableGen records such as `release_seqcst_store`, `unordered_monotonic_store`, `atomic_store_release_seqcst_32`, `atomic_store_release_seqcst_64`, `atomic_store_unordered_monotonic_32`, `atomic_store_unordered_monotonic_64`.
- **CN**: 该代码块声明或细化了 `release_seqcst_store`, `unordered_monotonic_store`, `atomic_store_release_seqcst_32`, `atomic_store_release_seqcst_64`, `atomic_store_unordered_monotonic_32`, `atomic_store_unordered_monotonic_64` 等 TableGen 记录。

### Lines 2081-2120 / 第 2081-2120 行
```tablegen
2081:           (PseudoAtomicStoreD GPR:$rj, GPR:$rk)>;
2082: 
2083: defm : LdPat<atomic_load_nonext_64, LD_D>;
2084: defm : StPat<atomic_store_unordered_monotonic_32, ST_W, GPR, i64>;
2085: defm : StPat<atomic_store_unordered_monotonic_64, ST_D, GPR, i64>;
2086: } // Predicates = [IsLA64]
2087: 
2088: /// Atomic Ops
2089: 
2090: class PseudoMaskedAM
2091:     : Pseudo<(outs GPR:$res, GPR:$scratch),
2092:              (ins GPR:$addr, GPR:$incr, GPR:$mask, grlenimm:$ordering)> {
2093:   let Constraints = "@earlyclobber $res,@earlyclobber $scratch";
2094:   let mayLoad = 1;
2095:   let mayStore = 1;
2096:   let hasSideEffects = 0;
2097:   let Size = 36;
2098: }
2099: 
2100: def PseudoMaskedAtomicSwap32 : PseudoMaskedAM;
2101: def PseudoMaskedAtomicLoadAdd32 : PseudoMaskedAM;
2102: def PseudoMaskedAtomicLoadSub32 : PseudoMaskedAM;
2103: def PseudoMaskedAtomicLoadNand32 : PseudoMaskedAM;
2104: 
2105: class PseudoAM : Pseudo<(outs GPR:$res, GPR:$scratch),
2106:                         (ins GPR:$addr, GPR:$incr, grlenimm:$ordering)> {
2107:   let Constraints = "@earlyclobber $res,@earlyclobber $scratch";
2108:   let mayLoad = 1;
2109:   let mayStore = 1;
2110:   let hasSideEffects = 0;
2111:   let Size = 24;
2112: }
2113: 
2114: def PseudoAtomicSwap32 : PseudoAM;
2115: def PseudoAtomicLoadNand32 : PseudoAM;
2116: def PseudoAtomicLoadNand64 : PseudoAM;
2117: def PseudoAtomicLoadAdd32 : PseudoAM;
2118: def PseudoAtomicLoadSub32 : PseudoAM;
2119: def PseudoAtomicLoadAnd32 : PseudoAM;
2120: def PseudoAtomicLoadOr32 : PseudoAM;
```
- **EN**: This block declares or refines TableGen records such as `PseudoMaskedAM`, `PseudoMaskedAtomicSwap32`, `PseudoMaskedAtomicLoadAdd32`, `PseudoMaskedAtomicLoadSub32`, `PseudoMaskedAtomicLoadNand32`, `PseudoAM`.
- **CN**: 该代码块声明或细化了 `PseudoMaskedAM`, `PseudoMaskedAtomicSwap32`, `PseudoMaskedAtomicLoadAdd32`, `PseudoMaskedAtomicLoadSub32`, `PseudoMaskedAtomicLoadNand32`, `PseudoAM` 等 TableGen 记录。

### Lines 2121-2160 / 第 2121-2160 行
```tablegen
2121: def PseudoAtomicLoadXor32 : PseudoAM;
2122: def PseudoAtomicLoadUMax32 : PseudoAM;
2123: def PseudoAtomicLoadUMin32 : PseudoAM;
2124: def PseudoAtomicLoadMax32 : PseudoAM;
2125: def PseudoAtomicLoadMin32 : PseudoAM;
2126: 
2127: multiclass PseudoBinPat<string Op, Pseudo BinInst> {
2128:   def : Pat<(!cast<PatFrag>(Op#"_monotonic") GPR:$addr, GPR:$incr),
2129:             (BinInst GPR:$addr, GPR:$incr, 2)>;
2130:   def : Pat<(!cast<PatFrag>(Op#"_acquire") GPR:$addr, GPR:$incr),
2131:             (BinInst GPR:$addr, GPR:$incr, 4)>;
2132:   def : Pat<(!cast<PatFrag>(Op#"_release") GPR:$addr, GPR:$incr),
2133:             (BinInst GPR:$addr, GPR:$incr, 5)>;
2134:   def : Pat<(!cast<PatFrag>(Op#"_acq_rel") GPR:$addr, GPR:$incr),
2135:             (BinInst GPR:$addr, GPR:$incr, 6)>;
2136:   def : Pat<(!cast<PatFrag>(Op#"_seq_cst") GPR:$addr, GPR:$incr),
2137:             (BinInst GPR:$addr, GPR:$incr, 7)>;
2138: }
2139: 
2140: class PseudoMaskedAMUMinUMax
2141:     : Pseudo<(outs GPR:$res, GPR:$scratch1, GPR:$scratch2),
2142:              (ins GPR:$addr, GPR:$incr, GPR:$mask, grlenimm:$ordering)> {
2143:   let Constraints = "@earlyclobber $res,@earlyclobber $scratch1,"
2144:                     "@earlyclobber $scratch2";
2145:   let mayLoad = 1;
2146:   let mayStore = 1;
2147:   let hasSideEffects = 0;
2148:   let Size = 48;
2149: }
2150: 
2151: def PseudoMaskedAtomicLoadUMax32 : PseudoMaskedAMUMinUMax;
2152: def PseudoMaskedAtomicLoadUMin32 : PseudoMaskedAMUMinUMax;
2153: 
2154: class PseudoMaskedAMMinMax
2155:     : Pseudo<(outs GPR:$res, GPR:$scratch1, GPR:$scratch2),
2156:              (ins GPR:$addr, GPR:$incr, GPR:$mask, grlenimm:$sextshamt,
2157:               grlenimm:$ordering)> {
2158:   let Constraints = "@earlyclobber $res,@earlyclobber $scratch1,"
2159:                     "@earlyclobber $scratch2";
2160:   let mayLoad = 1;
```
- **EN**: This block declares or refines TableGen records such as `PseudoAtomicLoadXor32`, `PseudoAtomicLoadUMax32`, `PseudoAtomicLoadUMin32`, `PseudoAtomicLoadMax32`, `PseudoAtomicLoadMin32`, `PseudoBinPat`.
- **CN**: 该代码块声明或细化了 `PseudoAtomicLoadXor32`, `PseudoAtomicLoadUMax32`, `PseudoAtomicLoadUMin32`, `PseudoAtomicLoadMax32`, `PseudoAtomicLoadMin32`, `PseudoBinPat` 等 TableGen 记录。

### Lines 2161-2200 / 第 2161-2200 行
```tablegen
2161:   let mayStore = 1;
2162:   let hasSideEffects = 0;
2163:   let Size = 56;
2164: }
2165: 
2166: def PseudoMaskedAtomicLoadMax32 : PseudoMaskedAMMinMax;
2167: def PseudoMaskedAtomicLoadMin32 : PseudoMaskedAMMinMax;
2168: 
2169: // Data prefetch
2170: 
2171: // TODO: Supports for preldx instruction.
2172: def : Pat<(prefetch (AddrRegImm GPR:$rj, simm12:$imm12), (i32 0), timm, (i32 1)),
2173:           (PRELD 0, GPR:$rj, simm12:$imm12)>; // data prefetch for loads
2174: def : Pat<(prefetch (AddrRegImm GPR:$rj, simm12:$imm12), (i32 1), timm, (i32 1)),
2175:           (PRELD 8, GPR:$rj, simm12:$imm12)>; // data prefetch for stores
2176: 
2177: /// Compare and exchange
2178: 
2179: class PseudoCmpXchg
2180:     : Pseudo<(outs GPR:$res, GPR:$scratch),
2181:              (ins GPR:$addr, GPR:$cmpval, GPR:$newval, grlenimm:$fail_order)> {
2182:   let Constraints = "@earlyclobber $res,@earlyclobber $scratch";
2183:   let mayLoad = 1;
2184:   let mayStore = 1;
2185:   let hasSideEffects = 0;
2186:   let Size = 36;
2187: }
2188: 
2189: def PseudoCmpXchg32 : PseudoCmpXchg;
2190: def PseudoCmpXchg64 : PseudoCmpXchg;
2191: 
2192: class PseudoCmpXchg128Pat 
2193:     : Pseudo<(outs GPR:$res_lo, GPR:$res_hi, GPR:$scratch),
2194:              (ins GPR:$addr, GPR:$cmpval_lo, GPR:$cmpval_hi, 
2195:                   GPR:$newval_lo, GPR:$newval_hi)> {
2196:   let Constraints = "@earlyclobber $res_lo,@earlyclobber $res_hi,@earlyclobber $scratch";
2197:   let mayLoad = 1;
2198:   let mayStore = 1;
2199:   let hasSideEffects = 0;
2200:   let Size = 36;
```
- **EN**: This block declares or refines TableGen records such as `PseudoMaskedAtomicLoadMax32`, `PseudoMaskedAtomicLoadMin32`, `PseudoCmpXchg`, `PseudoCmpXchg32`, `PseudoCmpXchg64`, `PseudoCmpXchg128Pat`.
- **CN**: 该代码块声明或细化了 `PseudoMaskedAtomicLoadMax32`, `PseudoMaskedAtomicLoadMin32`, `PseudoCmpXchg`, `PseudoCmpXchg32`, `PseudoCmpXchg64`, `PseudoCmpXchg128Pat` 等 TableGen 记录。

### Lines 2201-2240 / 第 2201-2240 行
```tablegen
2201: }
2202: 
2203: def PseudoCmpXchg128 : PseudoCmpXchg128Pat;
2204: def PseudoCmpXchg128Acquire : PseudoCmpXchg128Pat;
2205: 
2206: def PseudoMaskedCmpXchg32
2207:     : Pseudo<(outs GPR:$res, GPR:$scratch),
2208:              (ins GPR:$addr, GPR:$cmpval, GPR:$newval, GPR:$mask,
2209:               grlenimm:$fail_order)> {
2210:   let Constraints = "@earlyclobber $res,@earlyclobber $scratch";
2211:   let mayLoad = 1;
2212:   let mayStore = 1;
2213:   let hasSideEffects = 0;
2214:   let Size = 44;
2215: }
2216: 
2217: class PseudoMaskedAMMinMaxPat<Intrinsic intrin, Pseudo AMInst>
2218:     : Pat<(intrin GPR:$addr, GPR:$incr, GPR:$mask, GPR:$shiftamt,
2219:            timm:$ordering),
2220:           (AMInst GPR:$addr, GPR:$incr, GPR:$mask, GPR:$shiftamt,
2221:            timm:$ordering)>;
2222: 
2223: class AtomicPat<Intrinsic intrin, Pseudo AMInst>
2224:     : Pat<(intrin GPR:$addr, GPR:$incr, GPR:$mask, timm:$ordering),
2225:           (AMInst GPR:$addr, GPR:$incr, GPR:$mask, timm:$ordering)>;
2226: 
2227: // These atomic cmpxchg PatFrags only care about the failure ordering.
2228: // The PatFrags defined by multiclass `ternary_atomic_op_ord` in
2229: // TargetSelectionDAG.td care about the merged memory ordering that is the
2230: // stronger one between success and failure. But for LoongArch LL-SC we only
2231: // need to care about the failure ordering as explained in PR #67391. So we
2232: // define these PatFrags that will be used to define cmpxchg pats below.
2233: multiclass ternary_atomic_op_failure_ord {
2234:   def NAME#_failure_monotonic : PatFrag<(ops node:$ptr, node:$cmp, node:$val),
2235:       (!cast<SDPatternOperator>(NAME) node:$ptr, node:$cmp, node:$val), [{
2236:     AtomicOrdering Ordering = cast<AtomicSDNode>(N)->getFailureOrdering();
2237:     return Ordering == AtomicOrdering::Monotonic;
2238:   }]>;
2239:   def NAME#_failure_acquire : PatFrag<(ops node:$ptr, node:$cmp, node:$val),
2240:       (!cast<SDPatternOperator>(NAME) node:$ptr, node:$cmp, node:$val), [{
```
- **EN**: This block declares or refines TableGen records such as `PseudoCmpXchg128`, `PseudoCmpXchg128Acquire`, `PseudoMaskedCmpXchg32`, `PseudoMaskedAMMinMaxPat`, `AtomicPat`, `ternary_atomic_op_failure_ord`.
- **CN**: 该代码块声明或细化了 `PseudoCmpXchg128`, `PseudoCmpXchg128Acquire`, `PseudoMaskedCmpXchg32`, `PseudoMaskedAMMinMaxPat`, `AtomicPat`, `ternary_atomic_op_failure_ord` 等 TableGen 记录。

### Lines 2241-2280 / 第 2241-2280 行
```tablegen
2241:     AtomicOrdering Ordering = cast<AtomicSDNode>(N)->getFailureOrdering();
2242:     return Ordering == AtomicOrdering::Acquire;
2243:   }]>;
2244:   def NAME#_failure_release : PatFrag<(ops node:$ptr, node:$cmp, node:$val),
2245:       (!cast<SDPatternOperator>(NAME) node:$ptr, node:$cmp, node:$val), [{
2246:     AtomicOrdering Ordering = cast<AtomicSDNode>(N)->getFailureOrdering();
2247:     return Ordering == AtomicOrdering::Release;
2248:   }]>;
2249:   def NAME#_failure_acq_rel : PatFrag<(ops node:$ptr, node:$cmp, node:$val),
2250:       (!cast<SDPatternOperator>(NAME) node:$ptr, node:$cmp, node:$val), [{
2251:     AtomicOrdering Ordering = cast<AtomicSDNode>(N)->getFailureOrdering();
2252:     return Ordering == AtomicOrdering::AcquireRelease;
2253:   }]>;
2254:   def NAME#_failure_seq_cst : PatFrag<(ops node:$ptr, node:$cmp, node:$val),
2255:       (!cast<SDPatternOperator>(NAME) node:$ptr, node:$cmp, node:$val), [{
2256:     AtomicOrdering Ordering = cast<AtomicSDNode>(N)->getFailureOrdering();
2257:     return Ordering == AtomicOrdering::SequentiallyConsistent;
2258:   }]>;
2259: }
2260: 
2261: defm atomic_cmp_swap_i32 : ternary_atomic_op_failure_ord;
2262: defm atomic_cmp_swap_i64 : ternary_atomic_op_failure_ord;
2263: 
2264: // Atomic operation for word and double word
2265: multiclass binary_atomic_op_wd<string inst, string op, string signed = ""> {
2266:   def : Pat<(!cast<PatFrag>(op#"_i32_monotonic") GPR:$rj, GPR:$rk),
2267:             (!cast<Instruction>(inst#"_W"#signed) GPR:$rk, GPR:$rj)>;
2268:   def : Pat<(!cast<PatFrag>(op#"_i64_monotonic") GPR:$rj, GPR:$rk),
2269:             (!cast<Instruction>(inst#"_D"#signed) GPR:$rk, GPR:$rj)>;
2270: 
2271:   def : Pat<(!cast<PatFrag>(op#"_i32") GPR:$rj, GPR:$rk),
2272:             (!cast<Instruction>(inst#"__DB_W"#signed) GPR:$rk, GPR:$rj)>;
2273:   def : Pat<(!cast<PatFrag>(op#"_i64") GPR:$rj, GPR:$rk),
2274:             (!cast<Instruction>(inst#"__DB_D"#signed) GPR:$rk, GPR:$rj)>;
2275: }
2276: 
2277: // Atomic operation for byte and half word
2278: multiclass binary_atomic_op_bh<string inst, string op> {
2279:   def : Pat<(!cast<PatFrag>(op#"_i8_monotonic") GPR:$rj, GPR:$rk),
2280:             (!cast<Instruction>(inst#"_B") GPR:$rk, GPR:$rj)>;
```
- **EN**: This block declares or refines TableGen records such as `NAME`, `NAME`, `NAME`, `atomic_cmp_swap_i32`, `atomic_cmp_swap_i64`, `binary_atomic_op_wd`.
- **CN**: 该代码块声明或细化了 `NAME`, `NAME`, `NAME`, `atomic_cmp_swap_i32`, `atomic_cmp_swap_i64`, `binary_atomic_op_wd` 等 TableGen 记录。

### Lines 2281-2320 / 第 2281-2320 行
```tablegen
2281:   def : Pat<(!cast<PatFrag>(op#"_i16_monotonic") GPR:$rj, GPR:$rk),
2282:             (!cast<Instruction>(inst#"_H") GPR:$rk, GPR:$rj)>;
2283: 
2284:   def : Pat<(!cast<PatFrag>(op#"_i8") GPR:$rj, GPR:$rk),
2285:             (!cast<Instruction>(inst#"__DB_B") GPR:$rk, GPR:$rj)>;
2286:   def : Pat<(!cast<PatFrag>(op#"_i16") GPR:$rj, GPR:$rk),
2287:             (!cast<Instruction>(inst#"__DB_H") GPR:$rk, GPR:$rj)>;
2288: }
2289: 
2290: let Predicates = [ HasLAM_BH, IsLA64 ] in {
2291: defm : binary_atomic_op_bh<"AMSWAP", "atomic_swap">;
2292: defm : binary_atomic_op_bh<"AMADD", "atomic_load_add">;
2293: def : Pat<(atomic_load_sub_i8_monotonic GPR:$rj, GPR:$rk),
2294:           (AMADD_B (SUB_W R0, GPR:$rk), GPR:$rj)>;
2295: def : Pat<(atomic_load_sub_i16_monotonic GPR:$rj, GPR:$rk),
2296:           (AMADD_H (SUB_W R0, GPR:$rk), GPR:$rj)>;
2297: 
2298: def : Pat<(atomic_load_sub_i8 GPR:$rj, GPR:$rk),
2299:           (AMADD__DB_B (SUB_W R0, GPR:$rk), GPR:$rj)>;
2300: def : Pat<(atomic_load_sub_i16 GPR:$rj, GPR:$rk),
2301:           (AMADD__DB_H (SUB_W R0, GPR:$rk), GPR:$rj)>;
2302: } // Predicates = [ IsLA64, HasLAM_BH ]
2303: 
2304: let Predicates = [ HasLAMCAS, IsLA64 ] in {
2305: def : Pat<(atomic_cmp_swap_i8_monotonic GPR:$addr, GPR:$cmp, GPR:$new),
2306:           (AMCAS_B GPR:$cmp, GPR:$new, GPR:$addr)>;
2307: def : Pat<(atomic_cmp_swap_i16_monotonic GPR:$addr, GPR:$cmp, GPR:$new),
2308:           (AMCAS_H GPR:$cmp, GPR:$new, GPR:$addr)>;
2309: def : Pat<(atomic_cmp_swap_i32_monotonic GPR:$addr, GPR:$cmp, GPR:$new),
2310:           (AMCAS_W GPR:$cmp, GPR:$new, GPR:$addr)>;
2311: def : Pat<(atomic_cmp_swap_i64_monotonic GPR:$addr, GPR:$cmp, GPR:$new),
2312:           (AMCAS_D GPR:$cmp, GPR:$new, GPR:$addr)>;
2313: 
2314: def : Pat<(atomic_cmp_swap_i8 GPR:$addr, GPR:$cmp, GPR:$new),
2315:           (AMCAS__DB_B GPR:$cmp, GPR:$new, GPR:$addr)>;
2316: def : Pat<(atomic_cmp_swap_i16 GPR:$addr, GPR:$cmp, GPR:$new),
2317:           (AMCAS__DB_H GPR:$cmp, GPR:$new, GPR:$addr)>;
2318: def : Pat<(atomic_cmp_swap_i32 GPR:$addr, GPR:$cmp, GPR:$new),
2319:           (AMCAS__DB_W GPR:$cmp, GPR:$new, GPR:$addr)>;
2320: def : Pat<(atomic_cmp_swap_i64 GPR:$addr, GPR:$cmp, GPR:$new),
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2321-2360 / 第 2321-2360 行
```tablegen
2321:           (AMCAS__DB_D GPR:$cmp, GPR:$new, GPR:$addr)>;
2322: }
2323: 
2324: // Ordering constants must be kept in sync with the AtomicOrdering enum in
2325: // AtomicOrdering.h.
2326: multiclass PseudoCmpXchgPat<string Op, Pseudo CmpXchgInst,
2327:                             ValueType vt = GRLenVT> {
2328:   def : Pat<(vt (!cast<PatFrag>(Op#"_failure_monotonic") GPR:$addr, GPR:$cmp, GPR:$new)),
2329:             (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 2)>;
2330:   def : Pat<(vt (!cast<PatFrag>(Op#"_failure_acquire") GPR:$addr, GPR:$cmp, GPR:$new)),
2331:             (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 4)>;
2332:   def : Pat<(vt (!cast<PatFrag>(Op#"_failure_release") GPR:$addr, GPR:$cmp, GPR:$new)),
2333:             (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 5)>;
2334:   def : Pat<(vt (!cast<PatFrag>(Op#"_failure_acq_rel") GPR:$addr, GPR:$cmp, GPR:$new)),
2335:             (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 6)>;
2336:   def : Pat<(vt (!cast<PatFrag>(Op#"_failure_seq_cst") GPR:$addr, GPR:$cmp, GPR:$new)),
2337:             (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 7)>;
2338: }
2339: 
2340: let Predicates = [IsLA64] in {
2341: defm : binary_atomic_op_wd<"AMSWAP", "atomic_swap">;
2342: defm : binary_atomic_op_wd<"AMADD", "atomic_load_add">;
2343: defm : binary_atomic_op_wd<"AMAND", "atomic_load_and">;
2344: defm : binary_atomic_op_wd<"AMOR", "atomic_load_or">;
2345: defm : binary_atomic_op_wd<"AMXOR", "atomic_load_xor">;
2346: defm : binary_atomic_op_wd<"AMMIN", "atomic_load_umin", "U">;
2347: defm : binary_atomic_op_wd<"AMMAX", "atomic_load_umax", "U">;
2348: defm : binary_atomic_op_wd<"AMMIN", "atomic_load_min">;
2349: defm : binary_atomic_op_wd<"AMMAX", "atomic_load_max">;
2350: def : Pat<(atomic_load_sub_i32_monotonic GPR:$rj, GPR:$rk),
2351:           (AMADD_W (SUB_W R0, GPR:$rk), GPR:$rj)>;
2352: def : Pat<(atomic_load_sub_i64_monotonic GPR:$rj, GPR:$rk),
2353:           (AMADD_D (SUB_D R0, GPR:$rk), GPR:$rj)>;
2354: 
2355: def : Pat<(atomic_load_sub_i32 GPR:$rj, GPR:$rk),
2356:           (AMADD__DB_W (SUB_W R0, GPR:$rk), GPR:$rj)>;
2357: def : Pat<(atomic_load_sub_i64 GPR:$rj, GPR:$rk),
2358:           (AMADD__DB_D (SUB_D R0, GPR:$rk), GPR:$rj)>;
2359: 
2360: def : AtomicPat<int_loongarch_masked_atomicrmw_xchg_i64,
```
- **EN**: This block declares or refines TableGen records such as `PseudoCmpXchgPat`.
- **CN**: 该代码块声明或细化了 `PseudoCmpXchgPat` 等 TableGen 记录。

### Lines 2361-2400 / 第 2361-2400 行
```tablegen
2361:                 PseudoMaskedAtomicSwap32>;
2362: def : AtomicPat<int_loongarch_masked_atomicrmw_add_i64,
2363:                 PseudoMaskedAtomicLoadAdd32>;
2364: def : AtomicPat<int_loongarch_masked_atomicrmw_sub_i64,
2365:                 PseudoMaskedAtomicLoadSub32>;
2366: defm : PseudoBinPat<"atomic_load_nand_i64", PseudoAtomicLoadNand64>;
2367: def : AtomicPat<int_loongarch_masked_atomicrmw_nand_i64,
2368:                 PseudoMaskedAtomicLoadNand32>;
2369: 
2370: def : AtomicPat<int_loongarch_masked_atomicrmw_umax_i64,
2371:                 PseudoMaskedAtomicLoadUMax32>;
2372: def : AtomicPat<int_loongarch_masked_atomicrmw_umin_i64,
2373:                 PseudoMaskedAtomicLoadUMin32>;
2374: 
2375: defm : PseudoCmpXchgPat<"atomic_cmp_swap_i64", PseudoCmpXchg64, i64>;
2376: def : Pat<(int_loongarch_masked_cmpxchg_i64
2377:             GPR:$addr, GPR:$cmpval, GPR:$newval, GPR:$mask, timm:$fail_order),
2378:           (PseudoMaskedCmpXchg32
2379:             GPR:$addr, GPR:$cmpval, GPR:$newval, GPR:$mask, timm:$fail_order)>;
2380: 
2381: def : PseudoMaskedAMMinMaxPat<int_loongarch_masked_atomicrmw_max_i64,
2382:                               PseudoMaskedAtomicLoadMax32>;
2383: def : PseudoMaskedAMMinMaxPat<int_loongarch_masked_atomicrmw_min_i64,
2384:                               PseudoMaskedAtomicLoadMin32>;
2385: } // Predicates = [IsLA64]
2386: 
2387: defm : PseudoCmpXchgPat<"atomic_cmp_swap_i32", PseudoCmpXchg32>;
2388: defm : PseudoBinPat<"atomic_load_nand_i32", PseudoAtomicLoadNand32>;
2389: 
2390: let Predicates = [IsLA32] in {
2391: def : AtomicPat<int_loongarch_masked_atomicrmw_xchg_i32,
2392:                 PseudoMaskedAtomicSwap32>;
2393: defm : PseudoBinPat<"atomic_swap_i32", PseudoAtomicSwap32>;
2394: def : AtomicPat<int_loongarch_masked_atomicrmw_add_i32,
2395:                 PseudoMaskedAtomicLoadAdd32>;
2396: def : AtomicPat<int_loongarch_masked_atomicrmw_sub_i32,
2397:                 PseudoMaskedAtomicLoadSub32>;
2398: def : AtomicPat<int_loongarch_masked_atomicrmw_nand_i32,
2399:                 PseudoMaskedAtomicLoadNand32>;
2400: def : AtomicPat<int_loongarch_masked_atomicrmw_umax_i32,
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2401-2440 / 第 2401-2440 行
```tablegen
2401:                 PseudoMaskedAtomicLoadUMax32>;
2402: def : AtomicPat<int_loongarch_masked_atomicrmw_umin_i32,
2403:                 PseudoMaskedAtomicLoadUMin32>;
2404: 
2405: def : PseudoMaskedAMMinMaxPat<int_loongarch_masked_atomicrmw_max_i32,
2406:                               PseudoMaskedAtomicLoadMax32>;
2407: def : PseudoMaskedAMMinMaxPat<int_loongarch_masked_atomicrmw_min_i32,
2408:                               PseudoMaskedAtomicLoadMin32>;
2409: 
2410: def : Pat<(int_loongarch_masked_cmpxchg_i32
2411:             GPR:$addr, GPR:$cmpval, GPR:$newval, GPR:$mask, timm:$fail_order),
2412:           (PseudoMaskedCmpXchg32
2413:             GPR:$addr, GPR:$cmpval, GPR:$newval, GPR:$mask, timm:$fail_order)>;
2414: 
2415: defm : PseudoBinPat<"atomic_load_add_i32", PseudoAtomicLoadAdd32>;
2416: defm : PseudoBinPat<"atomic_load_sub_i32", PseudoAtomicLoadSub32>;
2417: defm : PseudoBinPat<"atomic_load_and_i32", PseudoAtomicLoadAnd32>;
2418: defm : PseudoBinPat<"atomic_load_or_i32", PseudoAtomicLoadOr32>;
2419: defm : PseudoBinPat<"atomic_load_xor_i32", PseudoAtomicLoadXor32>;
2420: defm : PseudoBinPat<"atomic_load_umax_i32", PseudoAtomicLoadUMax32>;
2421: defm : PseudoBinPat<"atomic_load_umin_i32", PseudoAtomicLoadUMin32>;
2422: defm : PseudoBinPat<"atomic_load_max_i32", PseudoAtomicLoadMax32>;
2423: defm : PseudoBinPat<"atomic_load_min_i32", PseudoAtomicLoadMin32>;
2424: } // Predicates = [IsLA32]
2425: 
2426: /// Intrinsics
2427: 
2428: def : Pat<(int_loongarch_cacop_d timm:$op, i64:$rj, timm:$imm12),
2429:           (CACOP timm:$op, GPR:$rj, timm:$imm12)>;
2430: def : Pat<(int_loongarch_cacop_w i32:$op, i32:$rj, i32:$imm12),
2431:           (CACOP timm:$op, GPR:$rj, timm:$imm12)>;
2432: def : Pat<(loongarch_dbar uimm15:$imm15), (DBAR uimm15:$imm15)>;
2433: def : Pat<(loongarch_ibar uimm15:$imm15), (IBAR uimm15:$imm15)>;
2434: def : Pat<(loongarch_break uimm15:$imm15), (BREAK uimm15:$imm15)>;
2435: def : Pat<(loongarch_syscall uimm15:$imm15), (SYSCALL uimm15:$imm15)>;
2436: 
2437: let Predicates = [IsLA64] in {
2438: // CRC Check Instructions
2439: def : PatGprGpr<loongarch_crc_w_b_w, CRC_W_B_W>;
2440: def : PatGprGpr<loongarch_crc_w_h_w, CRC_W_H_W>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2441-2480 / 第 2441-2480 行
```tablegen
2441: def : PatGprGpr<loongarch_crc_w_w_w, CRC_W_W_W>;
2442: def : PatGprGpr<loongarch_crc_w_d_w, CRC_W_D_W>;
2443: def : PatGprGpr<loongarch_crcc_w_b_w, CRCC_W_B_W>;
2444: def : PatGprGpr<loongarch_crcc_w_h_w, CRCC_W_H_W>;
2445: def : PatGprGpr<loongarch_crcc_w_w_w, CRCC_W_W_W>;
2446: def : PatGprGpr<loongarch_crcc_w_d_w, CRCC_W_D_W>;
2447: } // Predicates = [IsLA64]
2448: 
2449: /// Other pseudo-instructions
2450: 
2451: // Pessimistically assume the stack pointer will be clobbered
2452: let Defs = [R3], Uses = [R3] in {
2453: def ADJCALLSTACKDOWN : Pseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
2454:                               [(callseq_start timm:$amt1, timm:$amt2)]>;
2455: def ADJCALLSTACKUP   : Pseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
2456:                               [(callseq_end timm:$amt1, timm:$amt2)]>;
2457: } // Defs = [R3], Uses = [R3]
2458: 
2459: //===----------------------------------------------------------------------===//
2460: // Assembler Pseudo Instructions
2461: //===----------------------------------------------------------------------===//
2462: 
2463: def : InstAlias<"nop", (ANDI R0, R0, 0)>;
2464: def : InstAlias<"move $dst, $src", (OR GPR:$dst, GPR:$src, R0)>;
2465: // `ret` is supported since binutils commit 20f2e2686c79a5ac (version 2.40 and
2466: // later).
2467: def : InstAlias<"ret", (JIRL R0, R1, 0)>;
2468: def : InstAlias<"jr $rj", (JIRL R0, GPR:$rj, 0)>;
2469: 
2470: def : InstAlias<"rdcntid.w $rj", (RDTIMEL_W R0, GPR:$rj)>;
2471: def : InstAlias<"rdcntvh.w $rd", (RDTIMEH_W GPR:$rd, R0)>;
2472: def : InstAlias<"rdcntvl.w $rd", (RDTIMEL_W GPR:$rd, R0)>;
2473: 
2474: // Branches implemented with aliases.
2475: // Disassemble branch instructions not having a $zero operand to the
2476: // canonical mnemonics respectively, but disassemble BLT/BGE with a $zero
2477: // operand to the corresponding pseudo-instruction.
2478: // GNU Binutils behave like this since 2.41, e.g. "bgt" will be recognised
2479: // by the assembler but always disassembles as "blt" by objdump, while "bgtz"
2480: // will come back intact.
```
- **EN**: This block declares or refines TableGen records such as `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP`.
- **CN**: 该代码块声明或细化了 `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP` 等 TableGen 记录。

### Lines 2481-2520 / 第 2481-2520 行
```tablegen
2481: // Match this behaviour by setting a zero weight for the b{gt,le}{,u}
2482: // patterns only.
2483: def : InstAlias<"bgt $rj, $rd, $imm16",
2484:                 (BLT GPR:$rd, GPR:$rj, simm16_lsl2_br:$imm16), 0>;
2485: def : InstAlias<"bgtu $rj, $rd, $imm16",
2486:                 (BLTU GPR:$rd, GPR:$rj, simm16_lsl2_br:$imm16), 0>;
2487: def : InstAlias<"ble $rj, $rd, $imm16",
2488:                 (BGE GPR:$rd, GPR:$rj, simm16_lsl2_br:$imm16), 0>;
2489: def : InstAlias<"bleu $rj, $rd, $imm16",
2490:                 (BGEU GPR:$rd, GPR:$rj, simm16_lsl2_br:$imm16), 0>;
2491: def : InstAlias<"bltz $rd, $imm16",
2492:                 (BLT GPR:$rd, R0, simm16_lsl2_br:$imm16)>;
2493: def : InstAlias<"bgtz $rj, $imm16",
2494:                 (BLT R0, GPR:$rj, simm16_lsl2_br:$imm16)>;
2495: def : InstAlias<"blez $rj, $imm16",
2496:                 (BGE R0, GPR:$rj, simm16_lsl2_br:$imm16)>;
2497: def : InstAlias<"bgez $rd, $imm16",
2498:                 (BGE GPR:$rd, R0, simm16_lsl2_br:$imm16)>;
2499: 
2500: // Load immediate.
2501: let hasSideEffects = 0, mayLoad = 0, mayStore = 0, isCodeGenOnly = 0,
2502:     isAsmParserOnly = 1 in {
2503: def PseudoLI_W : Pseudo<(outs GPR:$rd), (ins imm32:$imm), [],
2504:                         "li.w", "$rd, $imm">;
2505: def PseudoLI_D : Pseudo<(outs GPR:$rd), (ins imm64:$imm), [],
2506:                         "li.d", "$rd, $imm">, Requires<[IsLA64]>;
2507: }
2508: 
2509: //===----------------------------------------------------------------------===//
2510: // Basic Floating-Point Instructions
2511: //===----------------------------------------------------------------------===//
2512: 
2513: include "LoongArchFloat32InstrInfo.td"
2514: include "LoongArchFloat64InstrInfo.td"
2515: 
2516: let Predicates = [HasBasicF], usesCustomInserter = 1 in {
2517:   def WRFCSR : Pseudo<(outs), (ins uimm2:$fcsr, GPR:$src),
2518:                [(loongarch_movgr2fcsr uimm2:$fcsr, GRLenVT:$src)]>;
2519:   def RDFCSR : Pseudo<(outs GPR:$rd), (ins uimm2:$fcsr),
2520:                [(set GPR:$rd, (loongarch_movfcsr2gr uimm2:$fcsr))]>;
```
- **EN**: It composes TableGen records by including `LoongArchFloat32InstrInfo.td`, `LoongArchFloat64InstrInfo.td` and reusing previously declared backend fragments. This block declares or refines TableGen records such as `PseudoLI_W`, `PseudoLI_D`, `WRFCSR`, `RDFCSR`.
- **CN**: 它通过包含 `LoongArchFloat32InstrInfo.td`, `LoongArchFloat64InstrInfo.td` 组合 TableGen 记录，并复用此前声明的后端片段。 该代码块声明或细化了 `PseudoLI_W`, `PseudoLI_D`, `WRFCSR`, `RDFCSR` 等 TableGen 记录。

### Lines 2521-2560 / 第 2521-2560 行
```tablegen
2521: }
2522: 
2523: //===----------------------------------------------------------------------===//
2524: // Privilege Instructions
2525: //===----------------------------------------------------------------------===//
2526: 
2527: // CSR Access Instructions
2528: let hasSideEffects = 1 in
2529: def CSRRD : FmtCSR<0x04000000, (outs GPR:$rd), (ins uimm14:$csr_num),
2530:                    "$rd, $csr_num">;
2531: let hasSideEffects = 1, Constraints = "$rd = $dst" in {
2532: def CSRWR : FmtCSR<0x04000020, (outs GPR:$dst),
2533:                    (ins GPR:$rd, uimm14:$csr_num), "$rd, $csr_num">;
2534: def CSRXCHG : FmtCSRXCHG<0x04000000, (outs GPR:$dst),
2535:                          (ins GPR:$rd, GPRNoR0R1:$rj, uimm14:$csr_num),
2536:                          "$rd, $rj, $csr_num">;
2537: } // hasSideEffects = 1, Constraints = "$rd = $dst"
2538: 
2539: // IOCSR Access Instructions
2540: def IOCSRRD_B : IOCSRRD<0x06480000>;
2541: def IOCSRRD_H : IOCSRRD<0x06480400>;
2542: def IOCSRRD_W : IOCSRRD<0x06480800>;
2543: def IOCSRWR_B : IOCSRWR<0x06481000>;
2544: def IOCSRWR_H : IOCSRWR<0x06481400>;
2545: def IOCSRWR_W : IOCSRWR<0x06481800>;
2546: let Predicates = [IsLA64] in {
2547: def IOCSRRD_D : IOCSRRD<0x06480c00>;
2548: def IOCSRWR_D : IOCSRWR<0x06481c00>;
2549: } // Predicates = [IsLA64]
2550: 
2551: // TLB Maintenance Instructions
2552: let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in {
2553: def TLBSRCH  : FmtI32<0x06482800>;
2554: def TLBRD    : FmtI32<0x06482c00>;
2555: def TLBWR    : FmtI32<0x06483000>;
2556: def TLBFILL  : FmtI32<0x06483400>;
2557: def TLBCLR   : FmtI32<0x06482000>;
2558: def TLBFLUSH : FmtI32<0x06482400>;
2559: def INVTLB : FmtINVTLB<(outs), (ins GPR:$rk, GPR:$rj, uimm5:$op),
2560:                        "$op, $rj, $rk">;
```
- **EN**: This block declares or refines TableGen records such as `CSRRD`, `CSRWR`, `CSRXCHG`, `IOCSRRD_B`, `IOCSRRD_H`, `IOCSRRD_W`.
- **CN**: 该代码块声明或细化了 `CSRRD`, `CSRWR`, `CSRXCHG`, `IOCSRRD_B`, `IOCSRRD_H`, `IOCSRRD_W` 等 TableGen 记录。

### Lines 2561-2600 / 第 2561-2600 行
```tablegen
2561: } // hasSideEffects = 1, mayLoad = 0, mayStore = 0
2562: 
2563: // Software Page Walking Instructions
2564: def LDDIR : Fmt2RI8<0x06400000, (outs GPR:$rd),
2565:                     (ins GPR:$rj, uimm8:$imm8), "$rd, $rj, $imm8">;
2566: def LDPTE : FmtLDPTE<(outs), (ins GPR:$rj, uimm8:$seq), "$rj, $seq">;
2567: 
2568: 
2569: // Other Miscellaneous Instructions
2570: let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in
2571: def ERTN : FmtI32<0x06483800>;
2572: def DBCL : MISC_I15<0x002a8000>;
2573: def IDLE : MISC_I15<0x06488000>;
2574: 
2575: //===----------------------------------------------------------------------===//
2576: // Privilege Intrinsics
2577: //===----------------------------------------------------------------------===//
2578: 
2579: def : Pat<(loongarch_csrrd uimm14:$imm14), (CSRRD uimm14:$imm14)>;
2580: def : Pat<(loongarch_csrwr GPR:$rd, uimm14:$imm14),
2581:           (CSRWR GPR:$rd, uimm14:$imm14)>;
2582: def : Pat<(loongarch_csrxchg GPR:$rd, GPRNoR0R1:$rj, uimm14:$imm14),
2583:           (CSRXCHG GPR:$rd, GPRNoR0R1:$rj, uimm14:$imm14)>;
2584: 
2585: def : Pat<(loongarch_iocsrrd_b GPR:$rj), (IOCSRRD_B GPR:$rj)>;
2586: def : Pat<(loongarch_iocsrrd_h GPR:$rj), (IOCSRRD_H GPR:$rj)>;
2587: def : Pat<(loongarch_iocsrrd_w GPR:$rj), (IOCSRRD_W GPR:$rj)>;
2588: 
2589: def : Pat<(loongarch_iocsrwr_b GPR:$rd, GPR:$rj), (IOCSRWR_B GPR:$rd, GPR:$rj)>;
2590: def : Pat<(loongarch_iocsrwr_h GPR:$rd, GPR:$rj), (IOCSRWR_H GPR:$rd, GPR:$rj)>;
2591: def : Pat<(loongarch_iocsrwr_w GPR:$rd, GPR:$rj), (IOCSRWR_W GPR:$rd, GPR:$rj)>;
2592: 
2593: def : Pat<(loongarch_cpucfg GPR:$rj), (CPUCFG GPR:$rj)>;
2594: 
2595: let Predicates = [IsLA64] in {
2596: def : Pat<(loongarch_iocsrrd_d GPR:$rj), (IOCSRRD_D GPR:$rj)>;
2597: def : Pat<(loongarch_iocsrwr_d GPR:$rd, GPR:$rj), (IOCSRWR_D GPR:$rd, GPR:$rj)>;
2598: def : Pat<(int_loongarch_asrtle_d GPR:$rj, GPR:$rk),
2599:           (ASRTLE_D GPR:$rj, GPR:$rk)>;
2600: def : Pat<(int_loongarch_asrtgt_d GPR:$rj, GPR:$rk),
```
- **EN**: This block declares or refines TableGen records such as `LDDIR`, `LDPTE`, `ERTN`, `DBCL`, `IDLE`.
- **CN**: 该代码块声明或细化了 `LDDIR`, `LDPTE`, `ERTN`, `DBCL`, `IDLE` 等 TableGen 记录。

### Lines 2601-2626 / 第 2601-2626 行
```tablegen
2601:           (ASRTGT_D GPR:$rj, GPR:$rk)>;
2602: def : Pat<(int_loongarch_lddir_d GPR:$rj, timm:$imm8),
2603:           (LDDIR GPR:$rj, timm:$imm8)>;
2604: def : Pat<(int_loongarch_ldpte_d GPR:$rj, timm:$imm8),
2605:           (LDPTE GPR:$rj, timm:$imm8)>;
2606: } // Predicates = [IsLA64]
2607: 
2608: //===----------------------------------------------------------------------===//
2609: // LSX Instructions
2610: //===----------------------------------------------------------------------===//
2611: include "LoongArchLSXInstrInfo.td"
2612: 
2613: //===----------------------------------------------------------------------===//
2614: // LASX Instructions
2615: //===----------------------------------------------------------------------===//
2616: include "LoongArchLASXInstrInfo.td"
2617: 
2618: //===----------------------------------------------------------------------===//
2619: // LVZ Instructions
2620: //===----------------------------------------------------------------------===//
2621: include "LoongArchLVZInstrInfo.td"
2622: 
2623: //===----------------------------------------------------------------------===//
2624: // LBT Instructions
2625: //===----------------------------------------------------------------------===//
2626: include "LoongArchLBTInstrInfo.td"
```
- **EN**: It composes TableGen records by including `LoongArchLSXInstrInfo.td`, `LoongArchLASXInstrInfo.td`, `LoongArchLVZInstrInfo.td`, `LoongArchLBTInstrInfo.td` and reusing previously declared backend fragments.
- **CN**: 它通过包含 `LoongArchLSXInstrInfo.td`, `LoongArchLASXInstrInfo.td`, `LoongArchLVZInstrInfo.td`, `LoongArchLBTInstrInfo.td` 组合 TableGen 记录，并复用此前声明的后端片段。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArchInstrFormats.td`
- `LoongArchFloatInstrFormats.td`
- `LoongArchLSXInstrFormats.td`
- `LoongArchLASXInstrFormats.td`
- `LoongArchLBTInstrFormats.td`
- `LoongArchFloat32InstrInfo.td`
- `LoongArchFloat64InstrInfo.td`
- `LoongArchLSXInstrInfo.td`
- `LoongArchLASXInstrInfo.td`
- `LoongArchLVZInstrInfo.td`
- `LoongArchLBTInstrInfo.td`
