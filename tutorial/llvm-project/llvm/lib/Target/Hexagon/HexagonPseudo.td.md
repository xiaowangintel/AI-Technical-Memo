# HexagonPseudo.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonPseudo.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): IsAsmParserOnly = 1 in
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。 重点涉及汇编/MC 层处理。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```tablegen
     1: //===--- HexagonPseudo.td -------------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: // The pat frags in the definitions below need to have a named register,
    10: // otherwise i32 will be assumed regardless of the register class. The
    11: // name of the register does not matter.
    12: def I1  : PatLeaf<(i1 PredRegs:$R)>;
    13: def I32 : PatLeaf<(i32 IntRegs:$R)>;
    14: def I64 : PatLeaf<(i64 DoubleRegs:$R)>;
    15: def F32 : PatLeaf<(f32 IntRegs:$R)>;
    16: def F64 : PatLeaf<(f64 DoubleRegs:$R)>;
    17: 
    18: let PrintMethod = "printGlobalOperand" in {
    19:   def globaladdress : Operand<i32>;
    20:   def globaladdressExt : Operand<i32>;
    21: }
    22: 
    23: let isPseudo = 1 in {
    24: let isCodeGenOnly = 0 in
    25: def A2_iconst : Pseudo<(outs IntRegs:$Rd32),
    26:     (ins s27_2Imm:$Ii), "${Rd32} = iconst(#${Ii})">;
    27: 
    28: def DUPLEX_Pseudo : InstHexagon<(outs),
    29:     (ins s32_0Imm:$offset), "DUPLEX", [], "", DUPLEX, TypePSEUDO>;
    30: }
    31: 
    32: let isExtendable = 1, opExtendable = 1, opExtentBits = 6,
    33:     isAsmParserOnly = 1 in
    34: def TFRI64_V2_ext : InstHexagon<(outs DoubleRegs:$dst),
    35:     (ins s32_0Imm:$src1, s8_0Imm:$src2),
    36:     "$dst = combine(#$src1,#$src2)", [], "",
    37:     A2_combineii.Itinerary, TypeALU32_2op>, OpcodeHexagon;
    38: 
    39: // HI/LO Instructions
    40: let isReMaterializable = 1, isMoveImm = 1, hasSideEffects = 0,
    41:     hasNewValue = 1, opNewValue = 0 in
    42: class REG_IMMED<string RegHalf, bit Rs, bits<3> MajOp, bit MinOp,
    43:                 InstHexagon rootInst>
    44:   : InstHexagon<(outs IntRegs:$dst),
    45:                 (ins u16_0Imm:$imm_value),
    46:                 "$dst"#RegHalf#" = #$imm_value", [], "",
    47:                 rootInst.Itinerary, rootInst.Type>, OpcodeHexagon {
    48:     bits<5> dst;
    49:     bits<32> imm_value;
    50: 
```
- EN: It declares types such as REG_IMMED, which carry the state or API of this component. It defines declarative TableGen records like I1, I32, I64, F32, F64, ... (11 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPseudo, showing how the code connects to sibling backend components.
- CN: 这里声明了 REG_IMMED 等类型，用来承载该组件的状态或接口。 这里定义了 I1, I32, I64, F32, F64, ... (11 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPseudo，说明了它与同级后端组件的连接关系。

### Lines 51-100 / 第 51-100 行

```tablegen
    51:     let Inst{27} = Rs;
    52:     let Inst{26-24} = MajOp;
    53:     let Inst{21} = MinOp;
    54:     let Inst{20-16} = dst;
    55:     let Inst{23-22} = imm_value{15-14};
    56:     let Inst{13-0} = imm_value{13-0};
    57: }
    58: 
    59: let isAsmParserOnly = 1 in {
    60:   def LO : REG_IMMED<".l", 0b0, 0b001, 0b1, A2_tfril>;
    61:   def HI : REG_IMMED<".h", 0b0, 0b010, 0b1, A2_tfrih>;
    62: }
    63: 
    64: let isReMaterializable = 1, isMoveImm = 1, isAsmParserOnly = 1 in {
    65:   def CONST32 : CONSTLDInst<(outs IntRegs:$Rd), (ins i32imm:$v),
    66:                 "$Rd = CONST32(#$v)", []>;
    67:   def CONST64 : CONSTLDInst<(outs DoubleRegs:$Rd), (ins i64imm:$v),
    68:                 "$Rd = CONST64(#$v)", []>;
    69: }
    70: 
    71: let hasSideEffects = 0, isReMaterializable = 1, isPseudo = 1,
    72:     isCodeGenOnly = 1 in
    73: def PS_true : InstHexagon<(outs PredRegs:$dst), (ins), "",
    74:               [(set I1:$dst, 1)], "", C2_orn.Itinerary, TypeCR>;
    75: 
    76: let hasSideEffects = 0, isReMaterializable = 1, isPseudo = 1,
    77:     isCodeGenOnly = 1 in
    78: def PS_false : InstHexagon<(outs PredRegs:$dst), (ins), "",
    79:                [(set I1:$dst, 0)], "", C2_andn.Itinerary, TypeCR>;
    80: 
    81: let Defs = [R29, R30], Uses = [R31, R30, R29], isPseudo = 1 in
    82: def ADJCALLSTACKDOWN : Pseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
    83:                               ".error \"should not emit\" ", []>;
    84: 
    85: let Defs = [R29, R30, R31], Uses = [R29], isPseudo = 1 in
    86: def ADJCALLSTACKUP : Pseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
    87:                              ".error \"should not emit\" ", []>;
    88: 
    89: 
    90: let isBranch = 1, isTerminator = 1, hasSideEffects = 0,
    91:     Defs = [PC, LC0], Uses = [SA0, LC0] in {
    92: def ENDLOOP0 : Endloop<(outs), (ins b30_2Imm:$offset),
    93:                        ":endloop0",
    94:                        []>;
    95: }
    96: 
    97: let isBranch = 1, isTerminator = 1, hasSideEffects = 0,
    98:     Defs = [PC, LC1], Uses = [SA1, LC1] in {
    99: def ENDLOOP1 : Endloop<(outs), (ins b30_2Imm:$offset),
   100:                        ":endloop1",
```
- EN: It defines declarative TableGen records like LO, HI, CONST32, CONST64, PS_true, ... (10 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 LO, HI, CONST32, CONST64, PS_true, ... (10 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 101-150 / 第 101-150 行

```tablegen
   101:                        []>;
   102: }
   103: 
   104: let isBranch = 1, isTerminator = 1, hasSideEffects = 0,
   105:     Defs = [PC, LC0, LC1], Uses = [SA0, SA1, LC0, LC1] in {
   106: def ENDLOOP01 : Endloop<(outs), (ins b30_2Imm:$offset),
   107:                         ":endloop01",
   108:                         []>;
   109: }
   110: 
   111: let isExtendable = 1, isExtentSigned = 1, opExtentBits = 9, opExtentAlign = 2,
   112:     opExtendable = 0, hasSideEffects = 0 in
   113: class LOOP_iBase<string mnemonic, InstHexagon rootInst>
   114:          : InstHexagon <(outs), (ins b30_2Imm:$offset, u10_0Imm:$src2),
   115:            mnemonic#"($offset,#$src2)",
   116:            [], "", rootInst.Itinerary, rootInst.Type>, OpcodeHexagon {
   117:     bits<9> offset;
   118:     bits<10> src2;
   119: 
   120:     let IClass = 0b0110;
   121: 
   122:     let Inst{27-22} = 0b100100;
   123:     let Inst{21} = !if (!eq(mnemonic, "loop0"), 0b0, 0b1);
   124:     let Inst{20-16} = src2{9-5};
   125:     let Inst{12-8} = offset{8-4};
   126:     let Inst{7-5} = src2{4-2};
   127:     let Inst{4-3} = offset{3-2};
   128:     let Inst{1-0} = src2{1-0};
   129: }
   130: 
   131: let isExtendable = 1, isExtentSigned = 1, opExtentBits = 9, opExtentAlign = 2,
   132:     opExtendable = 0, hasSideEffects = 0 in
   133: class LOOP_rBase<string mnemonic, InstHexagon rootInst>
   134:          : InstHexagon<(outs), (ins b30_2Imm:$offset, IntRegs:$src2),
   135:            mnemonic#"($offset,$src2)",
   136:            [], "", rootInst.Itinerary, rootInst.Type>, OpcodeHexagon {
   137:     bits<9> offset;
   138:     bits<5> src2;
   139: 
   140:     let IClass = 0b0110;
   141: 
   142:     let Inst{27-22} = 0b000000;
   143:     let Inst{21} = !if (!eq(mnemonic, "loop0"), 0b0, 0b1);
   144:     let Inst{20-16} = src2;
   145:     let Inst{12-8} = offset{8-4};
   146:     let Inst{4-3} = offset{3-2};
   147:   }
   148: 
   149: let Defs = [SA0, LC0, USR], isCodeGenOnly = 1, isExtended = 1,
   150:     opExtendable = 0 in {
```
- EN: It declares types such as LOOP_iBase, LOOP_rBase, which carry the state or API of this component. It defines declarative TableGen records like ENDLOOP01, LOOP_iBase, LOOP_rBase; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 LOOP_iBase, LOOP_rBase 等类型，用来承载该组件的状态或接口。 这里定义了 ENDLOOP01, LOOP_iBase, LOOP_rBase 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 151-200 / 第 151-200 行

```tablegen
   151:   def J2_loop0iext : LOOP_iBase<"loop0", J2_loop0i>;
   152:   def J2_loop1iext : LOOP_iBase<"loop1", J2_loop1i>;
   153: }
   154: 
   155: // Interestingly only loop0's appear to set usr.lpcfg
   156: let Defs = [SA1, LC1], isCodeGenOnly = 1, isExtended = 1, opExtendable = 0 in {
   157:   def J2_loop0rext : LOOP_rBase<"loop0", J2_loop0r>;
   158:   def J2_loop1rext : LOOP_rBase<"loop1", J2_loop1r>;
   159: }
   160: 
   161: let isCall = 1, hasSideEffects = 1, isPredicable = 0,
   162:     isExtended = 0, isExtendable = 1, opExtendable = 0,
   163:     isExtentSigned = 1, opExtentBits = 24, opExtentAlign = 2 in
   164: class T_Call<string ExtStr>
   165:   : InstHexagon<(outs), (ins a30_2Imm:$dst),
   166:       "call " # ExtStr # "$dst", [], "", J2_call.Itinerary, TypeJ>,
   167:     OpcodeHexagon {
   168:   let BaseOpcode = "call";
   169:   bits<24> dst;
   170: 
   171:   let IClass = 0b0101;
   172:   let Inst{27-25} = 0b101;
   173:   let Inst{24-16,13-1} = dst{23-2};
   174:   let Inst{0} = 0b0;
   175: }
   176: 
   177: let isCodeGenOnly = 1, isCall = 1, hasSideEffects = 1, Defs = [R16],
   178:     isPredicable = 0 in
   179: def CALLProfile :  T_Call<"">;
   180: 
   181: let isCodeGenOnly = 1, isCall = 1, hasSideEffects = 1,
   182:     Defs = [PC, R31, R6, R7, P0] in
   183: def PS_call_stk : T_Call<"">;
   184: 
   185: // This pseudo instruction is used to replace int_hexagon_instrprof_custom intrinsic
   186: // with a call to custom handler passed as the first argument to the intrinsic.
   187: 
   188: // Please Note:
   189: // 1) The call to the custom handler is being treated as a special one as the
   190: //    callee is responsible for saving and restoring all the registers it needs
   191: //    to modify. This includes caller saved registers as well as r0-r5 argument
   192: //    registers. This is done to reduce the impact of instrumentation on the
   193: //    code being instrumented/profiled.
   194: // 2) R14, R15 and R28 are reserved for PLT handling and therefore are
   195: //    part of the def list.
   196: // 3) R0 is used to pass the unique id associated with an instrumentation site
   197: //    to the handler.
   198: // 4) All the other registers (R29, R30, R31, PC) get modified by the call
   199: //    instruction.
   200: 
```
- EN: It declares types such as T_Call, which carry the state or API of this component. It defines declarative TableGen records like J2_loop0iext, J2_loop1iext, J2_loop0rext, J2_loop1rext, T_Call, ... (7 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 T_Call 等类型，用来承载该组件的状态或接口。 这里定义了 J2_loop0iext, J2_loop1iext, J2_loop0rext, J2_loop1rext, T_Call, ... (7 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```tablegen
   201: // TODO: It may be a good idea to add a separate pseudo instruction for
   202: // static relocation which doesn't need to reserve r14, r15 and r28.
   203: 
   204: let hasSideEffects = 1, isCall = 1, Defs = [R0, R14, R15, R28, R29, R30, R31, PC] in
   205: def PS_call_instrprof_custom :  Pseudo<(outs), (ins s32_0Imm:$dst, u32_0Imm:$Ii), "">;
   206: 
   207: // Call, no return.
   208: let isCall = 1, hasSideEffects = 1, cofMax1 = 1, isCodeGenOnly = 1 in
   209: def PS_callr_nr: InstHexagon<(outs), (ins IntRegs:$Rs),
   210:     "callr $Rs", [], "", J2_callr.Itinerary, TypeJ>, OpcodeHexagon {
   211:     bits<5> Rs;
   212:     bits<2> Pu;
   213:     let isPredicatedFalse = 1;
   214: 
   215:     let IClass = 0b0101;
   216:     let Inst{27-21} = 0b0000101;
   217:     let Inst{20-16} = Rs;
   218:   }
   219: 
   220: let isCall = 1, hasSideEffects = 1,
   221:     isExtended = 0, isExtendable = 1, opExtendable = 0, isCodeGenOnly = 1,
   222:     BaseOpcode = "PS_call_nr", isExtentSigned = 1, opExtentAlign = 2 in
   223: class Call_nr<bits<5> nbits, bit isFalse, dag iops,
   224:               InstrItinClass itin>
   225:   : Pseudo<(outs), iops, "">, PredRel {
   226:     bits<2> Pu;
   227:     bits<17> dst;
   228:     let opExtentBits = nbits;
   229:     let isPredicable = 0;  // !if(isPred, 0, 1);
   230:     let isPredicated = 0;  // isPred;
   231:     let isPredicatedFalse = isFalse;
   232:     let Itinerary = itin;
   233: }
   234: 
   235: def PS_call_nr : Call_nr<24, 0, (ins s32_0Imm:$Ii), J2_call.Itinerary>;
   236: //def PS_call_nrt: Call_nr<17, 1, 0, (ins PredRegs:$Pu, s32_0Imm:$dst),
   237: //                         J2_callt.Itinerary>;
   238: //def PS_call_nrf: Call_nr<17, 1, 1, (ins PredRegs:$Pu, s32_0Imm:$dst),
   239: //                         J2_callf.Itinerary>;
   240: 
   241: let isBranch = 1, isIndirectBranch = 1, isBarrier = 1, Defs = [PC],
   242:     isPredicable = 1, hasSideEffects = 0, InputType = "reg",
   243:     cofMax1 = 1 in
   244: class T_JMPr <InstHexagon rootInst>
   245:   :  InstHexagon<(outs), (ins IntRegs:$dst), "jumpr $dst", [],
   246:                  "", rootInst.Itinerary, rootInst.Type>, OpcodeHexagon {
   247:     bits<5> dst;
   248: 
   249:     let IClass = 0b0101;
   250:     let Inst{27-21} = 0b0010100;
```
- EN: It declares types such as Call_nr, T_JMPr, which carry the state or API of this component. It defines declarative TableGen records like PS_call_instrprof_custom, PS_callr_nr, Call_nr, PS_call_nr, T_JMPr; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Call_nr, T_JMPr 等类型，用来承载该组件的状态或接口。 这里定义了 PS_call_instrprof_custom, PS_callr_nr, Call_nr, PS_call_nr, T_JMPr 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 251-300 / 第 251-300 行

```tablegen
   251:     let Inst{20-16} = dst;
   252: }
   253: 
   254: // A return through builtin_eh_return.
   255: let isReturn = 1, isTerminator = 1, isBarrier = 1, hasSideEffects = 0,
   256:     isCodeGenOnly = 1, Defs = [PC], Uses = [R28], isPredicable = 0 in
   257: def EH_RETURN_JMPR : T_JMPr<J2_jumpr>;
   258: 
   259: // Indirect tail-call.
   260: let isPseudo = 1, isCall = 1, isReturn = 1, isBarrier = 1, isPredicable = 0,
   261:     isTerminator = 1, isCodeGenOnly = 1 in
   262: def PS_tailcall_r : T_JMPr<J2_jumpr>;
   263: 
   264: //
   265: // Direct tail-calls.
   266: let isPseudo = 1, isCall = 1, isReturn = 1, isBarrier = 1, isPredicable = 0,
   267:     isTerminator = 1, isCodeGenOnly = 1 in
   268: def PS_tailcall_i : Pseudo<(outs), (ins a30_2Imm:$dst), "", []>;
   269: 
   270: let isCodeGenOnly = 1, isPseudo = 1, Uses = [R30], hasSideEffects = 0 in
   271: def PS_aligna : Pseudo<(outs IntRegs:$Rd), (ins u32_0Imm:$A), "", []>;
   272: 
   273: // Generate frameindex addresses. The main reason for the offset operand is
   274: // that every instruction that is allowed to have frame index as an operand
   275: // will then have that operand followed by an immediate operand (the offset).
   276: // This simplifies the frame-index elimination code.
   277: //
   278: let isMoveImm = 1, isAsCheapAsAMove = 1, isReMaterializable = 1,
   279:     isPseudo = 1, isCodeGenOnly = 1, hasSideEffects = 0, isExtendable = 1,
   280:     isExtentSigned = 1, opExtentBits = 16, opExtentAlign = 0 in {
   281:   let opExtendable = 2 in
   282:   def PS_fi  : Pseudo<(outs IntRegs:$Rd),
   283:                       (ins IntRegs:$fi, s32_0Imm:$off), "">;
   284:   let opExtendable = 3 in
   285:   def PS_fia : Pseudo<(outs IntRegs:$Rd),
   286:                       (ins IntRegs:$Rs, IntRegs:$fi, s32_0Imm:$off), "">;
   287: }
   288: 
   289: class CondStr<string CReg, bit True, bit New> {
   290:   string S = "if (" # !if(True,"","!") # CReg # !if(New,".new","") # ") ";
   291: }
   292: class JumpOpcStr<string Mnemonic, bit Taken> {
   293:   string S = Mnemonic # !if(Taken, ":t", ":nt");
   294: }
   295: let isBranch = 1, isIndirectBranch = 1, Defs = [PC], isPredicated = 1,
   296:     hasSideEffects = 0, InputType = "reg", cofMax1 = 1 in
   297: class T_JMPr_c <bit PredNot, bit isPredNew, bit isTak, InstHexagon rootInst>
   298:   :  InstHexagon<(outs), (ins PredRegs:$src, IntRegs:$dst),
   299:                  CondStr<"$src", !if(PredNot,0,1), isPredNew>.S #
   300:                  JumpOpcStr<"jumpr", isTak>.S # " $dst",
```
- EN: It declares types such as CondStr, JumpOpcStr, T_JMPr_c, which carry the state or API of this component. It defines declarative TableGen records like EH_RETURN_JMPR, PS_tailcall_r, PS_tailcall_i, PS_aligna, PS_fi, ... (9 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 CondStr, JumpOpcStr, T_JMPr_c 等类型，用来承载该组件的状态或接口。 这里定义了 EH_RETURN_JMPR, PS_tailcall_r, PS_tailcall_i, PS_aligna, PS_fi, ... (9 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```tablegen
   301:                  [], "", rootInst.Itinerary, rootInst.Type>, OpcodeHexagon {
   302: 
   303:     let isTaken = isTak;
   304:     let isPredicatedFalse = PredNot;
   305:     let isPredicatedNew = isPredNew;
   306:     bits<2> src;
   307:     bits<5> dst;
   308: 
   309:     let IClass = 0b0101;
   310: 
   311:     let Inst{27-22} = 0b001101;
   312:     let Inst{21} = PredNot;
   313:     let Inst{20-16} = dst;
   314:     let Inst{12} = isTak;
   315:     let Inst{11} = isPredNew;
   316:     let Inst{9-8} = src;
   317: }
   318: 
   319: let isTerminator = 1, hasSideEffects = 0, isReturn = 1, isCodeGenOnly = 1,
   320:     isBarrier = 1, BaseOpcode = "JMPret" in {
   321:   def PS_jmpret : T_JMPr<J2_jumpr>, PredNewRel;
   322:   def PS_jmprett : T_JMPr_c<0, 0, 0, J2_jumprt>, PredNewRel;
   323:   def PS_jmpretf : T_JMPr_c<1, 0, 0, J2_jumprf>, PredNewRel;
   324:   def PS_jmprettnew : T_JMPr_c<0, 1, 0, J2_jumprtnew>, PredNewRel;
   325:   def PS_jmpretfnew : T_JMPr_c<1, 1, 0, J2_jumprfnew>, PredNewRel;
   326:   def PS_jmprettnewpt : T_JMPr_c<0, 1, 1, J2_jumprtnewpt>, PredNewRel;
   327:   def PS_jmpretfnewpt : T_JMPr_c<1, 1, 1, J2_jumprfnewpt>, PredNewRel;
   328: }
   329: 
   330: //defm V6_vtran2x2_map : HexagonMapping<(outs HvxVR:$Vy32, HvxVR:$Vx32), (ins HvxVR:$Vx32in, IntRegs:$Rt32), "vtrans2x2(${Vy32},${Vx32},${Rt32})", (V6_vshuff HvxVR:$Vy32, HvxVR:$Vx32, HvxVR:$Vx32in, IntRegs:$Rt32)>;
   331: 
   332: // The reason for the custom inserter is to record all ALLOCA instructions
   333: // in MachineFunctionInfo.
   334: let Defs = [R29], hasSideEffects = 1 in
   335: def PS_alloca: Pseudo <(outs IntRegs:$Rd),
   336:                        (ins IntRegs:$Rs, u32_0Imm:$A), "", []>;
   337: 
   338: // Load predicate.
   339: let isExtendable = 1, opExtendable = 2, isExtentSigned = 1, opExtentBits = 13,
   340:     isCodeGenOnly = 1, isPseudo = 1, hasSideEffects = 0 in
   341: def LDriw_pred : LDInst<(outs PredRegs:$dst),
   342:                         (ins IntRegs:$addr, s32_0Imm:$off),
   343:                         ".error \"should not emit\"", []>;
   344: 
   345: // Load modifier.
   346: let isExtendable = 1, opExtendable = 2, isExtentSigned = 1, opExtentBits = 13,
   347:     isCodeGenOnly = 1, isPseudo = 1, hasSideEffects = 0 in
   348: def LDriw_ctr : LDInst<(outs CtrRegs:$dst),
   349:                         (ins IntRegs:$addr, s32_0Imm:$off),
   350:                         ".error \"should not emit\"", []>;
```
- EN: It defines declarative TableGen records like PS_jmpret, PS_jmprett, PS_jmpretf, PS_jmprettnew, PS_jmpretfnew, ... (10 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMapping, showing how the code connects to sibling backend components.
- CN: 这里定义了 PS_jmpret, PS_jmprett, PS_jmpretf, PS_jmprettnew, PS_jmpretfnew, ... (10 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMapping，说明了它与同级后端组件的连接关系。

### Lines 351-400 / 第 351-400 行

```tablegen
   351: 
   352: let isCodeGenOnly = 1, isPseudo = 1, hasSideEffects = 1 in {
   353:   def PS_readcr : InstHexagon<(outs IntRegs:$Rd32), (ins CtrRegs:$Cs32),
   354:                               ".error \"should not emit\" ", [], "",
   355:                               A2_tfrcrr.Itinerary, TypeCR>;
   356: 
   357:   def PS_readcr64
   358:       : InstHexagon<(outs DoubleRegs:$Rdd32), (ins CtrRegs64:$Css32),
   359:                     ".error \"should not emit\" ", [], "", A4_tfrcpp.Itinerary,
   360:                     TypeCR>;
   361: }
   362: 
   363: let isCodeGenOnly = 1, isPseudo = 1 in
   364: def PS_pselect: InstHexagon<(outs DoubleRegs:$Rd),
   365:       (ins PredRegs:$Pu, DoubleRegs:$Rs, DoubleRegs:$Rt),
   366:       ".error \"should not emit\" ", [], "", A2_tfrpt.Itinerary, TypeALU32_2op>;
   367: 
   368: let isBranch = 1, isBarrier = 1, Defs = [PC], hasSideEffects = 0,
   369:     isPredicable = 1,
   370:     isExtendable = 1, opExtendable = 0, isExtentSigned = 1,
   371:     opExtentBits = 24, opExtentAlign = 2, InputType = "imm" in
   372: class T_JMP: InstHexagon<(outs), (ins b30_2Imm:$dst),
   373:       "jump $dst",
   374:       [], "", J2_jump.Itinerary, TypeJ>, OpcodeHexagon {
   375:     bits<24> dst;
   376:     let IClass = 0b0101;
   377: 
   378:     let Inst{27-25} = 0b100;
   379:     let Inst{24-16} = dst{23-15};
   380:     let Inst{13-1} = dst{14-2};
   381: }
   382: 
   383: // Restore registers and dealloc return function call.
   384: let isCall = 1, isBarrier = 1, isReturn = 1, isTerminator = 1,
   385:     Defs = [R29, R30, R31, PC], isPredicable = 0, isAsmParserOnly = 1 in {
   386:   def RESTORE_DEALLOC_RET_JMP_V4 : T_JMP;
   387: 
   388:   let isExtended = 1, opExtendable = 0 in
   389:   def RESTORE_DEALLOC_RET_JMP_V4_EXT : T_JMP;
   390: 
   391:   let Defs = [R14, R15, R28, R29, R30, R31, PC] in {
   392:     def RESTORE_DEALLOC_RET_JMP_V4_PIC : T_JMP;
   393: 
   394:     let isExtended = 1, opExtendable = 0 in
   395:     def RESTORE_DEALLOC_RET_JMP_V4_EXT_PIC : T_JMP;
   396:   }
   397: }
   398: 
   399: // Restore registers and dealloc frame before a tail call.
   400: let isCall = 1, Defs = [R29, R30, R31, PC], isAsmParserOnly = 1 in {
```
- EN: It declares types such as T_JMP:, which carry the state or API of this component. It defines declarative TableGen records like PS_readcr, PS_readcr64, PS_pselect, T_JMP, RESTORE_DEALLOC_RET_JMP_V4, ... (8 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 T_JMP: 等类型，用来承载该组件的状态或接口。 这里定义了 PS_readcr, PS_readcr64, PS_pselect, T_JMP, RESTORE_DEALLOC_RET_JMP_V4, ... (8 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 401-450 / 第 401-450 行

```tablegen
   401:   def RESTORE_DEALLOC_BEFORE_TAILCALL_V4 : T_Call<"">, PredRel;
   402: 
   403:   let isExtended = 1, opExtendable = 0 in
   404:   def RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT : T_Call<"">, PredRel;
   405: 
   406:   let Defs = [R14, R15, R28, R29, R30, R31, PC] in {
   407:     def RESTORE_DEALLOC_BEFORE_TAILCALL_V4_PIC : T_Call<"">, PredRel;
   408: 
   409:     let isExtended = 1, opExtendable = 0 in
   410:     def RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT_PIC : T_Call<"">, PredRel;
   411:   }
   412: }
   413: 
   414: // Save registers function call.
   415: let isCall = 1, Uses = [R29, R31], isAsmParserOnly = 1 in {
   416:   def SAVE_REGISTERS_CALL_V4 : T_Call<"">, PredRel;
   417: 
   418:   let isExtended = 1, opExtendable = 0 in
   419:   def SAVE_REGISTERS_CALL_V4_EXT : T_Call<"">, PredRel;
   420: 
   421:   let Defs = [P0] in
   422:   def SAVE_REGISTERS_CALL_V4STK : T_Call<"">, PredRel;
   423: 
   424:   let Defs = [P0], isExtended = 1, opExtendable = 0 in
   425:   def SAVE_REGISTERS_CALL_V4STK_EXT : T_Call<"">, PredRel;
   426: 
   427:   let Defs = [R14, R15, R28] in
   428:   def SAVE_REGISTERS_CALL_V4_PIC : T_Call<"">, PredRel;
   429: 
   430:   let Defs = [R14, R15, R28], isExtended = 1, opExtendable = 0 in
   431:   def SAVE_REGISTERS_CALL_V4_EXT_PIC : T_Call<"">, PredRel;
   432: 
   433:   let Defs = [R14, R15, R28, P0] in
   434:   def SAVE_REGISTERS_CALL_V4STK_PIC : T_Call<"">, PredRel;
   435: 
   436:   let Defs = [R14, R15, R28, P0], isExtended = 1, opExtendable = 0 in
   437:   def SAVE_REGISTERS_CALL_V4STK_EXT_PIC : T_Call<"">, PredRel;
   438: }
   439: 
   440: let Predicates = [UseHVX], isPseudo = 1, isCodeGenOnly = 1,
   441:                  hasSideEffects = 0, hasPostISelHook = 1 in
   442: class Vsplatr_template : InstHexagon<(outs HvxVR:$Vd), (ins IntRegs:$Rs),
   443:                          "", [], "", V6_lvsplatw.Itinerary, V6_lvsplatw.Type>;
   444: def PS_vsplatrb: Vsplatr_template;
   445: def PS_vsplatrh: Vsplatr_template;
   446: def PS_vsplatrw: Vsplatr_template;
   447: 
   448: let Predicates = [UseHVX], isPseudo = 1, isCodeGenOnly = 1,
   449:                  hasSideEffects = 0, hasPostISelHook = 1 in
   450: class Vsplati_template : InstHexagon<(outs HvxVR:$Vd), (ins s32_0Imm:$Val),
```
- EN: It declares types such as Vsplatr_template, Vsplati_template, which carry the state or API of this component. It defines declarative TableGen records like RESTORE_DEALLOC_BEFORE_TAILCALL_V4, RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT, RESTORE_DEALLOC_BEFORE_TAILCALL_V4_PIC, RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT_PIC, SAVE_REGISTERS_CALL_V4, ... (17 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Vsplatr_template, Vsplati_template 等类型，用来承载该组件的状态或接口。 这里定义了 RESTORE_DEALLOC_BEFORE_TAILCALL_V4, RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT, RESTORE_DEALLOC_BEFORE_TAILCALL_V4_PIC, RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT_PIC, SAVE_REGISTERS_CALL_V4, ... (17 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 451-500 / 第 451-500 行

```tablegen
   451:                          "", [], "", V6_lvsplatw.Itinerary, V6_lvsplatw.Type>;
   452: def PS_vsplatib: Vsplati_template;
   453: def PS_vsplatih: Vsplati_template;
   454: def PS_vsplatiw: Vsplati_template;
   455: 
   456: // Vector store pseudos
   457: let Predicates = [HasV60,UseHVX], isPseudo = 1, isCodeGenOnly = 1,
   458:     mayStore = 1, accessSize = HVXVectorAccess, hasSideEffects = 0 in
   459: class STriv_template<RegisterClass RC, InstHexagon rootInst>
   460:   : InstHexagon<(outs), (ins IntRegs:$addr, s32_0Imm:$off, RC:$src),
   461:     "", [], "", rootInst.Itinerary, rootInst.Type>;
   462: 
   463: def PS_vstorerv_ai: STriv_template<HvxVR, V6_vS32b_ai>,
   464:       Requires<[HasV60,UseHVX]>;
   465: def PS_vstorerv_nt_ai: STriv_template<HvxVR, V6_vS32b_nt_ai>,
   466:       Requires<[HasV60,UseHVX]>;
   467: def PS_vstorerw_ai: STriv_template<HvxWR, V6_vS32b_ai>,
   468:       Requires<[HasV60,UseHVX]>;
   469: def PS_vstorerw_nt_ai: STriv_template<HvxWR, V6_vS32b_nt_ai>,
   470:       Requires<[HasV60,UseHVX]>;
   471: 
   472: let isPseudo = 1, isCodeGenOnly = 1, mayStore = 1, hasSideEffects = 0 in
   473: def PS_vstorerq_ai: Pseudo<(outs),
   474:       (ins IntRegs:$Rs, s32_0Imm:$Off, HvxQR:$Qt), "", []>,
   475:       Requires<[HasV60,UseHVX]>;
   476: 
   477: // Vector load pseudos
   478: let Predicates = [HasV60, UseHVX], isPseudo = 1, isCodeGenOnly = 1,
   479:     mayLoad = 1, accessSize = HVXVectorAccess, hasSideEffects = 0 in
   480: class LDriv_template<RegisterClass RC, InstHexagon rootInst>
   481:   : InstHexagon<(outs RC:$dst), (ins IntRegs:$addr, s32_0Imm:$off),
   482:     "", [], "", rootInst.Itinerary, rootInst.Type>;
   483: 
   484: def PS_vloadrv_ai: LDriv_template<HvxVR, V6_vL32b_ai>,
   485:       Requires<[HasV60,UseHVX]>;
   486: def PS_vloadrv_nt_ai: LDriv_template<HvxVR, V6_vL32b_nt_ai>,
   487:       Requires<[HasV60,UseHVX]>;
   488: def PS_vloadrw_ai: LDriv_template<HvxWR, V6_vL32b_ai>,
   489:       Requires<[HasV60,UseHVX]>;
   490: def PS_vloadrw_nt_ai: LDriv_template<HvxWR, V6_vL32b_nt_ai>,
   491:       Requires<[HasV60,UseHVX]>;
   492: 
   493: let isPseudo = 1, isCodeGenOnly = 1, mayLoad = 1, hasSideEffects = 0 in
   494: def PS_vloadrq_ai: Pseudo<(outs HvxQR:$Qd),
   495:       (ins IntRegs:$Rs, s32_0Imm:$Off), "", []>,
   496:       Requires<[HasV60,UseHVX]>;
   497: 
   498: 
   499: let isCodeGenOnly = 1, isPseudo = 1, hasSideEffects = 0 in
   500: class VSELInst<dag outs, dag ins, InstHexagon rootInst>
```
- EN: It declares types such as STriv_template, LDriv_template, VSELInst, which carry the state or API of this component. It defines declarative TableGen records like PS_vsplatib, PS_vsplatih, PS_vsplatiw, STriv_template, PS_vstorerv_ai, ... (16 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 STriv_template, LDriv_template, VSELInst 等类型，用来承载该组件的状态或接口。 这里定义了 PS_vsplatib, PS_vsplatih, PS_vsplatiw, STriv_template, PS_vstorerv_ai, ... (16 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 501-550 / 第 501-550 行

```tablegen
   501:   : InstHexagon<outs, ins, "", [], "", rootInst.Itinerary, rootInst.Type>;
   502: 
   503: def PS_vselect: VSELInst<(outs HvxVR:$dst),
   504:       (ins PredRegs:$src1, HvxVR:$src2, HvxVR:$src3), V6_vcmov>,
   505:       Requires<[HasV60,UseHVX]>;
   506: def PS_wselect: VSELInst<(outs HvxWR:$dst),
   507:       (ins PredRegs:$src1, HvxWR:$src2, HvxWR:$src3), V6_vccombine>,
   508:       Requires<[HasV60,UseHVX]>;
   509: 
   510: let hasSideEffects = 0, isReMaterializable = 1, isPseudo = 1,
   511:     isCodeGenOnly = 1 in {
   512:   def PS_qtrue:  InstHexagon<(outs HvxQR:$Qd), (ins), "", [], "",
   513:                  V6_veqw.Itinerary, TypeCVI_VA>;
   514:   def PS_qfalse: InstHexagon<(outs HvxQR:$Qd), (ins), "", [], "",
   515:                  V6_vgtw.Itinerary, TypeCVI_VA>;
   516:   def PS_vdd0:   InstHexagon<(outs HvxWR:$Vd), (ins), "", [], "",
   517:                  V6_vsubw_dv.Itinerary, TypeCVI_VA_DV>;
   518: }
   519: 
   520: // Store predicate.
   521: let isExtendable = 1, opExtendable = 1, isExtentSigned = 1, opExtentBits = 13,
   522:     isCodeGenOnly = 1, isPseudo = 1, hasSideEffects = 0 in
   523: def STriw_pred : STInst<(outs),
   524:       (ins IntRegs:$addr, s32_0Imm:$off, PredRegs:$src1),
   525:       ".error \"should not emit\"", []>;
   526: // Store modifier.
   527: let isExtendable = 1, opExtendable = 1, isExtentSigned = 1, opExtentBits = 13,
   528:     isCodeGenOnly = 1, isPseudo = 1, hasSideEffects = 0 in
   529: def STriw_ctr : STInst<(outs),
   530:       (ins IntRegs:$addr, s32_0Imm:$off, CtrRegs:$src1),
   531:       ".error \"should not emit\"", []>;
   532: 
   533: let isExtendable = 1, opExtendable = 1, opExtentBits = 6,
   534:     isAsmParserOnly = 1 in
   535: def TFRI64_V4 : InstHexagon<(outs DoubleRegs:$dst),
   536:     (ins u64_0Imm:$src1),
   537:     "$dst = #$src1", [], "",
   538:     A2_combineii.Itinerary, TypeALU32_2op>, OpcodeHexagon;
   539: 
   540: // Hexagon doesn't have a vector multiply with C semantics.
   541: // Instead, generate a pseudo instruction that gets expanded into two
   542: // scalar MPYI instructions.
   543: // This is expanded by ExpandPostRAPseudos.
   544: let isPseudo = 1 in
   545: def PS_vmulw : PseudoM<(outs DoubleRegs:$Rd),
   546:       (ins DoubleRegs:$Rs, DoubleRegs:$Rt), "", []>;
   547: 
   548: let isPseudo = 1 in
   549: def PS_vmulw_acc : PseudoM<(outs DoubleRegs:$Rd),
   550:       (ins DoubleRegs:$Rx, DoubleRegs:$Rs, DoubleRegs:$Rt), "", [],
```
- EN: It defines declarative TableGen records like PS_vselect, PS_wselect, PS_qtrue, PS_qfalse, PS_vdd0, ... (10 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 PS_vselect, PS_wselect, PS_qtrue, PS_qfalse, PS_vdd0, ... (10 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 551-600 / 第 551-600 行

```tablegen
   551:       "$Rd = $Rx">;
   552: 
   553: def DuplexIClass0:  InstDuplex < 0 >;
   554: def DuplexIClass1:  InstDuplex < 1 >;
   555: def DuplexIClass2:  InstDuplex < 2 >;
   556: let isExtendable = 1 in {
   557:   def DuplexIClass3:  InstDuplex < 3 >;
   558:   def DuplexIClass4:  InstDuplex < 4 >;
   559:   def DuplexIClass5:  InstDuplex < 5 >;
   560:   def DuplexIClass6:  InstDuplex < 6 >;
   561:   def DuplexIClass7:  InstDuplex < 7 >;
   562: }
   563: def DuplexIClass8:  InstDuplex < 8 >;
   564: def DuplexIClass9:  InstDuplex < 9 >;
   565: def DuplexIClassA:  InstDuplex < 0xA >;
   566: def DuplexIClassB:  InstDuplex < 0xB >;
   567: def DuplexIClassC:  InstDuplex < 0xC >;
   568: def DuplexIClassD:  InstDuplex < 0xD >;
   569: def DuplexIClassE:  InstDuplex < 0xE >;
   570: def DuplexIClassF:  InstDuplex < 0xF >;
   571: 
   572: // Pseudos for circular buffer instructions. These are needed in order to
   573: // allocate the correct pair of CSx and Mx registers.
   574: multiclass NewCircularLoad<RegisterClass RC, MemAccessSize MS> {
   575: 
   576: let isCodeGenOnly = 1, isPseudo = 1, Defs = [CS], Uses = [CS],
   577:     addrMode = PostInc, accessSize = MS, hasSideEffects = 0 in {
   578:   // Use timing class of L2_loadrb_pci.
   579:   def NAME#_pci : LDInst<(outs RC:$Rd32, IntRegs:$Rx32),
   580:        (ins IntRegs:$Rx32in, s4_0Imm:$Ii, ModRegs:$Mu2, IntRegs:$Cs),
   581:        ".error \"should not emit\" ", [], "$Rx32 = $Rx32in", tc_5ceb2f9e>;
   582: 
   583:   // Use timing class of L2_loadrb_pcr.
   584:   def NAME#_pcr : LDInst<(outs RC:$Rd32, IntRegs:$Rx32),
   585:        (ins IntRegs:$Rx32in, ModRegs:$Mu2, IntRegs:$Cs),
   586:        ".error \"should not emit\" ", [], "$Rx32 = $Rx32in", tc_075c8dd8>;
   587: }
   588: }
   589: 
   590: defm PS_loadrub : NewCircularLoad<IntRegs, ByteAccess>;
   591: defm PS_loadrb : NewCircularLoad<IntRegs, ByteAccess>;
   592: defm PS_loadruh : NewCircularLoad<IntRegs, HalfWordAccess>;
   593: defm PS_loadrh : NewCircularLoad<IntRegs, HalfWordAccess>;
   594: defm PS_loadri : NewCircularLoad<IntRegs, WordAccess>;
   595: defm PS_loadrd : NewCircularLoad<DoubleRegs, DoubleWordAccess>;
   596: 
   597: multiclass NewCircularStore<RegisterClass RC, MemAccessSize MS> {
   598: 
   599: let isCodeGenOnly = 1, isPseudo = 1, Defs = [CS], Uses = [CS],
   600:     addrMode = PostInc, accessSize = MS, hasSideEffects = 0 in {
```
- EN: It defines declarative TableGen records like DuplexIClass0, DuplexIClass1, DuplexIClass2, DuplexIClass3, DuplexIClass4, ... (25 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里定义了 DuplexIClass0, DuplexIClass1, DuplexIClass2, DuplexIClass3, DuplexIClass4, ... (25 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 601-633 / 第 601-633 行

```tablegen
   601:   // Use timing class of S2_storerb_pci.
   602:   def NAME#_pci : STInst<(outs IntRegs:$Rx32),
   603:        (ins IntRegs:$Rx32in, s4_0Imm:$Ii, ModRegs:$Mu2, RC:$Rt32, IntRegs:$Cs),
   604:        ".error \"should not emit\" ", [], "$Rx32 = $Rx32in", tc_b4dc7630>;
   605: 
   606:   // Use timing class of S2_storerb_pcr.
   607:   def NAME#_pcr : STInst<(outs IntRegs:$Rx32),
   608:        (ins IntRegs:$Rx32in, ModRegs:$Mu2, RC:$Rt32, IntRegs:$Cs),
   609:        ".error \"should not emit\" ", [], "$Rx32 = $Rx32in", tc_a2b365d2>;
   610: }
   611: }
   612: 
   613: defm PS_storerb : NewCircularStore<IntRegs, ByteAccess>;
   614: defm PS_storerh : NewCircularStore<IntRegs, HalfWordAccess>;
   615: defm PS_storerf : NewCircularStore<IntRegs, HalfWordAccess>;
   616: defm PS_storeri : NewCircularStore<IntRegs, WordAccess>;
   617: defm PS_storerd : NewCircularStore<DoubleRegs, WordAccess>;
   618: 
   619: // A pseudo that generates a runtime crash. This is used to implement
   620: // __builtin_trap.
   621: let hasSideEffects = 1, isPseudo = 1, isCodeGenOnly = 1, isSolo = 1 in
   622: def PS_crash: InstHexagon<(outs), (ins), "", [], "", PSEUDO, TypePSEUDO>;
   623: 
   624: // This is actual trap1 instruction from before v65. It's here since it is
   625: // no longer included in DepInstrInfo.td.
   626: def PS_trap1 : HInst<(outs), (ins u8_0Imm:$Ii), "trap1(#$Ii)", tc_53c851ab,
   627:                      TypeJ>, Enc_a51a9a, Requires<[HasPreV65]> {
   628:   let Inst{1-0} = 0b00;
   629:   let Inst{7-5} = 0b000;
   630:   let Inst{13-13} = 0b0;
   631:   let Inst{31-16} = 0b0101010010000000;
   632: }
   633: 
```
- EN: It defines declarative TableGen records like NAME, PS_storerb, PS_storerh, PS_storerf, PS_storeri, ... (8 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 NAME, PS_storerb, PS_storerh, PS_storerf, PS_storeri, ... (8 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonPseudo, HexagonMapping`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
