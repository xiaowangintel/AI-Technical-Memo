# SystemZOperands.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZOperands.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: //===-- SystemZOperands.td - SystemZ instruction operands ----*- tblgen-*--===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: // Class definitions
  11: //===----------------------------------------------------------------------===//
  12: 
  13: class ImmediateAsmOperand<string name>
  14:   : AsmOperandClass {
  15:   let Name = name;
  16:   let RenderMethod = "addImmOperands";
  17: }
  18: class ImmediateTLSAsmOperand<string name>
  19:   : AsmOperandClass {
  20:   let Name = name;
  21:   let RenderMethod = "addImmTLSOperands";
  22: }
  23: 
  24: class ImmediateOp<ValueType vt, string asmop> : Operand<vt> {
  25:   let PrintMethod = "print"#asmop#"Operand";
  26:   let EncoderMethod = "getImmOpValue<SystemZ::FK_390_"#asmop#">";
  27:   let DecoderMethod = "decode"#asmop#"Operand";
  28:   let ParserMatchClass = !cast<AsmOperandClass>(asmop);
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `ImmediateAsmOperand`, `ImmediateTLSAsmOperand`, `ImmediateOp`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `ImmediateAsmOperand`, `ImmediateTLSAsmOperand`, `ImmediateOp` 等 TableGen 记录。

### Lines 29-56 / 第 29-56 行
```tablegen
  29:   let OperandType = "OPERAND_IMMEDIATE";
  30: }
  31: 
  32: class ImmOpWithPattern<ValueType vt, string asmop, code pred, SDNodeXForm xform,
  33:       SDNode ImmNode = imm> :
  34:   ImmediateOp<vt, asmop>, ImmLeaf<vt, pred, xform, ImmNode> {
  35:   let IsAPInt = true;
  36:   let FastIselShouldIgnore = true;
  37: }
  38: 
  39: // Constructs both a DAG pattern and instruction operand for an immediate
  40: // of type VT.  PRED returns true if a node is acceptable and XFORM returns
  41: // the operand value associated with the node.  ASMOP is the name of the
  42: // associated asm operand, and also forms the basis of the asm print method.
  43: multiclass Immediate<ValueType vt, code pred, SDNodeXForm xform, string asmop> {
  44:   def "" : ImmOpWithPattern<vt, asmop, pred, xform>;
  45: 
  46:   def _timm : ImmOpWithPattern<vt, asmop, pred, xform, timm>;
  47: }
  48: 
  49: // Constructs an asm operand for a PC-relative address.  SIZE says how
  50: // many bits there are.
  51: class PCRelAsmOperand<string size> : ImmediateAsmOperand<"PCRel"#size> {
  52:   let PredicateMethod = "isImm";
  53:   let ParserMethod = "parsePCRel"#size;
  54: }
  55: class PCRelTLSAsmOperand<string size>
  56:   : ImmediateTLSAsmOperand<"PCRelTLS"#size> {
```
- **EN**: This block declares or refines TableGen records such as `ImmOpWithPattern`, `Immediate`, `_timm`, `PCRelAsmOperand`, `PCRelTLSAsmOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `ImmOpWithPattern`, `Immediate`, `_timm`, `PCRelAsmOperand`, `PCRelTLSAsmOperand` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 57-84 / 第 57-84 行
```tablegen
  57:   let PredicateMethod = "isImmTLS";
  58:   let ParserMethod = "parsePCRelTLS"#size;
  59: }
  60: 
  61: // Constructs an operand for a PC-relative address with address type VT.
  62: // ASMOP is the associated asm operand.
  63: let OperandType = "OPERAND_PCREL" in {
  64:   class PCRelOperand<ValueType vt, AsmOperandClass asmop> : Operand<vt> {
  65:     let PrintMethod = "printPCRelOperand";
  66:     let ParserMatchClass = asmop;
  67:   }
  68:   class PCRelTLSOperand<ValueType vt, AsmOperandClass asmop> : Operand<vt> {
  69:     let PrintMethod = "printPCRelTLSOperand";
  70:     let ParserMatchClass = asmop;
  71:   }
  72: }
  73: 
  74: // Constructs both a DAG pattern and instruction operand for a PC-relative
  75: // address with address size VT.  SELF is the name of the operand and
  76: // ASMOP is the associated asm operand.
  77: class PCRelAddress<ValueType vt, string self, AsmOperandClass asmop>
  78:   : ComplexPattern<vt, 1, "selectPCRelAddress",
  79:                    [z_pcrel_wrapper, z_pcrel_offset]>,
  80:     PCRelOperand<vt, asmop> {
  81:   let MIOperandInfo = (ops !cast<Operand>(self));
  82: }
  83: 
  84: // Constructs an AsmOperandClass for addressing mode FORMAT, treating the
```
- **EN**: This block declares or refines TableGen records such as `PCRelOperand`, `PCRelTLSOperand`, `PCRelAddress`.
- **CN**: 该代码块声明或细化了 `PCRelOperand`, `PCRelTLSOperand`, `PCRelAddress` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85: // registers as having BITSIZE bits and displacements as having DISPSIZE bits.
  86: // LENGTH is "LenN" for addresses with an N-bit length field, otherwise it
  87: // is "".
  88: class AddressAsmOperand<string format, string bitsize, string dispsize,
  89:                         string length = "">
  90:   : AsmOperandClass {
  91:   let Name = format#bitsize#"Disp"#dispsize#length;
  92:   let ParserMethod = "parse"#format#bitsize;
  93:   let RenderMethod = "add"#format#"Operands";
  94: }
  95: 
  96: // Constructs an instruction operand for an addressing mode.  FORMAT,
  97: // BITSIZE, DISPSIZE and LENGTH are the parameters to an associated
  98: // AddressAsmOperand.  OPERANDS is a list of individual operands
  99: // (base register, displacement, etc.).
 100: class AddressOperand<string bitsize, string dispsize, string length,
 101:                      string format, dag operands>
 102:   : Operand<!cast<ValueType>("i"#bitsize)> {
 103:   let PrintMethod = "print"#format#"Operand";
 104:   let OperandType = "OPERAND_MEMORY";
 105:   let MIOperandInfo = operands;
 106:   let ParserMatchClass =
 107:     !cast<AddressAsmOperand>(format#bitsize#"Disp"#dispsize#length);
 108: }
 109: 
 110: // Constructs both a DAG pattern and instruction operand for an addressing mode.
 111: // FORMAT, BITSIZE, DISPSIZE and LENGTH are the parameters to an associated
 112: // AddressAsmOperand.  OPERANDS is a list of NUMOPS individual operands
```
- **EN**: This block declares or refines TableGen records such as `AddressAsmOperand`, `AddressOperand`.
- **CN**: 该代码块声明或细化了 `AddressAsmOperand`, `AddressOperand` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113: // (base register, displacement, etc.).  SELTYPE is the type of the memory
 114: // operand for selection purposes; sometimes we want different selection
 115: // choices for the same underlying addressing mode.  SUFFIX is similarly
 116: // a suffix appended to the displacement for selection purposes;
 117: // e.g. we want to reject small 20-bit displacements if a 12-bit form
 118: // also exists, but we want to accept them otherwise.
 119: class AddressingMode<string seltype, string bitsize, string dispsize,
 120:                      string suffix, string length, int numops, string format,
 121:                      dag operands>
 122:   : ComplexPattern<!cast<ValueType>("i"#bitsize), numops,
 123:                    "select"#seltype#dispsize#suffix#length,
 124:                    [add, sub, or, frameindex, z_adjdynalloc]>,
 125:     AddressOperand<bitsize, dispsize, length, format, operands>;
 126: 
 127: // An addressing mode with a base and displacement but no index.
 128: class BDMode<string type, string bitsize, string dispsize, string suffix>
 129:   : AddressingMode<type, bitsize, dispsize, suffix, "", 2, "BDAddr",
 130:                    (ops !cast<RegisterOperand>("ADDR"#bitsize),
 131:                         !cast<Operand>("disp"#dispsize#"imm"#bitsize))>;
 132: 
 133: // An addressing mode with a base, displacement and index.
 134: class BDXMode<string type, string bitsize, string dispsize, string suffix>
 135:   : AddressingMode<type, bitsize, dispsize, suffix, "", 3, "BDXAddr",
 136:                    (ops !cast<RegisterOperand>("ADDR"#bitsize),
 137:                         !cast<Operand>("disp"#dispsize#"imm"#bitsize),
 138:                         !cast<RegisterOperand>("ADDR"#bitsize))>;
 139: 
 140: // A BDMode paired with an immediate length operand of LENSIZE bits.
```
- **EN**: This block declares or refines TableGen records such as `AddressingMode`, `BDMode`, `BDXMode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `AddressingMode`, `BDMode`, `BDXMode` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 141-168 / 第 141-168 行
```tablegen
 141: class BDLMode<string type, string bitsize, string dispsize, string suffix,
 142:               string lensize>
 143:   : AddressingMode<type, bitsize, dispsize, suffix, "Len"#lensize, 3,
 144:                    "BDLAddr",
 145:                    (ops !cast<RegisterOperand>("ADDR"#bitsize),
 146:                         !cast<Operand>("disp"#dispsize#"imm"#bitsize),
 147:                         !cast<Operand>("len"#lensize#"imm"#bitsize))>;
 148: 
 149: // A BDMode paired with a register length operand.
 150: class BDRMode<string type, string bitsize, string dispsize, string suffix>
 151:   : AddressingMode<type, bitsize, dispsize, suffix, "", 3, "BDRAddr",
 152:                    (ops !cast<RegisterOperand>("ADDR"#bitsize),
 153:                         !cast<Operand>("disp"#dispsize#"imm"#bitsize),
 154:                         !cast<RegisterOperand>("GR"#bitsize))>;
 155: 
 156: // An addressing mode with a base, displacement and a vector index.
 157: class BDVMode<string bitsize, string dispsize>
 158:   : AddressOperand<bitsize, dispsize, "", "BDVAddr",
 159:                    (ops !cast<RegisterOperand>("ADDR"#bitsize),
 160:                         !cast<Operand>("disp"#dispsize#"imm"#bitsize),
 161:                         !cast<RegisterOperand>("VR128"))>;
 162: 
 163: // An addressing mode with a base, 32-bit displacement and 32-bit index.
 164: class LXAMode<string bitsize, string dispsize>
 165:   : AddressOperand<bitsize, dispsize, "", "LXAAddr",
 166:                    (ops !cast<RegisterOperand>("ADDR"#bitsize),
 167:                         !cast<Operand>("disp"#dispsize#"imm32"),
 168:                         !cast<RegisterOperand>("ADDR32"))>;
```
- **EN**: This block declares or refines TableGen records such as `BDLMode`, `BDRMode`, `BDVMode`, `LXAMode`.
- **CN**: 该代码块声明或细化了 `BDLMode`, `BDRMode`, `BDVMode`, `LXAMode` 等 TableGen 记录。

### Lines 169-196 / 第 169-196 行
```tablegen
 169: 
 170: //===----------------------------------------------------------------------===//
 171: // Extracting immediate operands from nodes
 172: // These all create MVT::i64 nodes to ensure the value is not sign-extended
 173: // when converted from an SDNode to a MachineOperand later on.
 174: //===----------------------------------------------------------------------===//
 175: 
 176: // Bits 0-15 (counting from the lsb).
 177: def LL16 : SDNodeXForm<imm, [{
 178:   uint64_t Value = N->getZExtValue() & 0x000000000000FFFFULL;
 179:   return CurDAG->getTargetConstant(Value, SDLoc(N), MVT::i64);
 180: }]>;
 181: 
 182: // Bits 16-31 (counting from the lsb).
 183: def LH16 : SDNodeXForm<imm, [{
 184:   uint64_t Value = (N->getZExtValue() & 0x00000000FFFF0000ULL) >> 16;
 185:   return CurDAG->getTargetConstant(Value, SDLoc(N), MVT::i64);
 186: }]>;
 187: 
 188: // Bits 32-47 (counting from the lsb).
 189: def HL16 : SDNodeXForm<imm, [{
 190:   uint64_t Value = (N->getZExtValue() & 0x0000FFFF00000000ULL) >> 32;
 191:   return CurDAG->getTargetConstant(Value, SDLoc(N), MVT::i64);
 192: }]>;
 193: 
 194: // Bits 48-63 (counting from the lsb).
 195: def HH16 : SDNodeXForm<imm, [{
 196:   uint64_t Value = (N->getZExtValue() & 0xFFFF000000000000ULL) >> 48;
```
- **EN**: This block declares or refines TableGen records such as `LL16`, `LH16`, `HL16`, `HH16`.
- **CN**: 该代码块声明或细化了 `LL16`, `LH16`, `HL16`, `HH16` 等 TableGen 记录。

### Lines 197-224 / 第 197-224 行
```tablegen
 197:   return CurDAG->getTargetConstant(Value, SDLoc(N), MVT::i64);
 198: }]>;
 199: 
 200: // Low 32 bits.
 201: def LF32 : SDNodeXForm<imm, [{
 202:   uint64_t Value = N->getZExtValue() & 0x00000000FFFFFFFFULL;
 203:   return CurDAG->getTargetConstant(Value, SDLoc(N), MVT::i64);
 204: }]>;
 205: 
 206: // High 32 bits.
 207: def HF32 : SDNodeXForm<imm, [{
 208:   uint64_t Value = N->getZExtValue() >> 32;
 209:   return CurDAG->getTargetConstant(Value, SDLoc(N), MVT::i64);
 210: }]>;
 211: 
 212: // Negated variants.
 213: def NEGLH16 : SDNodeXForm<imm, [{
 214:   uint64_t Value = (-N->getZExtValue() & 0x00000000FFFF0000ULL) >> 16;
 215:   return CurDAG->getTargetConstant(Value, SDLoc(N), MVT::i64);
 216: }]>;
 217: 
 218: def NEGLF32 : SDNodeXForm<imm, [{
 219:   uint64_t Value = -N->getZExtValue() & 0x00000000FFFFFFFFULL;
 220:   return CurDAG->getTargetConstant(Value, SDLoc(N), MVT::i64);
 221: }]>;
 222: 
 223: // Truncate an immediate to a 8-bit signed quantity.
 224: def SIMM8 : SDNodeXForm<imm, [{
```
- **EN**: This block declares or refines TableGen records such as `LF32`, `HF32`, `NEGLH16`, `NEGLF32`, `SIMM8`.
- **CN**: 该代码块声明或细化了 `LF32`, `HF32`, `NEGLH16`, `NEGLF32`, `SIMM8` 等 TableGen 记录。

### Lines 225-252 / 第 225-252 行
```tablegen
 225:   return CurDAG->getSignedTargetConstant(int8_t(N->getSExtValue()), SDLoc(N),
 226:                                          MVT::i64);
 227: }]>;
 228: 
 229: // Truncate an immediate to a 8-bit unsigned quantity.
 230: def UIMM8 : SDNodeXForm<imm, [{
 231:   return CurDAG->getTargetConstant(uint8_t(N->getZExtValue()), SDLoc(N),
 232:                                    MVT::i64);
 233: }]>;
 234: 
 235: // Truncate an immediate to a 8-bit unsigned quantity and mask off low bit.
 236: def UIMM8EVEN : SDNodeXForm<imm, [{
 237:   return CurDAG->getTargetConstant(N->getZExtValue() & 0xfe, SDLoc(N),
 238:                                    MVT::i64);
 239: }]>;
 240: 
 241: // Truncate an immediate to a 12-bit unsigned quantity.
 242: def UIMM12 : SDNodeXForm<imm, [{
 243:   return CurDAG->getTargetConstant(N->getZExtValue() & 0xfff, SDLoc(N),
 244:                                    MVT::i64);
 245: }]>;
 246: 
 247: // Truncate an immediate to a 16-bit signed quantity.
 248: def SIMM16 : SDNodeXForm<imm, [{
 249:   return CurDAG->getSignedTargetConstant(int16_t(N->getSExtValue()), SDLoc(N),
 250:                                          MVT::i64);
 251: }]>;
 252: 
```
- **EN**: This block declares or refines TableGen records such as `UIMM8`, `UIMM8EVEN`, `UIMM12`, `SIMM16`.
- **CN**: 该代码块声明或细化了 `UIMM8`, `UIMM8EVEN`, `UIMM12`, `SIMM16` 等 TableGen 记录。

### Lines 253-280 / 第 253-280 行
```tablegen
 253: // Negate and then truncate an immediate to a 16-bit signed quantity.
 254: def NEGSIMM16 : SDNodeXForm<imm, [{
 255:   return CurDAG->getSignedTargetConstant(int16_t(-N->getSExtValue()), SDLoc(N),
 256:                                          MVT::i64);
 257: }]>;
 258: 
 259: // Truncate an immediate to a 16-bit unsigned quantity.
 260: def UIMM16 : SDNodeXForm<imm, [{
 261:   return CurDAG->getTargetConstant(uint16_t(N->getZExtValue()), SDLoc(N),
 262:                                    MVT::i64);
 263: }]>;
 264: 
 265: // Truncate an immediate to a 32-bit signed quantity.
 266: def SIMM32 : SDNodeXForm<imm, [{
 267:   return CurDAG->getSignedTargetConstant(int32_t(N->getSExtValue()), SDLoc(N),
 268:                                          MVT::i64);
 269: }]>;
 270: 
 271: // Negate and then truncate an immediate to a 32-bit unsigned quantity.
 272: def NEGSIMM32 : SDNodeXForm<imm, [{
 273:   return CurDAG->getSignedTargetConstant(int32_t(-N->getSExtValue()), SDLoc(N),
 274:                                          MVT::i64);
 275: }]>;
 276: 
 277: // Truncate an immediate to a 32-bit unsigned quantity.
 278: def UIMM32 : SDNodeXForm<imm, [{
 279:   return CurDAG->getTargetConstant(uint32_t(N->getZExtValue()), SDLoc(N),
 280:                                    MVT::i64);
```
- **EN**: This block declares or refines TableGen records such as `NEGSIMM16`, `UIMM16`, `SIMM32`, `NEGSIMM32`, `UIMM32`.
- **CN**: 该代码块声明或细化了 `NEGSIMM16`, `UIMM16`, `SIMM32`, `NEGSIMM32`, `UIMM32` 等 TableGen 记录。

### Lines 281-308 / 第 281-308 行
```tablegen
 281: }]>;
 282: 
 283: // Negate and then truncate an immediate to a 32-bit unsigned quantity.
 284: def NEGUIMM32 : SDNodeXForm<imm, [{
 285:   return CurDAG->getTargetConstant(uint32_t(-N->getZExtValue()), SDLoc(N),
 286:                                    MVT::i64);
 287: }]>;
 288: 
 289: // Truncate an immediate to a 48-bit unsigned quantity.
 290: def UIMM48 : SDNodeXForm<imm, [{
 291:   return CurDAG->getTargetConstant(uint64_t(N->getZExtValue()) & 0xffffffffffff,
 292:                                    SDLoc(N), MVT::i64);
 293: }]>;
 294: 
 295: //===----------------------------------------------------------------------===//
 296: // Immediate asm operands.
 297: //===----------------------------------------------------------------------===//
 298: 
 299: def U1Imm  : ImmediateAsmOperand<"U1Imm">;
 300: def U2Imm  : ImmediateAsmOperand<"U2Imm">;
 301: def U3Imm  : ImmediateAsmOperand<"U3Imm">;
 302: def U4Imm  : ImmediateAsmOperand<"U4Imm">;
 303: def S8Imm  : ImmediateAsmOperand<"S8Imm">;
 304: def U8Imm  : ImmediateAsmOperand<"U8Imm">;
 305: def U12Imm : ImmediateAsmOperand<"U12Imm">;
 306: def S16Imm : ImmediateAsmOperand<"S16Imm">;
 307: def U16Imm : ImmediateAsmOperand<"U16Imm">;
 308: def S32Imm : ImmediateAsmOperand<"S32Imm">;
```
- **EN**: This block declares or refines TableGen records such as `NEGUIMM32`, `UIMM48`, `U1Imm`, `U2Imm`, `U3Imm`, `U4Imm`.
- **CN**: 该代码块声明或细化了 `NEGUIMM32`, `UIMM48`, `U1Imm`, `U2Imm`, `U3Imm`, `U4Imm` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309: def U32Imm : ImmediateAsmOperand<"U32Imm">;
 310: def U48Imm : ImmediateAsmOperand<"U48Imm">;
 311: 
 312: //===----------------------------------------------------------------------===//
 313: // i32 immediates
 314: //===----------------------------------------------------------------------===//
 315: 
 316: // Immediates for the lower and upper 16 bits of an i32, with the other
 317: // bits of the i32 being zero.
 318: defm imm32ll16 : Immediate<i32, [{
 319:   return Imm.isIntN(32) && SystemZ::isImmLL(Imm.getZExtValue());
 320: }], LL16, "U16Imm">;
 321: 
 322: defm imm32lh16 : Immediate<i32, [{
 323:   return Imm.isIntN(32) && SystemZ::isImmLH(Imm.getZExtValue());
 324: }], LH16, "U16Imm">;
 325: 
 326: // Immediates for the lower and upper 16 bits of an i32, with the other
 327: // bits of the i32 being one.
 328: defm imm32ll16c : Immediate<i32, [{
 329:   return Imm.isIntN(32) &&
 330:          SystemZ::isImmLL(uint32_t(~Imm.getZExtValue()));
 331: }], LL16, "U16Imm">;
 332: 
 333: defm imm32lh16c : Immediate<i32, [{
 334:   return Imm.isIntN(32) &&
 335:          SystemZ::isImmLH(uint32_t(~Imm.getZExtValue()));
 336: }], LH16, "U16Imm">;
```
- **EN**: This block declares or refines TableGen records such as `U32Imm`, `U48Imm`, `imm32ll16`, `imm32lh16`, `imm32ll16c`, `imm32lh16c`.
- **CN**: 该代码块声明或细化了 `U32Imm`, `U48Imm`, `imm32ll16`, `imm32lh16`, `imm32ll16c`, `imm32lh16c` 等 TableGen 记录。

### Lines 337-364 / 第 337-364 行
```tablegen
 337: 
 338: // Short immediates
 339: defm imm32zx1 : Immediate<i32, [{
 340:   return Imm.isIntN(1);
 341: }], NOOP_SDNodeXForm, "U1Imm">;
 342: 
 343: defm imm32zx2 : Immediate<i32, [{
 344:   return Imm.isIntN(2);
 345: }], NOOP_SDNodeXForm, "U2Imm">;
 346: 
 347: defm imm32zx3 : Immediate<i32, [{
 348:   return Imm.isIntN(3);
 349: }], NOOP_SDNodeXForm, "U3Imm">;
 350: 
 351: defm imm32zx4 : Immediate<i32, [{
 352:   return Imm.isIntN(4);
 353: }], NOOP_SDNodeXForm, "U4Imm">;
 354: 
 355: // Note: this enforces an even value during code generation only.
 356: // When used from the assembler, any 4-bit value is allowed.
 357: defm imm32zx4even : Immediate<i32, [{
 358:   return Imm.isIntN(4);
 359: }], UIMM8EVEN, "U4Imm">;
 360: 
 361: defm imm32sx8 : Immediate<i32, [{
 362:   return Imm.isSignedIntN(8);
 363: }], SIMM8, "S8Imm">;
 364: 
```
- **EN**: This block declares or refines TableGen records such as `imm32zx1`, `imm32zx2`, `imm32zx3`, `imm32zx4`, `imm32zx4even`, `imm32sx8`.
- **CN**: 该代码块声明或细化了 `imm32zx1`, `imm32zx2`, `imm32zx3`, `imm32zx4`, `imm32zx4even`, `imm32sx8` 等 TableGen 记录。

### Lines 365-392 / 第 365-392 行
```tablegen
 365: defm imm32zx8 : Immediate<i32, [{
 366:   return Imm.isIntN(8);
 367: }], UIMM8, "U8Imm">;
 368: 
 369: defm imm32zx8trunc : Immediate<i32, [{}], UIMM8, "U8Imm">;
 370: 
 371: defm imm32zx12 : Immediate<i32, [{
 372:   return Imm.isIntN(12);
 373: }], UIMM12, "U12Imm">;
 374: 
 375: defm imm32sx16 : Immediate<i32, [{
 376:   return Imm.isSignedIntN(16);
 377: }], SIMM16, "S16Imm">;
 378: 
 379: defm imm32sx16n : Immediate<i32, [{
 380:   return (-Imm).isSignedIntN(16);
 381: }], NEGSIMM16, "S16Imm">;
 382: 
 383: defm imm32zx16 : Immediate<i32, [{
 384:   return Imm.isIntN(16);
 385: }], UIMM16, "U16Imm">;
 386: 
 387: defm imm32sx16trunc : Immediate<i32, [{}], SIMM16, "S16Imm">;
 388: defm imm32zx16trunc : Immediate<i32, [{}], UIMM16, "U16Imm">;
 389: 
 390: // Full 32-bit immediates.  we need both signed and unsigned versions
 391: // because the assembler is picky.  E.g. AFI requires signed operands
 392: // while NILF requires unsigned ones.
```
- **EN**: This block declares or refines TableGen records such as `imm32zx8`, `imm32zx8trunc`, `imm32zx12`, `imm32sx16`, `imm32sx16n`, `imm32zx16`.
- **CN**: 该代码块声明或细化了 `imm32zx8`, `imm32zx8trunc`, `imm32zx12`, `imm32sx16`, `imm32sx16n`, `imm32zx16` 等 TableGen 记录。

### Lines 393-420 / 第 393-420 行
```tablegen
 393: defm simm32 : Immediate<i32, [{}], SIMM32, "S32Imm">;
 394: defm uimm32 : Immediate<i32, [{}], UIMM32, "U32Imm">;
 395: 
 396: defm simm32n : Immediate<i32, [{
 397:   auto SImm = Imm.trySExtValue();
 398:   return SImm.has_value() && isInt<32>(-*SImm);
 399: }], NEGSIMM32, "S32Imm">;
 400: 
 401: def imm32 : ImmLeaf<i32, [{}]>;
 402: 
 403: //===----------------------------------------------------------------------===//
 404: // 64-bit immediates
 405: //===----------------------------------------------------------------------===//
 406: 
 407: // Immediates for 16-bit chunks of an i64, with the other bits of the
 408: // i32 being zero.
 409: defm imm64ll16 : Immediate<i64, [{
 410:   return Imm.isIntN(64) && SystemZ::isImmLL(Imm.getZExtValue());
 411: }], LL16, "U16Imm">;
 412: 
 413: defm imm64lh16 : Immediate<i64, [{
 414:   return Imm.isIntN(64) && SystemZ::isImmLH(Imm.getZExtValue());
 415: }], LH16, "U16Imm">;
 416: 
 417: defm imm64hl16 : Immediate<i64, [{
 418:   return Imm.isIntN(64) && SystemZ::isImmHL(Imm.getZExtValue());
 419: }], HL16, "U16Imm">;
 420: 
```
- **EN**: This block declares or refines TableGen records such as `simm32`, `uimm32`, `simm32n`, `imm32`, `imm64ll16`, `imm64lh16`.
- **CN**: 该代码块声明或细化了 `simm32`, `uimm32`, `simm32n`, `imm32`, `imm64ll16`, `imm64lh16` 等 TableGen 记录。

### Lines 421-448 / 第 421-448 行
```tablegen
 421: defm imm64hh16 : Immediate<i64, [{
 422:   return Imm.isIntN(64) && SystemZ::isImmHH(Imm.getZExtValue());
 423: }], HH16, "U16Imm">;
 424: 
 425: // Immediates for 16-bit chunks of an i64, with the other bits of the
 426: // i32 being one.
 427: defm imm64ll16c : Immediate<i64, [{
 428:   return Imm.isIntN(64) &&
 429:          SystemZ::isImmLL(uint64_t(~Imm.getZExtValue()));
 430: }], LL16, "U16Imm">;
 431: 
 432: defm imm64lh16c : Immediate<i64, [{
 433:   return Imm.isIntN(64) &&
 434:          SystemZ::isImmLH(uint64_t(~Imm.getZExtValue()));
 435: }], LH16, "U16Imm">;
 436: 
 437: defm imm64hl16c : Immediate<i64, [{
 438:   return Imm.isIntN(64) &&
 439:          SystemZ::isImmHL(uint64_t(~Imm.getZExtValue()));
 440: }], HL16, "U16Imm">;
 441: 
 442: defm imm64hh16c : Immediate<i64, [{
 443:   return Imm.isIntN(64) &&
 444:          SystemZ::isImmHH(uint64_t(~Imm.getZExtValue()));
 445: }], HH16, "U16Imm">;
 446: 
 447: // Immediates for the lower and upper 32 bits of an i64, with the other
 448: // bits of the i32 being zero.
```
- **EN**: This block declares or refines TableGen records such as `imm64hh16`, `imm64ll16c`, `imm64lh16c`, `imm64hl16c`, `imm64hh16c`.
- **CN**: 该代码块声明或细化了 `imm64hh16`, `imm64ll16c`, `imm64lh16c`, `imm64hl16c`, `imm64hh16c` 等 TableGen 记录。

### Lines 449-476 / 第 449-476 行
```tablegen
 449: defm imm64lf32 : Immediate<i64, [{
 450:   return Imm.isIntN(64) && SystemZ::isImmLF(Imm.getZExtValue());
 451: }], LF32, "U32Imm">;
 452: 
 453: defm imm64hf32 : Immediate<i64, [{
 454:   return Imm.isIntN(64) && SystemZ::isImmHF(Imm.getZExtValue());
 455: }], HF32, "U32Imm">;
 456: 
 457: // Immediates for the lower and upper 32 bits of an i64, with the other
 458: // bits of the i32 being one.
 459: defm imm64lf32c : Immediate<i64, [{
 460:   return Imm.isIntN(64) &&
 461:          SystemZ::isImmLF(uint64_t(~Imm.getZExtValue()));
 462: }], LF32, "U32Imm">;
 463: 
 464: defm imm64hf32c : Immediate<i64, [{
 465:   return Imm.isIntN(64) &&
 466:          SystemZ::isImmHF(uint64_t(~Imm.getZExtValue()));
 467: }], HF32, "U32Imm">;
 468: 
 469: // Negated immediates that fit LF32 or LH16.
 470: defm imm64lh16n : Immediate<i64, [{
 471:   return Imm.isIntN(64) &&
 472:          SystemZ::isImmLH(uint64_t(-Imm.getZExtValue()));
 473: }], NEGLH16, "U16Imm">;
 474: 
 475: defm imm64lf32n : Immediate<i64, [{
 476:   return Imm.isIntN(64) &&
```
- **EN**: This block declares or refines TableGen records such as `imm64lf32`, `imm64hf32`, `imm64lf32c`, `imm64hf32c`, `imm64lh16n`, `imm64lf32n`.
- **CN**: 该代码块声明或细化了 `imm64lf32`, `imm64hf32`, `imm64lf32c`, `imm64hf32c`, `imm64lh16n`, `imm64lf32n` 等 TableGen 记录。

### Lines 477-504 / 第 477-504 行
```tablegen
 477:          SystemZ::isImmLF(uint64_t(-Imm.getZExtValue()));
 478: }], NEGLF32, "U32Imm">;
 479: 
 480: // Short immediates.
 481: defm imm64sx8 : Immediate<i64, [{
 482:   return Imm.isSignedIntN(8);
 483: }], SIMM8, "S8Imm">;
 484: 
 485: defm imm64zx8 : Immediate<i64, [{
 486:   return Imm.isIntN(8);;
 487: }], UIMM8, "U8Imm">;
 488: 
 489: defm imm64sx16 : Immediate<i64, [{
 490:   return Imm.isSignedIntN(16);
 491: }], SIMM16, "S16Imm">;
 492: 
 493: defm imm64sx16n : Immediate<i64, [{
 494:   return (-Imm).isSignedIntN(16);
 495: }], NEGSIMM16, "S16Imm">;
 496: 
 497: defm imm64zx16 : Immediate<i64, [{
 498:   return Imm.isIntN(16);
 499: }], UIMM16, "U16Imm">;
 500: 
 501: defm imm64sx32 : Immediate<i64, [{
 502:   return Imm.isSignedIntN(32);
 503: }], SIMM32, "S32Imm">;
 504: 
```
- **EN**: This block declares or refines TableGen records such as `imm64sx8`, `imm64zx8`, `imm64sx16`, `imm64sx16n`, `imm64zx16`, `imm64sx32`.
- **CN**: 该代码块声明或细化了 `imm64sx8`, `imm64zx8`, `imm64sx16`, `imm64sx16n`, `imm64zx16`, `imm64sx32` 等 TableGen 记录。

### Lines 505-532 / 第 505-532 行
```tablegen
 505: defm imm64sx32n : Immediate<i64, [{
 506:   return (-Imm).isSignedIntN(32);
 507: }], NEGSIMM32, "S32Imm">;
 508: 
 509: defm imm64zx32 : Immediate<i64, [{
 510:   return Imm.isIntN(32);
 511: }], UIMM32, "U32Imm">;
 512: 
 513: defm imm64zx32n : Immediate<i64, [{
 514:   return (-Imm).isIntN(32);
 515: }], NEGUIMM32, "U32Imm">;
 516: 
 517: defm imm64zx48 : Immediate<i64, [{
 518:   return Imm.isIntN(64);
 519: }], UIMM48, "U48Imm">;
 520: 
 521: class Imm64 : ImmLeaf<i64, [{}]>, Operand<i64> {
 522:   let OperandType = "OPERAND_IMMEDIATE";
 523: }
 524: def imm64 : Imm64;
 525: def len4imm64 : Imm64 {
 526:   let EncoderMethod = "getLenEncoding<SystemZ::FK_390_U4Imm>";
 527:   let DecoderMethod = "decodeLenOperand<4>";
 528: }
 529: def len8imm64 : Imm64 {
 530:   let EncoderMethod = "getLenEncoding<SystemZ::FK_390_U8Imm>";
 531:   let DecoderMethod = "decodeLenOperand<8>";
 532: }
```
- **EN**: This block declares or refines TableGen records such as `imm64sx32n`, `imm64zx32`, `imm64zx32n`, `imm64zx48`, `Imm64`, `imm64`.
- **CN**: 该代码块声明或细化了 `imm64sx32n`, `imm64zx32`, `imm64zx32n`, `imm64zx48`, `Imm64`, `imm64` 等 TableGen 记录。

### Lines 533-560 / 第 533-560 行
```tablegen
 533: 
 534: //===----------------------------------------------------------------------===//
 535: // Floating-point immediates
 536: //===----------------------------------------------------------------------===//
 537: 
 538: // Floating-point zero.
 539: def fpimm0 : FPImmLeaf<fAny, [{ return Imm.isExactlyValue(+0.0); }]>;
 540: 
 541: // Floating point negative zero.
 542: def fpimmneg0 : FPImmLeaf<fAny, [{ return Imm.isExactlyValue(-0.0); }]>;
 543: 
 544: //===----------------------------------------------------------------------===//
 545: // Symbolic address operands
 546: //===----------------------------------------------------------------------===//
 547: 
 548: // PC-relative asm operands.
 549: def PCRel12 : PCRelAsmOperand<"12">;
 550: def PCRel16 : PCRelAsmOperand<"16">;
 551: def PCRel24 : PCRelAsmOperand<"24">;
 552: def PCRel32 : PCRelAsmOperand<"32">;
 553: def PCRelTLS16 : PCRelTLSAsmOperand<"16">;
 554: def PCRelTLS32 : PCRelTLSAsmOperand<"32">;
 555: 
 556: // PC-relative offsets of a basic block.  The offset is sign-extended
 557: // and multiplied by 2.
 558: def brtarget16 : PCRelOperand<OtherVT, PCRel16> {
 559:   let EncoderMethod = "getPC16DBLEncoding";
 560:   let DecoderMethod = "decodePC16DBLBranchOperand";
```
- **EN**: This block declares or refines TableGen records such as `fpimm0`, `fpimmneg0`, `PCRel12`, `PCRel16`, `PCRel24`, `PCRel32`.
- **CN**: 该代码块声明或细化了 `fpimm0`, `fpimmneg0`, `PCRel12`, `PCRel16`, `PCRel24`, `PCRel32` 等 TableGen 记录。

### Lines 561-588 / 第 561-588 行
```tablegen
 561: }
 562: def brtarget32 : PCRelOperand<OtherVT, PCRel32> {
 563:   let EncoderMethod = "getPC32DBLEncoding";
 564:   let DecoderMethod = "decodePC32DBLBranchOperand";
 565: }
 566: 
 567: // Variants of brtarget for use with branch prediction preload.
 568: def brtarget12bpp : PCRelOperand<OtherVT, PCRel12> {
 569:   let EncoderMethod = "getPC12DBLBPPEncoding";
 570:   let DecoderMethod = "decodePC12DBLBranchOperand";
 571: }
 572: def brtarget16bpp : PCRelOperand<OtherVT, PCRel16> {
 573:   let EncoderMethod = "getPC16DBLBPPEncoding";
 574:   let DecoderMethod = "decodePC16DBLBranchOperand";
 575: }
 576: def brtarget24bpp : PCRelOperand<OtherVT, PCRel24> {
 577:   let EncoderMethod = "getPC24DBLBPPEncoding";
 578:   let DecoderMethod = "decodePC24DBLBranchOperand";
 579: }
 580: 
 581: // Variants of brtarget16/32 with an optional additional TLS symbol.
 582: // These are used to annotate calls to __tls_get_offset.
 583: def tlssym : Operand<i64> { }
 584: def brtarget16tls : PCRelTLSOperand<OtherVT, PCRelTLS16> {
 585:   let MIOperandInfo = (ops brtarget16:$func, tlssym:$sym);
 586:   let EncoderMethod = "getPC16DBLTLSEncoding";
 587:   let DecoderMethod = "decodePC16DBLBranchOperand";
 588: }
```
- **EN**: This block declares or refines TableGen records such as `brtarget32`, `brtarget12bpp`, `brtarget16bpp`, `brtarget24bpp`, `tlssym`, `brtarget16tls`.
- **CN**: 该代码块声明或细化了 `brtarget32`, `brtarget12bpp`, `brtarget16bpp`, `brtarget24bpp`, `tlssym`, `brtarget16tls` 等 TableGen 记录。

### Lines 589-616 / 第 589-616 行
```tablegen
 589: def brtarget32tls : PCRelTLSOperand<OtherVT, PCRelTLS32> {
 590:   let MIOperandInfo = (ops brtarget32:$func, tlssym:$sym);
 591:   let EncoderMethod = "getPC32DBLTLSEncoding";
 592:   let DecoderMethod = "decodePC32DBLBranchOperand";
 593: }
 594: 
 595: // A PC-relative offset of a global value.  The offset is sign-extended
 596: // and multiplied by 2.
 597: def pcrel32 : PCRelAddress<i64, "pcrel32", PCRel32> {
 598:   let EncoderMethod = "getPC32DBLEncoding";
 599:   let DecoderMethod = "decodePC32DBLOperand";
 600: }
 601: 
 602: //===----------------------------------------------------------------------===//
 603: // Addressing modes
 604: //===----------------------------------------------------------------------===//
 605: 
 606: class DispOp<ValueType vt, code pred> : Operand<vt>, IntImmLeaf<vt, pred>;
 607: 
 608: // 12-bit displacement operands.
 609: let EncoderMethod = "getImmOpValue<SystemZ::FK_390_U12Imm>",
 610:     DecoderMethod = "decodeU12ImmOperand" in {
 611:   def disp12imm32 : DispOp<i32, [{ return Imm.isIntN(12); }]>;
 612:   def disp12imm64 : DispOp<i64, [{ return Imm.isIntN(12); }]>;
 613: }
 614: 
 615: // 20-bit displacement operands.
 616: let EncoderMethod = "getImmOpValue<SystemZ::FK_390_S20Imm>",
```
- **EN**: This block declares or refines TableGen records such as `brtarget32tls`, `pcrel32`, `DispOp`, `disp12imm32`, `disp12imm64`.
- **CN**: 该代码块声明或细化了 `brtarget32tls`, `pcrel32`, `DispOp`, `disp12imm32`, `disp12imm64` 等 TableGen 记录。

### Lines 617-644 / 第 617-644 行
```tablegen
 617:     DecoderMethod = "decodeS20ImmOperand" in {
 618:   def disp20imm32 : DispOp<i32, [{ return Imm.isSignedIntN(20); }]>;
 619:   def disp20imm64 : DispOp<i64, [{ return Imm.isSignedIntN(20); }]>;
 620: }
 621: 
 622: def BDAddr32Disp12      : AddressAsmOperand<"BDAddr",   "32", "12">;
 623: def BDAddr32Disp20      : AddressAsmOperand<"BDAddr",   "32", "20">;
 624: def BDAddr64Disp12      : AddressAsmOperand<"BDAddr",   "64", "12">;
 625: def BDAddr64Disp20      : AddressAsmOperand<"BDAddr",   "64", "20">;
 626: def BDXAddr64Disp12     : AddressAsmOperand<"BDXAddr",  "64", "12">;
 627: def BDXAddr64Disp20     : AddressAsmOperand<"BDXAddr",  "64", "20">;
 628: def BDLAddr64Disp12Len4 : AddressAsmOperand<"BDLAddr",  "64", "12", "Len4">;
 629: def BDLAddr64Disp12Len8 : AddressAsmOperand<"BDLAddr",  "64", "12", "Len8">;
 630: def BDRAddr64Disp12     : AddressAsmOperand<"BDRAddr",  "64", "12">;
 631: def BDVAddr64Disp12     : AddressAsmOperand<"BDVAddr",  "64", "12">;
 632: def LXAAddr64Disp20     : AddressAsmOperand<"LXAAddr",  "64", "20">;
 633: 
 634: // DAG patterns and operands for addressing modes.  Each mode has
 635: // the form <type><range><group>[<len>] where:
 636: //
 637: // <type> is one of:
 638: //   shift    : base + displacement (32-bit)
 639: //   bdaddr   : base + displacement
 640: //   mviaddr  : like bdaddr, but reject cases with a natural index
 641: //   bdxaddr  : base + displacement + index
 642: //   laaddr   : like bdxaddr, but used for Load Address operations
 643: //   lxaaddr  : like bdxaddr, but used for Load (Logical) Indexed Address
 644: //   dynalloc : base + displacement + index + ADJDYNALLOC
```
- **EN**: This block declares or refines TableGen records such as `disp20imm32`, `disp20imm64`, `BDAddr32Disp12`, `BDAddr32Disp20`, `BDAddr64Disp12`, `BDAddr64Disp20`.
- **CN**: 该代码块声明或细化了 `disp20imm32`, `disp20imm64`, `BDAddr32Disp12`, `BDAddr32Disp20`, `BDAddr64Disp12`, `BDAddr64Disp20` 等 TableGen 记录。

### Lines 645-672 / 第 645-672 行
```tablegen
 645: //   bdladdr  : base + displacement with a length field
 646: //   bdvaddr  : base + displacement with a vector index
 647: //
 648: // <range> is one of:
 649: //   12       : the displacement is an unsigned 12-bit value
 650: //   20       : the displacement is a signed 20-bit value
 651: //
 652: // <group> is one of:
 653: //   pair     : used when there is an equivalent instruction with the opposite
 654: //              range value (12 or 20)
 655: //   only     : used when there is no equivalent instruction with the opposite
 656: //              range value
 657: //
 658: // <len> is one of:
 659: //
 660: //   <empty>  : there is no length field
 661: //   len8     : the length field is 8 bits, with a range of [1, 0x100].
 662: def shift12only       : BDMode <"BDAddr",   "32", "12", "Only">;
 663: def shift20only       : BDMode <"BDAddr",   "32", "20", "Only">;
 664: def bdaddr12only      : BDMode <"BDAddr",   "64", "12", "Only">;
 665: def bdaddr12pair      : BDMode <"BDAddr",   "64", "12", "Pair">;
 666: def bdaddr20only      : BDMode <"BDAddr",   "64", "20", "Only">;
 667: def bdaddr20pair      : BDMode <"BDAddr",   "64", "20", "Pair">;
 668: def mviaddr12pair     : BDMode <"MVIAddr",  "64", "12", "Pair">;
 669: def mviaddr20pair     : BDMode <"MVIAddr",  "64", "20", "Pair">;
 670: def bdxaddr12only     : BDXMode<"BDXAddr",  "64", "12", "Only">;
 671: def bdxaddr12pair     : BDXMode<"BDXAddr",  "64", "12", "Pair">;
 672: def bdxaddr20only     : BDXMode<"BDXAddr",  "64", "20", "Only">;
```
- **EN**: This block declares or refines TableGen records such as `shift12only`, `shift20only`, `bdaddr12only`, `bdaddr12pair`, `bdaddr20only`, `bdaddr20pair`.
- **CN**: 该代码块声明或细化了 `shift12only`, `shift20only`, `bdaddr12only`, `bdaddr12pair`, `bdaddr20only`, `bdaddr20pair` 等 TableGen 记录。

### Lines 673-696 / 第 673-696 行
```tablegen
 673: def bdxaddr20only128  : BDXMode<"BDXAddr",  "64", "20", "Only128">;
 674: def bdxaddr20pair     : BDXMode<"BDXAddr",  "64", "20", "Pair">;
 675: def dynalloc12only    : BDXMode<"DynAlloc", "64", "12", "Only">;
 676: def laaddr12pair      : BDXMode<"LAAddr",   "64", "12", "Pair">;
 677: def laaddr20pair      : BDXMode<"LAAddr",   "64", "20", "Pair">;
 678: def lxaaddr20only     : LXAMode<            "64", "20">;
 679: def bdladdr12onlylen4 : BDLMode<"BDLAddr",  "64", "12", "Only", "4">;
 680: def bdladdr12onlylen8 : BDLMode<"BDLAddr",  "64", "12", "Only", "8">;
 681: def bdraddr12only     : BDRMode<"BDRAddr",  "64", "12", "Only">;
 682: def bdvaddr12only     : BDVMode<            "64", "12">;
 683: 
 684: //===----------------------------------------------------------------------===//
 685: // Miscellaneous
 686: //===----------------------------------------------------------------------===//
 687: 
 688: // A 4-bit condition-code mask.
 689: def cond4 : ImmLeaf<i32, [{ return (Imm.getZExtValue() < 16); }],
 690:                     NOOP_SDNodeXForm, timm>,
 691:             Operand<i32> {
 692:   let PrintMethod = "printCond4Operand";
 693:   let OperandType = "OPERAND_IMMEDIATE";
 694:   let IsAPInt = true;
 695:   let FastIselShouldIgnore = true;
 696: }
```
- **EN**: This block declares or refines TableGen records such as `bdxaddr20only128`, `bdxaddr20pair`, `dynalloc12only`, `laaddr12pair`, `laaddr20pair`, `lxaaddr20only`.
- **CN**: 该代码块声明或细化了 `bdxaddr20only128`, `bdxaddr20pair`, `dynalloc12only`, `laaddr12pair`, `laaddr20pair`, `lxaaddr20only` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
