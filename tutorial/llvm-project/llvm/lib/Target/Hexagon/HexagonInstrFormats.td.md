# HexagonInstrFormats.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonInstrFormats.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Parse" bits are explicitly NOT defined in the opcode space to prevent
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```tablegen
     1: //==- HexagonInstrFormats.td - Hexagon Instruction Formats --*- tablegen -*-==//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: // Addressing modes for load/store instructions
    10: class AddrModeType<bits<3> value> {
    11:   bits<3> Value = value;
    12: }
    13: 
    14: def NoAddrMode     : AddrModeType<0>;  // No addressing mode
    15: def Absolute       : AddrModeType<1>;  // Absolute addressing mode
    16: def AbsoluteSet    : AddrModeType<2>;  // Absolute set addressing mode
    17: def BaseImmOffset  : AddrModeType<3>;  // Indirect with offset
    18: def BaseLongOffset : AddrModeType<4>;  // Indirect with long offset
    19: def BaseRegOffset  : AddrModeType<5>;  // Indirect with register offset
    20: def PostInc        : AddrModeType<6>;  // Post increment addressing mode
    21: 
    22: class MemAccessSize<bits<4> value> {
    23:   bits<4> Value = value;
    24: }
    25: 
    26: // These numbers must match the MemAccessSize enumeration values in
    27: // HexagonBaseInfo.h.
    28: def NoMemAccess      : MemAccessSize<0>;
    29: def ByteAccess       : MemAccessSize<1>;
    30: def HalfWordAccess   : MemAccessSize<2>;
    31: def WordAccess       : MemAccessSize<3>;
    32: def DoubleWordAccess : MemAccessSize<4>;
    33: def HVXVectorAccess  : MemAccessSize<5>;
    34: 
    35: 
    36: //===----------------------------------------------------------------------===//
    37: //                         Instruction Class Declaration +
    38: //===----------------------------------------------------------------------===//
    39: 
    40: // "Parse" bits are explicitly NOT defined in the opcode space to prevent
    41: //  TableGen from using them for generation of the decoder tables.
    42: class OpcodeHexagon {
    43:   field bits<32> Inst = ?; // Default to an invalid insn.
    44:   bits<4> IClass = 0; // ICLASS
    45:   bits<1> zero = 0;
    46: 
    47:   let Inst{31-28} = IClass;
    48: }
    49: 
    50: class InstHexagon<dag outs, dag ins, string asmstr, list<dag> pattern,
```
- EN: It declares types such as AddrModeType, MemAccessSize, OpcodeHexagon, InstHexagon, which carry the state or API of this component. It defines declarative TableGen records like AddrModeType, NoAddrMode, Absolute, AbsoluteSet, BaseImmOffset, ... (17 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrFormats, HexagonBaseInfo, showing how the code connects to sibling backend components.
- CN: 这里声明了 AddrModeType, MemAccessSize, OpcodeHexagon, InstHexagon 等类型，用来承载该组件的状态或接口。 这里定义了 AddrModeType, NoAddrMode, Absolute, AbsoluteSet, BaseImmOffset, ... (17 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrFormats, HexagonBaseInfo，说明了它与同级后端组件的连接关系。

### Lines 51-100 / 第 51-100 行

```tablegen
    51:                   string cstr, InstrItinClass itin, IType type>
    52:   : Instruction {
    53:   let Namespace = "Hexagon";
    54: 
    55:   dag OutOperandList = outs;
    56:   dag InOperandList = ins;
    57:   let AsmString = asmstr;
    58:   let Pattern = pattern;
    59:   let Constraints = cstr;
    60:   let Itinerary = itin;
    61:   let Size = 4;
    62: 
    63:   // *** Must match MCTargetDesc/HexagonBaseInfo.h ***
    64: 
    65:   // Instruction type according to the ISA.
    66:   IType Type = type;
    67:   let TSFlags{6-0} = Type.Value;
    68: 
    69:   // Solo instructions, i.e., those that cannot be in a packet with others.
    70:   bits<1> isSolo = 0;
    71:   let TSFlags{7} = isSolo;
    72:   // Packed only with A or X-type instructions.
    73:   bits<1> isSoloAX = 0;
    74:   let TSFlags{8} = isSoloAX;
    75:   // Restricts slot 1 to ALU-only instructions.
    76:   bits<1> isRestrictSlot1AOK = 0;
    77:   let TSFlags{9} = isRestrictSlot1AOK;
    78: 
    79:   // Predicated instructions.
    80:   bits<1> isPredicated = 0;
    81:   let TSFlags{10} = isPredicated;
    82:   bits<1> isPredicatedFalse = 0;
    83:   let TSFlags{11} = isPredicatedFalse;
    84:   bits<1> isPredicatedNew = 0;
    85:   let TSFlags{12} = isPredicatedNew;
    86:   bits<1> isPredicateLate = 0;
    87:   let TSFlags{13} = isPredicateLate; // Late predicate producer insn.
    88: 
    89:   // New-value insn helper fields.
    90:   bits<1> isNewValue = 0;
    91:   let TSFlags{14} = isNewValue; // New-value consumer insn.
    92:   bits<1> hasNewValue = 0;
    93:   let TSFlags{15} = hasNewValue; // New-value producer insn.
    94:   bits<3> opNewValue = 0;
    95:   let TSFlags{18-16} = opNewValue; // New-value produced operand.
    96:   bits<1> isNVStorable = 0;
    97:   let TSFlags{19} = isNVStorable; // Store that can become new-value store.
    98:   bits<1> isNVStore = 0;
    99:   let TSFlags{20} = isNVStore; // New-value store insn.
   100:   bits<1> isCVLoadable = 0;
```
- EN: Notable Hexagon symbols referenced here include HexagonBaseInfo, showing how the code connects to sibling backend components.
- CN: 这里引用的重要 Hexagon 符号包括 HexagonBaseInfo，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```tablegen
   101:   let TSFlags{21} = isCVLoadable; // Load that can become cur-value load.
   102:   bits<1> isCVLoad = 0;
   103:   let TSFlags{22} = isCVLoad; // Cur-value load insn.
   104: 
   105:   // Immediate extender helper fields.
   106:   bits<1> isExtendable = 0;
   107:   let TSFlags{23} = isExtendable; // Insn may be extended.
   108:   bits<1> isExtended = 0;
   109:   let TSFlags{24} = isExtended; // Insn must be extended.
   110:   bits<3> opExtendable = 0;
   111:   let TSFlags{27-25} = opExtendable; // Which operand may be extended.
   112:   bits<1> isExtentSigned = 0;
   113:   let TSFlags{28} = isExtentSigned; // Signed or unsigned range.
   114:   bits<5> opExtentBits = 0;
   115:   let TSFlags{33-29} = opExtentBits; //Number of bits of range before extending.
   116:   bits<2> opExtentAlign = 0;
   117:   let TSFlags{35-34} = opExtentAlign; // Alignment exponent before extending.
   118: 
   119:   bit cofMax1 = 0;
   120:   let TSFlags{36} = cofMax1;
   121:   bit cofRelax1 = 0;
   122:   let TSFlags{37} = cofRelax1;
   123:   bit cofRelax2 = 0;
   124:   let TSFlags{38} = cofRelax2;
   125: 
   126:   bit isRestrictNoSlot1Store = 0;
   127:   let TSFlags{39} = isRestrictNoSlot1Store;
   128: 
   129:   // Addressing mode for load/store instructions.
   130:   AddrModeType addrMode = NoAddrMode;
   131:   let TSFlags{42-40} = addrMode.Value;
   132: 
   133:   // Memory access size for mem access instructions (load/store)
   134:   MemAccessSize accessSize = NoMemAccess;
   135:   let TSFlags{46-43} = accessSize.Value;
   136: 
   137:   bits<1> isTaken = 0;
   138:   let TSFlags {47} = isTaken; // Branch prediction.
   139: 
   140:   bits<1> isFP = 0;
   141:   let TSFlags {48} = isFP; // Floating-point.
   142: 
   143:   bits<1> hasNewValue2 = 0;
   144:   let TSFlags{50} = hasNewValue2; // Second New-value producer insn.
   145:   bits<3> opNewValue2 = 0;
   146:   let TSFlags{53-51} = opNewValue2; // Second New-value produced operand.
   147: 
   148:   bits<1> isAccumulator = 0;
   149:   let TSFlags{54} = isAccumulator;
   150: 
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```tablegen
   151:   bits<1> prefersSlot3 = 0;
   152:   let TSFlags{55} = prefersSlot3; // Complex XU
   153: 
   154:   bits<1> hasHvxTmp = 0;
   155:   let TSFlags{56} = hasHvxTmp;  // vector register vX.tmp false-write
   156: 
   157:   bit CVINew = 0;
   158:   let TSFlags{58} = CVINew;
   159: 
   160:   bit isCVI = 0;
   161:   let TSFlags{59} = isCVI;
   162: 
   163:   bit isHVXALU = 0;
   164:   let TSFlags{60} = isHVXALU;
   165: 
   166:   bit isHVXALU2SRC = 0;
   167:   let TSFlags{61} = isHVXALU2SRC;
   168: 
   169:   bit hasUnaryRestriction = 0;
   170:   let TSFlags{62} = hasUnaryRestriction;
   171: 
   172:   // Fields used for relation models.
   173:   bit isNonTemporal = 0;
   174:   string isNT = ""; // set to "true" for non-temporal vector stores.
   175:   string BaseOpcode = "";
   176:   string CextOpcode = "";
   177:   string PredSense = "";
   178:   string PNewValue = "";
   179:   string NValueST  = "";    // Set to "true" for new-value stores.
   180:   string InputType = "";    // Input is "imm" or "reg" type.
   181:   string isFloat = "false"; // Set to "true" for the floating-point load/store.
   182:   string isBrTaken = !if(isTaken, "true", "false"); // Set to "true"/"false" for jump instructions
   183: 
   184:   let PredSense = !if(isPredicated, !if(isPredicatedFalse, "false", "true"),
   185:                                     "");
   186:   let PNewValue = !if(isPredicatedNew, "new", "");
   187:   let NValueST = !if(isNVStore, "true", "false");
   188:   let isNT = !if(isNonTemporal, "true", "false");
   189: 
   190:   let hasSideEffects = 0;
   191:   // *** Must match MCTargetDesc/HexagonBaseInfo.h ***
   192: }
   193: 
   194: class HInst<dag outs, dag ins, string asmstr, InstrItinClass itin, IType type> :
   195:       InstHexagon<outs, ins, asmstr, [], "", itin, type>;
   196: 
   197: //===----------------------------------------------------------------------===//
   198: //                         Instruction Classes Definitions +
   199: //===----------------------------------------------------------------------===//
   200: 
```
- EN: It declares types such as HInst, which carry the state or API of this component. It defines declarative TableGen records like HInst; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonBaseInfo, showing how the code connects to sibling backend components.
- CN: 这里声明了 HInst 等类型，用来承载该组件的状态或接口。 这里定义了 HInst 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonBaseInfo，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```tablegen
   201: let mayLoad = 1 in
   202: class LDInst<dag outs, dag ins, string asmstr, list<dag> pattern = [],
   203:              string cstr = "", InstrItinClass itin = LD_tc_ld_SLOT01>
   204:   : InstHexagon<outs, ins, asmstr, pattern, cstr, itin, TypeLD>, OpcodeHexagon;
   205: 
   206: class CONSTLDInst<dag outs, dag ins, string asmstr, list<dag> pattern = [],
   207:              string cstr = "", InstrItinClass itin = LD_tc_ld_SLOT01>
   208:   : InstHexagon<outs, ins, asmstr, pattern, cstr, itin, TypeLD>, OpcodeHexagon;
   209: 
   210: let mayStore = 1 in
   211: class STInst<dag outs, dag ins, string asmstr, list<dag> pattern = [],
   212:              string cstr = "", InstrItinClass itin = ST_tc_st_SLOT01>
   213:   : InstHexagon<outs, ins, asmstr, pattern, cstr, itin, TypeST>, OpcodeHexagon;
   214: 
   215: let isCodeGenOnly = 1, isPseudo = 1 in
   216: class Endloop<dag outs, dag ins, string asmstr, list<dag> pattern = [],
   217:               string cstr = "", InstrItinClass itin = tc_ENDLOOP>
   218:   : InstHexagon<outs, ins, asmstr, pattern, cstr, itin, TypeENDLOOP>,
   219:     OpcodeHexagon;
   220: 
   221: let isCodeGenOnly = 1, isPseudo = 1 in
   222: class Pseudo<dag outs, dag ins, string asmstr, list<dag> pattern = [],
   223:              string cstr = "">
   224:   : InstHexagon<outs, ins, asmstr, pattern, cstr, PSEUDO, TypePSEUDO>,
   225:     OpcodeHexagon;
   226: 
   227: let isCodeGenOnly = 1, isPseudo = 1 in
   228: class PseudoM<dag outs, dag ins, string asmstr, list<dag> pattern = [],
   229:               string cstr="">
   230:   : InstHexagon<outs, ins, asmstr, pattern, cstr, PSEUDOM, TypePSEUDO>,
   231:     OpcodeHexagon;
   232: 
   233: //===----------------------------------------------------------------------===//
   234: //                         Special Instructions -
   235: //===----------------------------------------------------------------------===//
   236: 
   237: // The 'invalid_decode' instruction is used by the disassembler to
   238: // show an instruction that didn't decode correctly.  This feature
   239: // is only leveraged in a special disassembler mode that's activated
   240: // by a command line flag.
   241: def tc_invalid : InstrItinClass;
   242: class Enc_invalid : OpcodeHexagon {
   243: }
   244: def invalid_decode : HInst<
   245: (outs ),
   246: (ins ),
   247: "<invalid>",
   248: tc_invalid, TypeALU32_2op>, Enc_invalid {
   249: let Inst{13-0} = 0b00000000000000;
   250: let Inst{31-16} = 0b0000000000000000;
```
- EN: It declares types such as LDInst, CONSTLDInst, STInst, Endloop, ... (7 total), which carry the state or API of this component. It defines declarative TableGen records like LDInst, CONSTLDInst, STInst, Endloop, Pseudo, ... (9 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 LDInst, CONSTLDInst, STInst, Endloop, ... (7 total) 等类型，用来承载该组件的状态或接口。 这里定义了 LDInst, CONSTLDInst, STInst, Endloop, Pseudo, ... (9 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 251-300 / 第 251-300 行

```tablegen
   251: let isCodeGenOnly = 1;
   252: }
   253: 
   254: //===----------------------------------------------------------------------===//
   255: //                      Duplex Instruction Class Declaration
   256: //===----------------------------------------------------------------------===//
   257: 
   258: class OpcodeDuplex {
   259:   field bits<32> Inst = ?; // Default to an invalid insn.
   260:   bits<4> IClass = 0; // ICLASS
   261:   bits<13> ISubHi = 0; // Low sub-insn
   262:   bits<13> ISubLo = 0; // High sub-insn
   263: 
   264:   let Inst{31-29} = IClass{3-1};
   265:   let Inst{13}    = IClass{0};
   266:   let Inst{15-14} = 0;
   267:   let Inst{28-16} = ISubHi;
   268:   let Inst{12-0}  = ISubLo;
   269: }
   270: 
   271: class InstDuplex<bits<4> iClass, string cstr = ""> : Instruction,
   272:                                                      OpcodeDuplex {
   273:   let Namespace = "Hexagon";
   274:   IType Type = TypeDUPLEX;  // uses slot 0,1
   275:   let isCodeGenOnly = 1;
   276:   let hasSideEffects = 0;
   277:   dag OutOperandList = (outs);
   278:   dag InOperandList = (ins);
   279:   let IClass = iClass;
   280:   let Constraints = cstr;
   281:   let Itinerary = DUPLEX;
   282:   let Size = 4;
   283: 
   284:   // *** Must match MCTargetDesc/HexagonBaseInfo.h ***
   285: 
   286:   let TSFlags{6-0} = Type.Value;
   287: 
   288:   // Predicated instructions.
   289:   bits<1> isPredicated = 0;
   290:   let TSFlags{7} = isPredicated;
   291:   bits<1> isPredicatedFalse = 0;
   292:   let TSFlags{8} = isPredicatedFalse;
   293:   bits<1> isPredicatedNew = 0;
   294:   let TSFlags{9} = isPredicatedNew;
   295: 
   296:   // New-value insn helper fields.
   297:   bits<1> isNewValue = 0;
   298:   let TSFlags{10} = isNewValue; // New-value consumer insn.
   299:   bits<1> hasNewValue = 0;
   300:   let TSFlags{11} = hasNewValue; // New-value producer insn.
```
- EN: It declares types such as OpcodeDuplex, InstDuplex, which carry the state or API of this component. It defines declarative TableGen records like OpcodeDuplex, InstDuplex; these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonBaseInfo, showing how the code connects to sibling backend components.
- CN: 这里声明了 OpcodeDuplex, InstDuplex 等类型，用来承载该组件的状态或接口。 这里定义了 OpcodeDuplex, InstDuplex 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonBaseInfo，说明了它与同级后端组件的连接关系。

### Lines 301-327 / 第 301-327 行

```tablegen
   301:   bits<3> opNewValue = 0;
   302:   let TSFlags{14-12} = opNewValue; // New-value produced operand.
   303:   bits<1> isNVStorable = 0;
   304:   let TSFlags{15} = isNVStorable; // Store that can become new-value store.
   305:   bits<1> isNVStore = 0;
   306:   let TSFlags{16} = isNVStore; // New-value store insn.
   307: 
   308:   // Immediate extender helper fields.
   309:   bits<1> isExtendable = 0;
   310:   let TSFlags{17} = isExtendable; // Insn may be extended.
   311:   bits<1> isExtended = 0;
   312:   let TSFlags{18} = isExtended; // Insn must be extended.
   313:   bits<3> opExtendable = 0;
   314:   let TSFlags{21-19} = opExtendable; // Which operand may be extended.
   315:   bits<1> isExtentSigned = 0;
   316:   let TSFlags{22} = isExtentSigned; // Signed or unsigned range.
   317:   bits<5> opExtentBits = 0;
   318:   let TSFlags{27-23} = opExtentBits; //Number of bits of range before extending.
   319:   bits<2> opExtentAlign = 0;
   320:   let TSFlags{29-28} = opExtentAlign; // Alignment exponent before extending.
   321: }
   322: 
   323: //===----------------------------------------------------------------------===//
   324: //                         Instruction Classes Definitions -
   325: //===----------------------------------------------------------------------===//
   326: 
   327: include "HexagonInstrFormatsV65.td"
```
- EN: It includes TableGen fragments such as HexagonInstrFormatsV65.td so records can build on shared target definitions. Notable Hexagon symbols referenced here include HexagonInstrFormatsV65, showing how the code connects to sibling backend components.
- CN: 这里包含了 HexagonInstrFormatsV65.td 等 TableGen 片段，使记录可以复用共享目标定义。 这里引用的重要 Hexagon 符号包括 HexagonInstrFormatsV65，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- pattern-driven instruction selection / 基于模式的指令选择

## Dependencies / 依赖关系

- Included TableGen files / 包含的 TableGen 文件: `HexagonInstrFormatsV65.td`
- Hexagon symbols / Hexagon 符号: `HexagonInstrFormats, HexagonBaseInfo, HexagonInstrFormatsV65`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
