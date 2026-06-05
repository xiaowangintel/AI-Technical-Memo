# M68kInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kInstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines M68k backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 M68k 后端元数据；具体而言，它定义或实现目标指令信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: //===-- M68kInstrInfo.td - Main M68k Instruction Definition -*- tablegen -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file describes the M68k instruction set, defining the instructions
  11: /// and properties of the instructions which are needed for code generation,
  12: /// machine code emission, and analysis.
  13: ///
  14: //===----------------------------------------------------------------------===//
  15: 
  16: include "M68kInstrFormats.td"
  17: 
  18: //===----------------------------------------------------------------------===//
  19: // Profiles
  20: //===----------------------------------------------------------------------===//
  21: 
  22: def MxSDT_CallSeqStart : SDCallSeqStart<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
  23: def MxSDT_CallSeqEnd   : SDCallSeqEnd<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
  24: 
  25: def MxSDT_Call    : SDTypeProfile<0, -1, [SDTCisVT<0, iPTR>]>;
  26: 
  27: def MxSDT_Ret     : SDTypeProfile<0, -1, [
  28:   /* ADJ */ SDTCisVT<0, i32>
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It composes TableGen records by including `M68kInstrFormats.td` and reusing previously declared backend fragments. This block declares or refines TableGen records such as `MxSDT_CallSeqStart`, `MxSDT_CallSeqEnd`, `MxSDT_Call`, `MxSDT_Ret`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它通过包含 `M68kInstrFormats.td` 组合 TableGen 记录，并复用此前声明的后端片段。 该代码块声明或细化了 `MxSDT_CallSeqStart`, `MxSDT_CallSeqEnd`, `MxSDT_Call`, `MxSDT_Ret` 等 TableGen 记录。

### Lines 29-56 / 第 29-56 行
```tablegen
  29: ]>;
  30: 
  31: def MxSDT_TCRet   : SDTypeProfile<0, 2, [SDTCisPtrTy<0>, SDTCisVT<1, i32>]>;
  32: 
  33: def MxSDT_Wrapper : SDTypeProfile<1, 1, [SDTCisSameAs<0, 1>, SDTCisPtrTy<0>]>;
  34: 
  35: def MxSDT_UnArithCCROut : SDTypeProfile<2, 1, [
  36:   /* RES */ SDTCisInt<0>,
  37:   /* CCR */ SDTCisVT<1, i8>,
  38:   /* OPD */ SDTCisSameAs<0, 2>
  39: ]>;
  40: 
  41: // RES, CCR <- op LHS, RHS
  42: def MxSDT_BiArithCCROut : SDTypeProfile<2, 2, [
  43:   /* RES */ SDTCisInt<0>,
  44:   /* CCR */ SDTCisVT<1, i8>,
  45:   /* LHS */ SDTCisSameAs<0, 2>,
  46:   /* RHS */ SDTCisSameAs<0, 3>
  47: ]>;
  48: 
  49: // RES, CCR <- op LHS, RHS, CCR
  50: def MxSDT_BiArithCCRInOut : SDTypeProfile<2, 3, [
  51:   /* RES 1 */ SDTCisInt<0>,
  52:   /*   CCR */ SDTCisVT<1, i8>,
  53:   /*   LHS */ SDTCisSameAs<0, 2>,
  54:   /*   RHS */ SDTCisSameAs<0, 3>,
  55:   /*   CCR */ SDTCisSameAs<1, 4>
  56: ]>;
```
- **EN**: This block declares or refines TableGen records such as `MxSDT_TCRet`, `MxSDT_Wrapper`, `MxSDT_UnArithCCROut`, `MxSDT_BiArithCCROut`, `MxSDT_BiArithCCRInOut`.
- **CN**: 该代码块声明或细化了 `MxSDT_TCRet`, `MxSDT_Wrapper`, `MxSDT_UnArithCCROut`, `MxSDT_BiArithCCROut`, `MxSDT_BiArithCCRInOut` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57: 
  58: def MxSDT_CmpTest : SDTypeProfile<1, 2, [
  59:    /* CCR */ SDTCisVT<0, i8>,
  60:    /* Ops */ SDTCisSameAs<1, 2>
  61: ]>;
  62: 
  63: def MxSDT_Cmov : SDTypeProfile<1, 4, [
  64:   /*  ARG */ SDTCisSameAs<0, 1>,
  65:   /*  ARG */ SDTCisSameAs<1, 2>,
  66:   /* Cond */ SDTCisVT<3, i8>,
  67:   /*  CCR */ SDTCisVT<4, i8>
  68: ]>;
  69: 
  70: def MxSDT_BrCond : SDTypeProfile<0, 3, [
  71:   /* Dest */ SDTCisVT<0, OtherVT>,
  72:   /* Cond */ SDTCisVT<1, i8>,
  73:   /*  CCR */ SDTCisVT<2, i8>
  74: ]>;
  75: 
  76: def MxSDT_SetCC : SDTypeProfile<1, 2, [
  77:   /* BOOL */ SDTCisVT<0, i8>,
  78:   /* Cond */ SDTCisVT<1, i8>,
  79:   /*  CCR */ SDTCisVT<2, i8>
  80: ]>;
  81: 
  82: def MxSDT_SetCC_C : SDTypeProfile<1, 2, [
  83:   /* BOOL */ SDTCisInt<0>,
  84:   /* Cond */ SDTCisVT<1, i8>,
```
- **EN**: This block declares or refines TableGen records such as `MxSDT_CmpTest`, `MxSDT_Cmov`, `MxSDT_BrCond`, `MxSDT_SetCC`, `MxSDT_SetCC_C`.
- **CN**: 该代码块声明或细化了 `MxSDT_CmpTest`, `MxSDT_Cmov`, `MxSDT_BrCond`, `MxSDT_SetCC`, `MxSDT_SetCC_C` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85:   /*  CCR */ SDTCisVT<2, i8>
  86: ]>;
  87: 
  88: 
  89: def MxSDT_SEG_ALLOCA : SDTypeProfile<1, 1,[
  90:   /*  MEM */ SDTCisVT<0, iPTR>,
  91:   /* SIZE */ SDTCisVT<1, iPTR>
  92: ]>;
  93: 
  94: 
  95: //===----------------------------------------------------------------------===//
  96: // Nodes
  97: //===----------------------------------------------------------------------===//
  98: 
  99: def MxCallSeqStart : SDNode<"ISD::CALLSEQ_START", MxSDT_CallSeqStart,
 100:                             [SDNPHasChain, SDNPOutGlue]>;
 101: 
 102: def MxCallSeqEnd   : SDNode<"ISD::CALLSEQ_END", MxSDT_CallSeqEnd,
 103:                             [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
 104: 
 105: def MxCall         : SDNode<"M68kISD::CALL", MxSDT_Call,
 106:                             [SDNPHasChain, SDNPOutGlue,
 107:                              SDNPOptInGlue, SDNPVariadic]>;
 108: 
 109: def MxRet   : SDNode<"M68kISD::RET", MxSDT_Ret,
 110:                      [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
 111: 
 112: def MxTCRet : SDNode<"M68kISD::TC_RETURN", MxSDT_TCRet,
```
- **EN**: This block declares or refines TableGen records such as `MxSDT_SEG_ALLOCA`, `MxCallSeqStart`, `MxCallSeqEnd`, `MxCall`, `MxRet`, `MxTCRet`.
- **CN**: 该代码块声明或细化了 `MxSDT_SEG_ALLOCA`, `MxCallSeqStart`, `MxCallSeqEnd`, `MxCall`, `MxRet`, `MxTCRet` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113:                      [SDNPHasChain,  SDNPOptInGlue, SDNPVariadic]>;
 114: 
 115: def MxGlobalBaseReg : SDNode<"M68kISD::GLOBAL_BASE_REG",
 116:                              SDTypeProfile<1, 0, [SDTCisVT<0, iPTR>]>>;
 117: 
 118: // A wrapper node for TargetConstantPool,
 119: // TargetExternalSymbol, and TargetGlobalAddress.
 120: def MxWrapper   : SDNode<"M68kISD::Wrapper",   MxSDT_Wrapper>;
 121: 
 122: // Special wrapper used under M68k PIC mode for PC
 123: // relative displacements.
 124: def MxWrapperPC : SDNode<"M68kISD::WrapperPC", MxSDT_Wrapper>;
 125: 
 126: // Arithmetic operations with CCR results.
 127: def MxAdd  : SDNode<"M68kISD::ADD",  MxSDT_BiArithCCROut, [SDNPCommutative]>;
 128: def MxSub  : SDNode<"M68kISD::SUB",  MxSDT_BiArithCCROut>;
 129: def MxOr   : SDNode<"M68kISD::OR",   MxSDT_BiArithCCROut, [SDNPCommutative]>;
 130: def MxXor  : SDNode<"M68kISD::XOR",  MxSDT_BiArithCCROut, [SDNPCommutative]>;
 131: def MxAnd  : SDNode<"M68kISD::AND",  MxSDT_BiArithCCROut, [SDNPCommutative]>;
 132: 
 133: def MxAddX : SDNode<"M68kISD::ADDX", MxSDT_BiArithCCRInOut>;
 134: def MxSubX : SDNode<"M68kISD::SUBX", MxSDT_BiArithCCRInOut>;
 135: 
 136: def MxSMul : SDNode<"M68kISD::SMUL", MxSDT_BiArithCCROut, [SDNPCommutative]>;
 137: def MxUMul : SDNode<"M68kISD::UMUL", MxSDT_BiArithCCROut, [SDNPCommutative]>;
 138: 
 139: // M68k compare and logical compare instructions. Subtracts the source
 140: // operand from the destination data register and sets the condition
```
- **EN**: This block declares or refines TableGen records such as `MxGlobalBaseReg`, `MxWrapper`, `MxWrapperPC`, `MxAdd`, `MxSub`, `MxOr`.
- **CN**: 该代码块声明或细化了 `MxGlobalBaseReg`, `MxWrapper`, `MxWrapperPC`, `MxAdd`, `MxSub`, `MxOr` 等 TableGen 记录。

### Lines 141-168 / 第 141-168 行
```tablegen
 141: // codes according to the result. Immediate always goes first.
 142: def MxCmp     : SDNode<"M68kISD::CMP", MxSDT_CmpTest>;
 143: 
 144: // M68k bit-test instructions.
 145: def MxBtst    : SDNode<"M68kISD::BTST", MxSDT_CmpTest>;
 146: 
 147: // M68k conditional moves. Operand 0 and operand 1 are the two values
 148: // to select from. Operand 2 is the condition code, and operand 3 is the
 149: // flag operand produced by a CMP or TEST instruction. It also writes a
 150: // flag result.
 151: def MxCmov    : SDNode<"M68kISD::CMOV",        MxSDT_Cmov>;
 152: 
 153: // M68k conditional branches. Operand 0 is the chain operand, operand 1
 154: // is the block to branch if condition is true, operand 2 is the
 155: // condition code, and operand 3 is the flag operand produced by a CMP
 156: // or TEST instruction.
 157: def MxBrCond  : SDNode<"M68kISD::BRCOND",      MxSDT_BrCond, [SDNPHasChain]>;
 158: 
 159: // M68k SetCC. Operand 0 is condition code, and operand 1 is the CCR
 160: // operand, usually produced by a CMP instruction.
 161: def MxSetCC   : SDNode<"M68kISD::SETCC",       MxSDT_SetCC>;
 162: 
 163: // Same as SETCC except it's materialized with a subx and the value is all
 164: // one's or all zero's.
 165: def MxSetCC_C : SDNode<"M68kISD::SETCC_CARRY", MxSDT_SetCC_C>;
 166: 
 167: // For allocating variable amounts of stack space when using
 168: // segmented stacks. Check if the current stacklet has enough space, and
```
- **EN**: This block declares or refines TableGen records such as `MxCmp`, `MxBtst`, `MxCmov`, `MxBrCond`, `MxSetCC`, `MxSetCC_C`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MxCmp`, `MxBtst`, `MxCmov`, `MxBrCond`, `MxSetCC`, `MxSetCC_C` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 169-196 / 第 169-196 行
```tablegen
 169: // falls back to heap allocation if not.
 170: def MxSegAlloca : SDNode<"M68kISD::SEG_ALLOCA", MxSDT_SEG_ALLOCA,
 171:                          [SDNPHasChain]>;
 172: 
 173: 
 174: //===----------------------------------------------------------------------===//
 175: // Operands
 176: //===----------------------------------------------------------------------===//
 177: 
 178: /// Size is the size of the data, either bits of a register or number of bits
 179: /// addressed in memory. Size id is a letter that identifies size.
 180: class MxSize<int num, string id, string full> {
 181:   int Num = num;
 182:   string Id = id;
 183:   string Full = full;
 184: }
 185: 
 186: def MxSize8  : MxSize<8,  "b", "byte">;
 187: def MxSize16 : MxSize<16, "w", "word">;
 188: def MxSize32 : MxSize<32, "l", "long">;
 189: def MxSizeF32 : MxSize<32, "s", "f32">;
 190: def MxSizeF64 : MxSize<64, "d", "f64">;
 191: def MxSizeF80 : MxSize<80, "x", "f80">;
 192: 
 193: class MxOpClass<string name,
 194:                 list<AsmOperandClass> superClasses = []> : AsmOperandClass {
 195:   let Name = name;
 196:   let ParserMethod = "parseMemOp";
```
- **EN**: This block declares or refines TableGen records such as `MxSegAlloca`, `MxSize`, `MxSize8`, `MxSize16`, `MxSize32`, `MxSizeF32`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MxSegAlloca`, `MxSize`, `MxSize8`, `MxSize16`, `MxSize32`, `MxSizeF32` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 197-224 / 第 197-224 行
```tablegen
 197:   let SuperClasses = superClasses;
 198: }
 199: 
 200: def MxRegClass : MxOpClass<"Reg">;
 201: // Splitting asm register class to avoid ambiguous on operands'
 202: // MatchClassKind. For instance, without this separation,
 203: // both ADD32dd and ADD32dr has {MCK_RegClass, MCK_RegClass} for
 204: // their operands, which makes AsmParser unable to pick the correct
 205: // one in a deterministic way.
 206: let RenderMethod = "addRegOperands", SuperClasses = [MxRegClass]in {
 207:   def MxARegClass : MxOpClass<"AReg">;
 208:   def MxDRegClass : MxOpClass<"DReg">;
 209: 
 210:   def MxFPDRegClass : MxOpClass<"FPDReg">;
 211:   def MxFPCRegClass : MxOpClass<"FPCReg">;
 212: }
 213: 
 214: class MxOperand<ValueType vt, MxSize size, string letter, RegisterClass rc, dag pat = (null_frag)> {
 215:   ValueType VT = vt;
 216:   string Letter = letter;
 217:   MxSize Size = size;
 218:   RegisterClass RC = rc;
 219:   dag Pat = pat;
 220: }
 221: 
 222: class MxRegOp<ValueType vt,
 223:               RegisterClass rc,
 224:               MxSize size,
```
- **EN**: This block declares or refines TableGen records such as `MxRegClass`, `MxARegClass`, `MxDRegClass`, `MxFPDRegClass`, `MxFPCRegClass`, `MxOperand`.
- **CN**: 该代码块声明或细化了 `MxRegClass`, `MxARegClass`, `MxDRegClass`, `MxFPDRegClass`, `MxFPCRegClass`, `MxOperand` 等 TableGen 记录。

### Lines 225-252 / 第 225-252 行
```tablegen
 225:               string letter,
 226:               string pm = "printOperand">
 227:     : RegisterOperand<rc, pm>,
 228:       MxOperand<vt, size, letter, rc> {
 229:   let ParserMatchClass = MxRegClass;
 230: }
 231: 
 232: // REGISTER DIRECT. The operand is in the data register specified by
 233: // the effective address register field.
 234: foreach size = [16, 32] in {
 235:   def MxXRD # size : MxRegOp<!cast<ValueType>("i"#size), !cast<MxRegClass>("XR"#size),
 236:                              !cast<MxSize>("MxSize"#size), "r">;
 237:   def MxXRD # size # _TC : MxRegOp<!cast<ValueType>("i"#size), !cast<MxRegClass>("XR"#size#"_TC"),
 238:                                    !cast<MxSize>("MxSize"#size), "r">;
 239: } // foreach size
 240: 
 241: // DATA REGISTER DIRECT. The operand is in the data register specified by
 242: // the effective address register field.
 243: let ParserMatchClass = MxDRegClass in {
 244: foreach size = [8, 16, 32] in {
 245:   def MxDRD # size : MxRegOp<!cast<ValueType>("i"#size), !cast<MxRegClass>("DR"#size),
 246:                              !cast<MxSize>("MxSize"#size), "d">;
 247:   if !gt(size, 8) then
 248:   def MxDRD # size # _TC : MxRegOp<!cast<ValueType>("i"#size), !cast<MxRegClass>("DR"#size#"_TC"),
 249:                                    !cast<MxSize>("MxSize"#size), "d">;
 250: } // foreach size
 251: } // let ParserMatchClass
 252: 
```
- **EN**: This block declares or refines TableGen records such as `MxXRD`, `MxXRD`, `MxDRD`, `MxDRD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxXRD`, `MxXRD`, `MxDRD`, `MxDRD` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 253-280 / 第 253-280 行
```tablegen
 253: // ADDRESS REGISTER DIRECT. The operand is in the address register specified by
 254: // the effective address register field.
 255: let ParserMatchClass = MxARegClass in {
 256: foreach size = [16, 32] in {
 257:   def MxARD # size : MxRegOp<!cast<ValueType>("i"#size), !cast<MxRegClass>("AR"#size),
 258:                              !cast<MxSize>("MxSize"#size), "a">;
 259:   def MxARD # size # _TC : MxRegOp<!cast<ValueType>("i"#size), !cast<MxRegClass>("AR"#size#"_TC"),
 260:                                    !cast<MxSize>("MxSize"#size), "a">;
 261: } // foreach size
 262: } // let ParserMatchClass
 263: 
 264: // FLOATING POINT DATA REGISTER.
 265: let ParserMatchClass = MxFPDRegClass in {
 266:   def MxFPR32 : MxRegOp<f32, FPDR32, MxSizeF32, "fp">;
 267:   def MxFPR64 : MxRegOp<f64, FPDR64, MxSizeF64, "fp">;
 268:   def MxFPR80 : MxRegOp<f80, FPDR80, MxSizeF80, "fp">;
 269: }
 270: 
 271: // FLOATING POINT SYSTEM CONTROL REGISTER
 272: let ParserMatchClass = MxFPCRegClass in {
 273:   def MxFPCSR : MxRegOp<i32, FPCSC, MxSize32, "fpcs">;
 274:   def MxFPIR  : MxRegOp<i32, FPIC,  MxSize32, "fpi">;
 275: }
 276: 
 277: class MxMemOp<dag ops, MxSize size, string letter,
 278:               string printMethod = "printOperand",
 279:               AsmOperandClass parserMatchClass = ImmAsmOperand>
 280:     : Operand<iPTR>, MxOperand<iPTR, size, letter, ?> {
```
- **EN**: This block declares or refines TableGen records such as `MxARD`, `MxARD`, `MxFPR32`, `MxFPR64`, `MxFPR80`, `MxFPCSR`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxARD`, `MxARD`, `MxFPR32`, `MxFPR64`, `MxFPR80`, `MxFPCSR` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 281-308 / 第 281-308 行
```tablegen
 281:   let PrintMethod = printMethod;
 282:   let MIOperandInfo = ops;
 283:   let ParserMatchClass = parserMatchClass;
 284:   let OperandType = "OPERAND_MEMORY";
 285: }
 286: 
 287: // ADDRESS REGISTER INDIRECT. The address of the operand is in the address
 288: // register specified by the register field. The reference is classified as
 289: // a data reference with the exception of the jump and jump-to-subroutine
 290: // instructions.
 291: def MxARI         : MxOpClass<"ARI">;
 292: foreach size = ["8", "16", "32"] in {
 293:   defvar ResSize = !cast<MxSize>("MxSize"#size);
 294:   def MxARI # size       : MxMemOp<(ops AR32),    ResSize, "j", "printARI"#size#"Mem", MxARI>;
 295:   def MxARI # size # _TC : MxMemOp<(ops AR32_TC), ResSize, "j", "printARI"#size#"Mem", MxARI>;
 296: } // foreach size
 297: 
 298: // ADDRESS REGISTER INDIRECT WITH POSTINCREMENT. The address of the operand is
 299: // in the address register specified by the register field. After the operand
 300: // address is used, it is incremented by one, two, or four depending upon whether
 301: // the size of the operand is byte, word, or long word. If the address register
 302: // is the stack pointer and the operand size is byte, the address is incremented
 303: // by two rather than one to keep the stack pointer on a word boundary.
 304: // The reference is classified as a data reference.
 305: def MxARIPI       : MxOpClass<"ARIPI">;
 306: foreach size = ["8", "16", "32"] in {
 307:   defvar ResSize = !cast<MxSize>("MxSize"#size);
 308:   def MxARIPI # size       : MxMemOp<(ops AR32),    ResSize, "o", "printARIPI"#size#"Mem", MxARIPI>;
```
- **EN**: This block declares or refines TableGen records such as `MxARI`, `MxARI`, `MxARI`, `MxARIPI`, `MxARIPI`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxARI`, `MxARI`, `MxARI`, `MxARIPI`, `MxARIPI` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 309-336 / 第 309-336 行
```tablegen
 309:   def MxARIPI # size # _TC : MxMemOp<(ops AR32_TC), ResSize, "o", "printARIPI"#size#"Mem", MxARIPI>;
 310: } // foreach size
 311: 
 312: // ADDRESS REGISTER INDIRECT WITH PREDECREMENT. The address of the operand is in
 313: // the address register specified by the register field. Before the operand
 314: // address is used, it is decremented by one, two, or four depending upon whether
 315: // the operand size is byte, word, or long word. If the address register is
 316: // the stack pointer and the operand size is byte, the address is decremented by
 317: // two rather than one to keep the stack pointer on a word boundary.
 318: // The reference is classified as a data reference.
 319: def MxARIPD       : MxOpClass<"ARIPD">;
 320: foreach size = ["8", "16", "32"] in {
 321:   defvar ResSize = !cast<MxSize>("MxSize"#size);
 322:   def MxARIPD # size       : MxMemOp<(ops AR32),    ResSize, "e", "printARIPD"#size#"Mem", MxARIPD>;
 323:   def MxARIPD # size # _TC : MxMemOp<(ops AR32_TC), ResSize, "e", "printARIPD"#size#"Mem", MxARIPD>;
 324: } // foreach size
 325: 
 326: // ADDRESS REGISTER INDIRECT WITH DISPLACEMENT. This addressing mode requires one
 327: // word of extension. The address of the operand is the sum of the address in
 328: // the address register and the sign-extended 16-bit displacement integer in the
 329: // extension word. The reference is classified as a data reference with the
 330: // exception of the jump and jump-to-subroutine instructions.
 331: def MxARID        : MxOpClass<"ARID">;
 332: foreach size = ["8", "16", "32"] in {
 333:   defvar ResSize = !cast<MxSize>("MxSize"#size);
 334:   def MxARID # size       : MxMemOp<(ops i16imm:$disp, AR32:$reg),    ResSize, "p", "printARID"#size#"Mem", MxARID>;
 335:   def MxARID # size # _TC : MxMemOp<(ops i16imm:$disp, AR32_TC:$reg), ResSize, "p", "printARID"#size#"Mem", MxARID>;
 336: } // foreach size
```
- **EN**: This block declares or refines TableGen records such as `MxARIPI`, `MxARIPD`, `MxARIPD`, `MxARIPD`, `MxARID`, `MxARID`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxARIPI`, `MxARIPD`, `MxARIPD`, `MxARIPD`, `MxARID`, `MxARID` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 337-364 / 第 337-364 行
```tablegen
 337: 
 338: // ADDRESS REGISTER INDIRECT WITH INDEX. This addressing mode requires one word
 339: // of extension. The address of the operand is the sum of the address in the
 340: // address register, the signextended displacement integer in the low order eight
 341: // bits of the extension word, and the contents of the index register.
 342: // The reference is classified as a data reference with the exception of the
 343: // jump and jump-to-subroutine instructions
 344: def MxARII       : MxOpClass<"ARII">;
 345: foreach size = ["8", "16", "32"] in {
 346:   defvar ResSize = !cast<MxSize>("MxSize"#size);
 347:   def MxARII # size       : MxMemOp<(ops i8imm:$disp, AR32:$reg,    XR32:$index), ResSize, "f", "printARII"#size#"Mem", MxARII>;
 348:   def MxARII # size # _TC : MxMemOp<(ops i8imm:$disp, AR32_TC:$reg, XR32:$index), ResSize, "f", "printARII"#size#"Mem", MxARII>;
 349: } // foreach size
 350: 
 351: // ABSOLUTE SHORT ADDRESS. This addressing mode requires one word of extension.
 352: // The address of the operand is the extension word. The 16-bit address is sign
 353: // extended before it is used.  The reference is classified as a data reference
 354: // with the exception of the jump and jump-tosubroutine instructions.
 355: def MxAddr     : MxOpClass<"Addr">;
 356: let RenderMethod = "addAddrOperands" in {
 357:   // This hierarchy ensures Addr8 will always be parsed
 358:   // before other larger-width variants.
 359:   def MxAddr32   : MxOpClass<"Addr32", [MxAddr]>;
 360:   def MxAddr16   : MxOpClass<"Addr16", [MxAddr32]>;
 361:   def MxAddr8    : MxOpClass<"Addr8",  [MxAddr16]>;
 362: }
 363: 
 364: def MxAS8      : MxMemOp<(ops OtherVT), MxSize8,  "B", "printAS8Mem",  MxAddr8>;
```
- **EN**: This block declares or refines TableGen records such as `MxARII`, `MxARII`, `MxARII`, `MxAddr`, `MxAddr32`, `MxAddr16`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxARII`, `MxARII`, `MxARII`, `MxAddr`, `MxAddr32`, `MxAddr16` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 365-392 / 第 365-392 行
```tablegen
 365: def MxAS16     : MxMemOp<(ops OtherVT), MxSize16, "B", "printAS16Mem", MxAddr16>;
 366: def MxAS32     : MxMemOp<(ops OtherVT), MxSize32, "B", "printAS32Mem", MxAddr32>;
 367: 
 368: // ABSOLUTE LONG ADDRESS. This addressing mode requires two words of extension.
 369: // The address of the operand is developed by the concatenation of the extension
 370: // words. The high order part of the address is the first extension word; the low
 371: // order part of the address is the second extension word. The reference is
 372: // classified as a data reference with the exception of the jump and jump
 373: // to-subroutine instructions.
 374: def MxAL8      : MxMemOp<(ops OtherVT), MxSize8,  "b", "printAL8Mem",  MxAddr8>;
 375: def MxAL16     : MxMemOp<(ops OtherVT), MxSize16, "b", "printAL16Mem", MxAddr16>;
 376: def MxAL32     : MxMemOp<(ops OtherVT), MxSize32, "b", "printAL32Mem", MxAddr32>;
 377: 
 378: def MxPCD : MxOpClass<"PCD">;
 379: def MxPCI : MxOpClass<"PCI">;
 380: 
 381: let OperandType = "OPERAND_PCREL" in {
 382: foreach size = ["8", "16", "32"] in {
 383: defvar ResSize = !cast<MxSize>("MxSize"#size);
 384: // PROGRAM COUNTER WITH DISPLACEMENT. This addressing mode requires one word of
 385: // extension. The address of the operand is the sum of the address in the program
 386: // counter and the Sign-extended 16-bit displacement integer in the extension
 387: // word. The value in the program counter is the address of the extension word.
 388: // The reference is classified as a program reference.
 389: def MxPCD # size : MxMemOp<(ops i16imm), ResSize,  "q", "printPCD"#size#"Mem", MxPCD>;
 390: 
 391: // PROGRAM COUNTER WITH INDEX. This addressing mode requires one word of
 392: // extension. The address is the sum of the address in the program counter, the
```
- **EN**: This block declares or refines TableGen records such as `MxAS16`, `MxAS32`, `MxAL8`, `MxAL16`, `MxAL32`, `MxPCD`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxAS16`, `MxAS32`, `MxAL8`, `MxAL16`, `MxAL32`, `MxPCD` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 393-420 / 第 393-420 行
```tablegen
 393: // sign-extended displacement integer in the lower eight bits of the extension
 394: // word, and the contents of the index register.  The value in the program
 395: // counter is the address of the extension word. This reference is classified as
 396: // a program reference.
 397: def MxPCI # size : MxMemOp<(ops i8imm:$disp, XR32:$index), ResSize,  "k", "printPCI"#size#"Mem", MxPCI>;
 398: } // foreach size
 399: } // OPERAND_PCREL
 400: 
 401: def MxImm : AsmOperandClass {
 402:   let Name = "MxImm";
 403:   let PredicateMethod = "isImm";
 404:   let RenderMethod = "addImmOperands";
 405:   let ParserMethod = "parseImm";
 406: }
 407: 
 408: class MxOp<ValueType vt, MxSize size, string letter>
 409:     : Operand<vt>,
 410:       MxOperand<vt, size, letter, ?> {
 411:   let ParserMatchClass = MxImm;
 412: }
 413: 
 414: let OperandType = "OPERAND_IMMEDIATE",
 415:     PrintMethod = "printImmediate" in {
 416: // IMMEDIATE DATA. This addressing mode requires either one or two words of
 417: // extension depending on the size of the operation.
 418: //     Byte Operation - operand is low order byte of extension word
 419: //     Word Operation - operand is extension word
 420: //     Long Word Operation - operand is in the two extension words,
```
- **EN**: This block declares or refines TableGen records such as `MxPCI`, `MxImm`, `MxOp`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxPCI`, `MxImm`, `MxOp` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 421-448 / 第 421-448 行
```tablegen
 421: //                           high order 16 bits are in the first
 422: //                           extension word, low order 16 bits are
 423: //                           in the second extension word.
 424: def Mxi8imm  : MxOp<i8,  MxSize8,  "i">;
 425: def Mxi16imm : MxOp<i16, MxSize16, "i">;
 426: def Mxi32imm : MxOp<i32, MxSize32, "i">;
 427: } // OPERAND_IMMEDIATE
 428: 
 429: class MxBrTargetOperand<int N> : Operand<OtherVT> {
 430:   let OperandType = "OPERAND_PCREL";
 431:   let PrintMethod = "printPCRelImm";
 432:   let ParserMatchClass = !cast<AsmOperandClass>("MxAddr"#N);
 433: }
 434: // Branch targets have OtherVT type and print as pc-relative values.
 435: def MxBrTarget8  : MxBrTargetOperand<8>;
 436: def MxBrTarget16 : MxBrTargetOperand<16>;
 437: def MxBrTarget32 : MxBrTargetOperand<32>;
 438: 
 439: // Used with MOVEM
 440: def MxMoveMaskClass : MxOpClass<"MoveMask">;
 441: class MxMoveMaskOp : MxOp<i16, MxSize16, "m"> {
 442:   let OperandType = "OPERAND_IMMEDIATE";
 443:   let PrintMethod = "printMoveMask";
 444:   let ParserMatchClass = MxMoveMaskClass;
 445: }
 446: 
 447: def MxMoveMask : MxMoveMaskOp;
 448: // The encoding of mask is reversed when the memory operand has an addressing
```
- **EN**: This block declares or refines TableGen records such as `Mxi8imm`, `Mxi16imm`, `Mxi32imm`, `MxBrTargetOperand`, `MxBrTarget8`, `MxBrTarget16`.
- **CN**: 该代码块声明或细化了 `Mxi8imm`, `Mxi16imm`, `Mxi32imm`, `MxBrTargetOperand`, `MxBrTarget8`, `MxBrTarget16` 等 TableGen 记录。

### Lines 449-476 / 第 449-476 行
```tablegen
 449: // mode of 'e', that is, pre-decrement.
 450: def MxInverseMoveMask : MxMoveMaskOp {
 451:   let EncoderMethod = "encodeInverseMoveMask";
 452: }
 453: 
 454: //===----------------------------------------------------------------------===//
 455: // Predicates
 456: //===----------------------------------------------------------------------===//
 457: 
 458: def SmallCode    : Predicate<"TM.getCodeModel() == CodeModel::Small">;
 459: def KernelCode   : Predicate<"TM.getCodeModel() == CodeModel::Kernel">;
 460: def FarData      : Predicate<"TM.getCodeModel() != CodeModel::Small &&"
 461:                              "TM.getCodeModel() != CodeModel::Kernel">;
 462: def NearData     : Predicate<"TM.getCodeModel() == CodeModel::Small ||"
 463:                              "TM.getCodeModel() == CodeModel::Kernel">;
 464: def IsPIC        : Predicate<"TM.isPositionIndependent()">;
 465: def IsNotPIC     : Predicate<"!TM.isPositionIndependent()">;
 466: 
 467: // ISA versions
 468: foreach i = [0,1,2,4,6] in
 469: def AtLeastM680 # i # "0" : Predicate<"Subtarget->atLeastM680"#i#"0()">,
 470:                             AssemblerPredicate<(all_of
 471:                                                 !cast<SubtargetFeature>("FeatureISA"#i#"0"))>;
 472: def AtLeastM68881 : Predicate<"Subtarget->atLeastM68881()">,
 473:                     AssemblerPredicate<(all_of FeatureISA881)>;
 474: def AtLeastM68882 : Predicate<"Subtarget->atLeastM68882()">,
 475:                     AssemblerPredicate<(all_of FeatureISA882)>;
 476: 
```
- **EN**: This block declares or refines TableGen records such as `MxInverseMoveMask`, `SmallCode`, `KernelCode`, `FarData`, `NearData`, `IsPIC`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxInverseMoveMask`, `SmallCode`, `KernelCode`, `FarData`, `NearData`, `IsPIC` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 477-504 / 第 477-504 行
```tablegen
 477: //===----------------------------------------------------------------------===//
 478: // Condition Codes
 479: //
 480: // These MUST be kept in sync with codes enum in M68kInstrInfo.h
 481: //===----------------------------------------------------------------------===//
 482: 
 483: def MxCONDt   : PatLeaf<(i8 0)>;  // True
 484: def MxCONDf   : PatLeaf<(i8 1)>;  // False
 485: def MxCONDhi  : PatLeaf<(i8 2)>;  // High
 486: def MxCONDls  : PatLeaf<(i8 3)>;  // Less or Same
 487: def MxCONDcc  : PatLeaf<(i8 4)>;  // Carry Clear
 488: def MxCONDcs  : PatLeaf<(i8 5)>;  // Carry Set
 489: def MxCONDne  : PatLeaf<(i8 6)>;  // Not Equal
 490: def MxCONDeq  : PatLeaf<(i8 7)>;  // Equal
 491: def MxCONDvc  : PatLeaf<(i8 8)>;  // Overflow Clear
 492: def MxCONDvs  : PatLeaf<(i8 9)>;  // Overflow Set
 493: def MxCONDpl  : PatLeaf<(i8 10)>; // Plus
 494: def MxCONDmi  : PatLeaf<(i8 11)>; // Minus
 495: def MxCONDge  : PatLeaf<(i8 12)>; // Greater or Equal
 496: def MxCONDlt  : PatLeaf<(i8 13)>; // Less Than
 497: def MxCONDgt  : PatLeaf<(i8 14)>; // Greater Than
 498: def MxCONDle  : PatLeaf<(i8 15)>; // Less or Equal
 499: 
 500: 
 501: //===----------------------------------------------------------------------===//
 502: // Complex Patterns
 503: //===----------------------------------------------------------------------===//
 504: 
```
- **EN**: This block declares or refines TableGen records such as `MxCONDt`, `MxCONDf`, `MxCONDhi`, `MxCONDls`, `MxCONDcc`, `MxCONDcs`.
- **CN**: 该代码块声明或细化了 `MxCONDt`, `MxCONDf`, `MxCONDhi`, `MxCONDls`, `MxCONDcc`, `MxCONDcs` 等 TableGen 记录。

### Lines 505-532 / 第 505-532 行
```tablegen
 505: // NOTE Though this CP is not strictly necessarily it will simplify instruciton
 506: // definitions
 507: let WantsParent = true in {
 508:   def MxCP_ARI   : ComplexPattern<iPTR, 1, "SelectARI">;
 509: 
 510:   def MxCP_ARIPI : ComplexPattern<iPTR, 1, "SelectARIPI">;
 511: 
 512:   def MxCP_ARIPD : ComplexPattern<iPTR, 1, "SelectARIPD">;
 513: 
 514:   def MxCP_ARID  : ComplexPattern<iPTR, 2, "SelectARID",
 515:                                   [add, sub, mul, or, shl, frameindex]>;
 516: 
 517:   def MxCP_ARII  : ComplexPattern<iPTR, 3, "SelectARII",
 518:                                   [add, sub, mul, or, shl, frameindex]>;
 519: 
 520:   def MxCP_AL    : ComplexPattern<iPTR, 1, "SelectAL",
 521:                                   [add, sub, mul, or, shl]>;
 522: 
 523:   def MxCP_PCD   : ComplexPattern<iPTR, 1, "SelectPCD",
 524:                                   [add, sub, mul, or, shl]>;
 525: 
 526:   def MxCP_PCI   : ComplexPattern<iPTR, 2, "SelectPCI",
 527:                                   [add, sub, mul, or, shl]>;
 528: }
 529: 
 530: //===----------------------------------------------------------------------===//
 531: // Pattern Fragments
 532: //===----------------------------------------------------------------------===//
```
- **EN**: This block declares or refines TableGen records such as `MxCP_ARI`, `MxCP_ARIPI`, `MxCP_ARIPD`, `MxCP_ARID`, `MxCP_ARII`, `MxCP_AL`.
- **CN**: 该代码块声明或细化了 `MxCP_ARI`, `MxCP_ARIPI`, `MxCP_ARIPD`, `MxCP_ARID`, `MxCP_ARII`, `MxCP_AL` 等 TableGen 记录。

### Lines 533-560 / 第 533-560 行
```tablegen
 533: 
 534: def Mxi8immSExt8  : PatLeaf<(i8  imm)>;
 535: def MximmSExt8    : PatLeaf<(imm), [{ return isInt<8>(N->getSExtValue()); }]>;
 536: 
 537: def Mxi16immSExt16 : PatLeaf<(i16 imm)>;
 538: def MximmSExt16    : PatLeaf<(imm), [{ return isInt<16>(N->getSExtValue()); }]>;
 539: 
 540: def Mxi32immSExt32 : PatLeaf<(i32 imm)>;
 541: def MximmSExt32    : PatLeaf<(imm), [{ return isInt<32>(N->getSExtValue()); }]>;
 542: 
 543: // Used for Shifts and Rotations, since M68k immediates in these instructions
 544: // are 1 <= i <= 8. Generally, if immediate is bigger than 8 it will be moved
 545: // to a register and then an operation is performed.
 546: //
 547: // TODO Need to evaluate whether splitting one big shift(or rotate)
 548: // into a few smaller is faster than doing a move, if so do custom lowering
 549: def Mximm8_1to8   : ImmLeaf<i8,  [{ return Imm >= 1 && Imm <= 8; }]>;
 550: def Mximm16_1to8  : ImmLeaf<i16, [{ return Imm >= 1 && Imm <= 8; }]>;
 551: def Mximm32_1to8  : ImmLeaf<i32, [{ return Imm >= 1 && Imm <= 8; }]>;
 552: 
 553: // Helper fragments for loads.
 554: // It's always safe to treat a anyext i16 load as a i32 load if the i16 is
 555: // known to be 32-bit aligned or better. Ditto for i8 to i16.
 556: def Mxloadi16 : PatFrag<(ops node:$ptr), (i16 (unindexedload node:$ptr)), [{
 557:   LoadSDNode *LD = cast<LoadSDNode>(N);
 558:   ISD::LoadExtType ExtType = LD->getExtensionType();
 559:   if (ExtType == ISD::NON_EXTLOAD)
 560:     return true;
```
- **EN**: This block declares or refines TableGen records such as `Mxi8immSExt8`, `MximmSExt8`, `Mxi16immSExt16`, `MximmSExt16`, `Mxi32immSExt32`, `MximmSExt32`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `Mxi8immSExt8`, `MximmSExt8`, `Mxi16immSExt16`, `MximmSExt16`, `Mxi32immSExt32`, `MximmSExt32` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 561-588 / 第 561-588 行
```tablegen
 561:   if (ExtType == ISD::EXTLOAD)
 562:     return LD->getAlign() >= 2 && !LD->isSimple();
 563:   return false;
 564: }]>;
 565: 
 566: def Mxloadi32 : PatFrag<(ops node:$ptr), (i32 (unindexedload node:$ptr)), [{
 567:   LoadSDNode *LD = cast<LoadSDNode>(N);
 568:   ISD::LoadExtType ExtType = LD->getExtensionType();
 569:   if (ExtType == ISD::NON_EXTLOAD)
 570:     return true;
 571:   if (ExtType == ISD::EXTLOAD)
 572:     return LD->getAlign() >= 4 && !LD->isSimple();
 573:   return false;
 574: }]>;
 575: 
 576: def Mxstoreloadi8_safe : PatFrag<(ops node:$dst, node:$src),
 577:                                  (store (i8 (unindexedload node:$src)),
 578:                                         node:$dst), [{ return isSafeStoreLoad(N); }]>;
 579: 
 580: def Mxstoreloadi16_safe : PatFrag<(ops node:$dst, node:$src),
 581:                                   (store (i16 (unindexedload node:$src)),
 582:                                          node:$dst), [{ return isSafeStoreLoad(N); }]>;
 583: 
 584: def Mxstoreloadi32_safe : PatFrag<(ops node:$dst, node:$src),
 585:                                   (store (i32 (unindexedload node:$src)),
 586:                                          node:$dst),[{ return isSafeStoreLoad(N); }]>;
 587: 
 588: def Mxloadi8         : PatFrag<(ops node:$ptr), (i8  (load node:$ptr))>;
```
- **EN**: This block declares or refines TableGen records such as `Mxloadi32`, `Mxstoreloadi8_safe`, `Mxstoreloadi16_safe`, `Mxstoreloadi32_safe`, `Mxloadi8`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `Mxloadi32`, `Mxstoreloadi8_safe`, `Mxstoreloadi16_safe`, `Mxstoreloadi32_safe`, `Mxloadi8` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 589-616 / 第 589-616 行
```tablegen
 589: 
 590: def MxSExtLoadi16i8  : PatFrag<(ops node:$ptr), (i16 (sextloadi8 node:$ptr))>;
 591: def MxSExtLoadi32i8  : PatFrag<(ops node:$ptr), (i32 (sextloadi8 node:$ptr))>;
 592: def MxSExtLoadi32i16 : PatFrag<(ops node:$ptr), (i32 (sextloadi16 node:$ptr))>;
 593: 
 594: def MxZExtLoadi8i1   : PatFrag<(ops node:$ptr), (i8  (zextloadi1 node:$ptr))>;
 595: def MxZExtLoadi16i1  : PatFrag<(ops node:$ptr), (i16 (zextloadi1 node:$ptr))>;
 596: def MxZExtLoadi32i1  : PatFrag<(ops node:$ptr), (i32 (zextloadi1 node:$ptr))>;
 597: def MxZExtLoadi16i8  : PatFrag<(ops node:$ptr), (i16 (zextloadi8 node:$ptr))>;
 598: def MxZExtLoadi32i8  : PatFrag<(ops node:$ptr), (i32 (zextloadi8 node:$ptr))>;
 599: def MxZExtLoadi32i16 : PatFrag<(ops node:$ptr), (i32 (zextloadi16 node:$ptr))>;
 600: 
 601: def MxExtLoadi8i1    : PatFrag<(ops node:$ptr), (i8  (extloadi1 node:$ptr))>;
 602: def MxExtLoadi16i1   : PatFrag<(ops node:$ptr), (i16 (extloadi1 node:$ptr))>;
 603: def MxExtLoadi32i1   : PatFrag<(ops node:$ptr), (i32 (extloadi1 node:$ptr))>;
 604: def MxExtLoadi16i8   : PatFrag<(ops node:$ptr), (i16 (extloadi8 node:$ptr))>;
 605: def MxExtLoadi32i8   : PatFrag<(ops node:$ptr), (i32 (extloadi8 node:$ptr))>;
 606: def MxExtLoadi32i16  : PatFrag<(ops node:$ptr), (i32 (extloadi16 node:$ptr))>;
 607: 
 608: 
 609: //===----------------------------------------------------------------------===//
 610: // Type Fixtures
 611: //
 612: // Type Fixtures are ValueType related information sets that usually go together
 613: //===----------------------------------------------------------------------===//
 614: 
 615: // TODO make it folded like MxType8.F.Op nad MxType8.F.Pat
 616: // TODO move strings into META subclass
```
- **EN**: This block declares or refines TableGen records such as `MxSExtLoadi16i8`, `MxSExtLoadi32i8`, `MxSExtLoadi32i16`, `MxZExtLoadi8i1`, `MxZExtLoadi16i1`, `MxZExtLoadi32i1`.
- **CN**: 该代码块声明或细化了 `MxSExtLoadi16i8`, `MxSExtLoadi32i8`, `MxSExtLoadi32i16`, `MxZExtLoadi8i1`, `MxZExtLoadi16i1`, `MxZExtLoadi32i1` 等 TableGen 记录。

### Lines 617-644 / 第 617-644 行
```tablegen
 617: // vt: Type of data this fixture refers to
 618: // prefix: Prefix used to identify type
 619: // postfix: Prefix used to qualify type
 620: class MxType<ValueType vt, string prefix, string postfix,
 621:              // rLet: Register letter
 622:              // rOp:  Supported any register operand
 623:              string rLet, MxOperand rOp,
 624:              // jOp:  Supported ARI operand
 625:              // jPat: What ARI pattern to use
 626:              MxOperand jOp, ComplexPattern jPat,
 627:              // oOp:  Supported ARIPI operand
 628:              // oPat: What ARIPI pattern is used
 629:              MxOperand oOp, ComplexPattern oPat,
 630:              // eOp:  Supported ARIPD operand
 631:              // ePat: What ARIPD pattern is used
 632:              MxOperand eOp, ComplexPattern ePat,
 633:              // pOp:  Supported ARID operand
 634:              // pPat: What ARID pattern is used
 635:              MxOperand pOp, ComplexPattern pPat,
 636:              // fOp:  Supported ARII operand
 637:              // fPat: What ARII pattern is used
 638:              MxOperand fOp, ComplexPattern fPat,
 639:              // bOp:  Supported absolute operand
 640:              // bPat: What absolute pattern is used
 641:              MxOperand bOp, ComplexPattern bPat,
 642:              // qOp:  Supported PCD operand
 643:              // qPat: What PCD pattern is used
 644:              MxOperand qOp, ComplexPattern qPat,
```
- **EN**: This block declares or refines TableGen records such as `MxType`.
- **CN**: 该代码块声明或细化了 `MxType` 等 TableGen 记录。

### Lines 645-672 / 第 645-672 行
```tablegen
 645:              // kOp:  Supported PCI operand
 646:              // kPat: What PCI pattern is used
 647:              MxOperand kOp, ComplexPattern kPat,
 648:              // iOp:  Supported immediate operand
 649:              // iPat: What immediate pattern is used
 650:              MxOperand iOp, PatFrag iPat,
 651:              // load: What load operation is used with MEM
 652:              PatFrag load> {
 653:   int Size = vt.Size;
 654:   ValueType VT = vt;
 655:   string Prefix = prefix;
 656:   string Postfix = postfix;
 657: 
 658:   string RLet = rLet;
 659:   MxOperand ROp = rOp;
 660: 
 661:   MxOperand JOp = jOp;
 662:   ComplexPattern JPat = jPat;
 663: 
 664:   MxOperand OOp = oOp;
 665:   ComplexPattern OPat = oPat;
 666: 
 667:   MxOperand EOp = eOp;
 668:   ComplexPattern EPat = ePat;
 669: 
 670:   MxOperand POp = pOp;
 671:   ComplexPattern PPat = pPat;
 672: 
```
- **EN**: This span continues the file's main responsibility: defines M68k backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 673-700 / 第 673-700 行
```tablegen
 673:   MxOperand FOp = fOp;
 674:   ComplexPattern FPat = fPat;
 675: 
 676:   MxOperand BOp = bOp;
 677:   ComplexPattern BPat = bPat;
 678: 
 679:   MxOperand QOp = qOp;
 680:   ComplexPattern QPat = qPat;
 681: 
 682:   MxOperand KOp = kOp;
 683:   ComplexPattern KPat = kPat;
 684: 
 685:   MxOperand IOp = iOp;
 686:   PatFrag IPat = iPat;
 687: 
 688:   PatFrag Load = load;
 689: }
 690: 
 691: // Provides an alternative way to access the MxOperand and
 692: // patterns w.r.t a specific addressing mode.
 693: class MxOpBundle<int size, MxOperand op, ComplexPattern pat> {
 694:   int Size = size;
 695:   MxOperand Op = op;
 696:   ComplexPattern Pat = pat;
 697: }
 698: 
 699: class MxImmOpBundle<int size, MxOperand op, PatFrag pat>
 700:   : MxOpBundle<size, op, ?> {
```
- **EN**: This block declares or refines TableGen records such as `MxOpBundle`, `MxImmOpBundle`.
- **CN**: 该代码块声明或细化了 `MxOpBundle`, `MxImmOpBundle` 等 TableGen 记录。

### Lines 701-728 / 第 701-728 行
```tablegen
 701:   PatFrag ImmPat = pat;
 702: }
 703: 
 704: // TODO: We can use MxOp<S>AddrMode_<AM> in more places to
 705: // replace MxType-based operand factoring.
 706: foreach size = [8, 16, 32] in {
 707:   // Dn
 708:   def MxOp#size#AddrMode_d
 709:     : MxOpBundle<size, !cast<MxOperand>("MxDRD"#size), ?>;
 710: 
 711:   // (An)
 712:   def MxOp#size#AddrMode_j
 713:     : MxOpBundle<size, !cast<MxOperand>("MxARI"#size), MxCP_ARI>;
 714: 
 715:   // (An)+
 716:   def MxOp#size#AddrMode_o
 717:     : MxOpBundle<size, !cast<MxOperand>("MxARIPI"#size), MxCP_ARIPI>;
 718: 
 719:   // -(An)
 720:   def MxOp#size#AddrMode_e
 721:     : MxOpBundle<size, !cast<MxOperand>("MxARIPD"#size), MxCP_ARIPD>;
 722: 
 723:   // (i,An)
 724:   def MxOp#size#AddrMode_p
 725:     : MxOpBundle<size, !cast<MxOperand>("MxARID"#size), MxCP_ARID>;
 726: 
 727:   // (i,An,Xn)
 728:   def MxOp#size#AddrMode_f
```
- **EN**: This block declares or refines TableGen records such as `MxOp`, `MxOp`, `MxOp`, `MxOp`, `MxOp`, `MxOp`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxOp`, `MxOp`, `MxOp`, `MxOp`, `MxOp`, `MxOp` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 729-756 / 第 729-756 行
```tablegen
 729:     : MxOpBundle<size, !cast<MxOperand>("MxARII"#size), MxCP_ARII>;
 730: 
 731:   // (ABS).L
 732:   def MxOp#size#AddrMode_b
 733:     : MxOpBundle<size, !cast<MxOperand>("MxAL"#size), MxCP_AL>;
 734: 
 735:   // (i,PC)
 736:   def MxOp#size#AddrMode_q
 737:     : MxOpBundle<size, !cast<MxOperand>("MxPCD"#size), MxCP_PCD>;
 738: 
 739:   // (i,PC,Xn)
 740:   def MxOp#size#AddrMode_k
 741:     : MxOpBundle<size, !cast<MxOperand>("MxPCI"#size), MxCP_PCI>;
 742: 
 743:   // #imm
 744:   def MxOp#size#AddrMode_i
 745:     : MxImmOpBundle<size, !cast<MxOperand>("Mxi"#size#"imm"),
 746:                     !cast<PatFrag>("Mxi"#size#"immSExt"#size)>;
 747: } // foreach size = [8, 16, 32]
 748: 
 749: foreach size = [16, 32] in {
 750:   // An
 751:   def MxOp#size#AddrMode_a
 752:     : MxOpBundle<size, !cast<MxOperand>("MxARD"#size), ?>;
 753: 
 754:   // Xn
 755:   def MxOp#size#AddrMode_r
 756:     : MxOpBundle<size, !cast<MxOperand>("MxXRD"#size), ?>;
```
- **EN**: This block declares or refines TableGen records such as `MxOp`, `MxOp`, `MxOp`, `MxOp`, `MxOp`, `MxOp`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxOp`, `MxOp`, `MxOp`, `MxOp`, `MxOp`, `MxOp` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 757-784 / 第 757-784 行
```tablegen
 757: } // foreach size = [16, 32]
 758: 
 759: foreach size = [32, 64, 80] in
 760: def MxOp#size#AddrMode_fpr
 761:   : MxOpBundle<size, !cast<MxOperand>("MxFPR"#size), ?>;
 762: 
 763: def MxOp32AddrMode_fpcs : MxOpBundle<32, MxFPCSR, ?>;
 764: def MxOp32AddrMode_fpi  : MxOpBundle<32, MxFPIR, ?>;
 765: 
 766: class MxType8Class<string rLet, MxOperand reg>
 767:     : MxType<i8, "b", "", rLet, reg,
 768:              MxARI8,   MxCP_ARI,
 769:              MxARIPI8, MxCP_ARIPI,
 770:              MxARIPD8, MxCP_ARIPD,
 771:              MxARID8,  MxCP_ARID,
 772:              MxARII8,  MxCP_ARII,
 773:              MxAL8,    MxCP_AL,
 774:              MxPCD8,   MxCP_PCD,
 775:              MxPCI8,   MxCP_PCI,
 776:              Mxi8imm,  Mxi8immSExt8,
 777:              Mxloadi8>;
 778: 
 779: def MxType8 : MxType8Class<?,?>;
 780: 
 781: class MxType16Class<string rLet, MxOperand reg>
 782:     : MxType<i16, "w", "", rLet, reg,
 783:              MxARI16,   MxCP_ARI,
 784:              MxARIPI16, MxCP_ARIPI,
```
- **EN**: This block declares or refines TableGen records such as `MxOp`, `MxOp32AddrMode_fpcs`, `MxOp32AddrMode_fpi`, `MxType8Class`, `MxType8`, `MxType16Class`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxOp`, `MxOp32AddrMode_fpcs`, `MxOp32AddrMode_fpi`, `MxType8Class`, `MxType8`, `MxType16Class` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 785-812 / 第 785-812 行
```tablegen
 785:              MxARIPD16, MxCP_ARIPD,
 786:              MxARID16,  MxCP_ARID,
 787:              MxARII16,  MxCP_ARII,
 788:              MxAL16,    MxCP_AL,
 789:              MxPCD16,   MxCP_PCD,
 790:              MxPCI16,   MxCP_PCI,
 791:              Mxi16imm,  Mxi16immSExt16,
 792:              Mxloadi16>;
 793: 
 794: def MxType16 : MxType16Class<?,?>;
 795: 
 796: class MxType32Class<string rLet, MxOperand reg>
 797:     : MxType<i32, "l", "", rLet, reg,
 798:              MxARI32,   MxCP_ARI,
 799:              MxARIPI32, MxCP_ARIPI,
 800:              MxARIPD32, MxCP_ARIPD,
 801:              MxARID32,  MxCP_ARID,
 802:              MxARII32,  MxCP_ARII,
 803:              MxAL32,    MxCP_AL,
 804:              MxPCD32,   MxCP_PCD,
 805:              MxPCI32,   MxCP_PCI,
 806:              Mxi32imm,  Mxi32immSExt32,
 807:              Mxloadi32>;
 808: 
 809: def MxType32 : MxType32Class<?,?>;
 810: 
 811: 
 812: def MxType8d : MxType8Class<"d", MxDRD8>;
```
- **EN**: This block declares or refines TableGen records such as `MxType16`, `MxType32Class`, `MxType32`, `MxType8d`.
- **CN**: 该代码块声明或细化了 `MxType16`, `MxType32Class`, `MxType32`, `MxType8d` 等 TableGen 记录。

### Lines 813-840 / 第 813-840 行
```tablegen
 813: 
 814: def MxType16d : MxType16Class<"d", MxDRD16>;
 815: def MxType16a : MxType16Class<"a", MxARD16>;
 816: def MxType16r : MxType16Class<"r", MxXRD16>;
 817: def MxType32d : MxType32Class<"d", MxDRD32>;
 818: def MxType32a : MxType32Class<"a", MxARD32>;
 819: def MxType32r : MxType32Class<"r", MxXRD32>;
 820: 
 821: let Postfix = "_TC" in {
 822: def MxType16d_TC : MxType16Class<"d", MxDRD16_TC>;
 823: def MxType16a_TC : MxType16Class<"a", MxARD16_TC>;
 824: def MxType16r_TC : MxType16Class<"r", MxXRD16_TC>;
 825: def MxType32d_TC : MxType32Class<"d", MxDRD32_TC>;
 826: def MxType32a_TC : MxType32Class<"a", MxARD32_TC>;
 827: def MxType32r_TC : MxType32Class<"r", MxXRD32_TC>;
 828: }
 829: 
 830: 
 831: //===----------------------------------------------------------------------===//
 832: // Subsystems
 833: //===----------------------------------------------------------------------===//
 834: 
 835: include "M68kInstrData.td"
 836: include "M68kInstrShiftRotate.td"
 837: include "M68kInstrBits.td"
 838: include "M68kInstrArithmetic.td"
 839: include "M68kInstrControl.td"
 840: include "M68kInstrAtomics.td"
```
- **EN**: It composes TableGen records by including `M68kInstrData.td`, `M68kInstrShiftRotate.td`, `M68kInstrBits.td`, `M68kInstrArithmetic.td`, `M68kInstrControl.td`, `M68kInstrAtomics.td` and reusing previously declared backend fragments. This block declares or refines TableGen records such as `MxType16d`, `MxType16a`, `MxType16r`, `MxType32d`, `MxType32a`, `MxType32r`.
- **CN**: 它通过包含 `M68kInstrData.td`, `M68kInstrShiftRotate.td`, `M68kInstrBits.td`, `M68kInstrArithmetic.td`, `M68kInstrControl.td`, `M68kInstrAtomics.td` 组合 TableGen 记录，并复用此前声明的后端片段。 该代码块声明或细化了 `MxType16d`, `MxType16a`, `MxType16r`, `MxType32d`, `MxType32a`, `MxType32r` 等 TableGen 记录。

### Lines 841-842 / 第 841-842 行
```tablegen
 841: 
 842: include "M68kInstrCompiler.td"
```
- **EN**: It composes TableGen records by including `M68kInstrCompiler.td` and reusing previously declared backend fragments.
- **CN**: 它通过包含 `M68kInstrCompiler.td` 组合 TableGen 记录，并复用此前声明的后端片段。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `M68kInstrFormats.td`
- `M68kInstrData.td`
- `M68kInstrShiftRotate.td`
- `M68kInstrBits.td`
- `M68kInstrArithmetic.td`
- `M68kInstrControl.td`
- `M68kInstrAtomics.td`
- `M68kInstrCompiler.td`
