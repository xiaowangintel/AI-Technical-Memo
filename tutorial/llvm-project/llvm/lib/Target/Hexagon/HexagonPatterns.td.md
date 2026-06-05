# HexagonPatterns.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonPatterns.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file). When making changes, adding AddedComplexity to existing
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```tablegen
     1: //===- HexagonPatterns.td - Selection Patterns for Hexagon -*- tablegen -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: // Table of contents:
    10: //     (0) Definitions
    11: //     (1) Immediates
    12: //     (2) Type casts
    13: //     (3) Extend/truncate/saturate
    14: //     (4) Logical
    15: //     (5) Compare
    16: //     (6) Select
    17: //     (7) Insert/extract
    18: //     (8) Shift/permute
    19: //     (9) Arithmetic/bitwise
    20: //    (10) Bit
    21: //    (11) PIC
    22: //    (12) Load
    23: //    (13) Store
    24: //    (14) Memop
    25: //    (15) Call
    26: //    (16) Branch
    27: //    (17) Misc
    28: 
    29: // Guidelines (in no particular order):
    30: // 1. Avoid relying on pattern ordering to give preference to one pattern
    31: //    over another, prefer using AddedComplexity instead. The reason for
    32: //    this is to avoid unintended conseqeuences (caused by altering the
    33: //    order) when making changes. The current order of patterns in this
    34: //    file obviously does play some role, but none of the ordering was
    35: //    deliberately chosen (other than to create a logical structure of
    36: //    this file). When making changes, adding AddedComplexity to existing
    37: //    patterns may be needed.
    38: // 2. Maintain the logical structure of the file, try to put new patterns
    39: //    in designated sections.
    40: // 3. Do not use A2_combinew instruction directly, use Combinew fragment
    41: //    instead. It uses REG_SEQUENCE, which is more amenable to optimizations.
    42: // 4. Most selection macros are based on PatFrags. For DAGs that involve
    43: //    SDNodes, use pf1/pf2 to convert them to PatFrags. Use common frags
    44: //    whenever possible (see the Definitions section). When adding new
    45: //    macro, try to make is general to enable reuse across sections.
    46: // 5. Compound instructions (e.g. Rx+Rs*Rt) are generated under the condition
    47: //    that the nested operation has only one use. Having it separated in case
    48: //    of multiple uses avoids duplication of (processor) work.
    49: // 6. The v4 vector instructions (64-bit) are treated as core instructions,
    50: //    for example, A2_vaddh is in the "arithmetic" section with A2_add.
    51: // 7. When adding a pattern for an instruction with a constant-extendable
    52: //    operand, allow all possible kinds of inputs for the immediate value
    53: //    (see AnyImm/anyimm and their variants in the Definitions section).
    54: 
    55: 
    56: // --(0) Definitions -----------------------------------------------------
    57: //
    58: 
    59: // This complex pattern exists only to create a machine instruction operand
    60: // of type "frame index". There doesn't seem to be a way to do that directly
    61: // in the patterns.
    62: def AddrFI: ComplexPattern<i32, 1, "SelectAddrFI", [frameindex], []>;
    63: 
    64: // These complex patterns are not strictly necessary, since global address
    65: // folding will happen during DAG combining. For distinguishing between GA
    66: // and GP, pat frags with HexagonCONST32 and HexagonCONST32_GP can be used.
    67: def AddrGA: ComplexPattern<i32, 1, "SelectAddrGA", [], []>;
    68: def AddrGP: ComplexPattern<i32, 1, "SelectAddrGP", [], []>;
    69: def AnyImm: ComplexPattern<i32, 1, "SelectAnyImm", [], []>;
    70: def AnyInt: ComplexPattern<i32, 1, "SelectAnyInt", [], []>;
    71: 
    72: // Global address or a constant being a multiple of 2^n.
    73: def AnyImm0: ComplexPattern<i32, 1, "SelectAnyImm0", [], []>;
    74: def AnyImm1: ComplexPattern<i32, 1, "SelectAnyImm1", [], []>;
    75: def AnyImm2: ComplexPattern<i32, 1, "SelectAnyImm2", [], []>;
    76: def AnyImm3: ComplexPattern<i32, 1, "SelectAnyImm3", [], []>;
    77: 
    78: 
    79: // Type helper frags.
    80: def V2I1:   PatLeaf<(v2i1    PredRegs:$R)>;
    81: def V4I1:   PatLeaf<(v4i1    PredRegs:$R)>;
    82: def V8I1:   PatLeaf<(v8i1    PredRegs:$R)>;
    83: def V4I8:   PatLeaf<(v4i8    IntRegs:$R)>;
    84: def V2I16:  PatLeaf<(v2i16   IntRegs:$R)>;
    85: 
    86: def V8I8:   PatLeaf<(v8i8    DoubleRegs:$R)>;
    87: def V4I16:  PatLeaf<(v4i16   DoubleRegs:$R)>;
    88: def V2I32:  PatLeaf<(v2i32   DoubleRegs:$R)>;
    89: 
    90: def SDTVecLeaf:
    91:   SDTypeProfile<1, 0, [SDTCisVec<0>]>;
    92: def SDTVecVecIntOp:
    93:   SDTypeProfile<1, 3, [SDTCisVec<0>, SDTCisVec<1>, SDTCisSameAs<1,2>,
    94:                        SDTCisVT<3,i32>]>;
    95: def SDTIntBinOpWithCarryInOut:
    96:   SDTypeProfile<2, 3, [SDTCisInt<0>, SDTCisVT<1, i1>, SDTCisSameAs<2, 0>,
    97:                        SDTCisSameAs<3, 0>, SDTCisVT<4, i1>]>;
    98: 
    99: // Add with carry: (X, Y, Cin) -> (X+Y, Cout).
   100: def HexagonADDC: SDNode<"HexagonISD::ADDC", SDTIntBinOpWithCarryInOut>;
   101: 
   102: // Sub with carry: (X, Y, Cin) -> (X+~Y+Cin, Cout).
   103: def HexagonSUBC: SDNode<"HexagonISD::SUBC", SDTIntBinOpWithCarryInOut>;
   104: 
   105: // Same as ISD::SMUL_LOHI, but opaque to the combiner.
   106: // We want to legalize MULH[SU] to [SU]MUL_LOHI, but the
   107: // combiner will keep rewriting it back to MULH[SU].
   108: def HexagonSMUL_LOHI: SDNode<"HexagonISD::SMUL_LOHI", SDTIntBinHiLoOp>;
   109: 
   110: // Same as ISD::UMUL_LOHI, but opaque to the combiner.
   111: def HexagonUMUL_LOHI: SDNode<"HexagonISD::UMUL_LOHI", SDTIntBinHiLoOp>;
   112: 
   113: // Like SMUL_LOHI, but unsigned*signed.
   114: def HexagonUSMUL_LOHI: SDNode<"HexagonISD::USMUL_LOHI", SDTIntBinHiLoOp>;
   115: 
   116: def HexagonPTRUE:      SDNode<"HexagonISD::PTRUE",      SDTVecLeaf>;
   117: def HexagonPFALSE:     SDNode<"HexagonISD::PFALSE",     SDTVecLeaf>;
   118: 
   119: // Align two vectors (in Op0, Op1) to one that would have
   120: // been loaded from address in Op2.
   121: def HexagonVALIGN:     SDNode<"HexagonISD::VALIGN",     SDTVecVecIntOp>;
   122: 
   123: // Align vector address: Op0 & -Op1, except when it is
   124: // an address in a vector load, then it's a no-op.
   125: def HexagonVALIGNADDR: SDNode<"HexagonISD::VALIGNADDR", SDTIntUnaryOp>;
   126: 
   127: def HexagonMULHUS:     SDNode<"HexagonISD::MULHUS",     SDTIntBinOp>;
   128: 
   129: def SDTSaturate:
   130:   SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisInt<1>, SDTCisVT<2, OtherVT>]>;
   131: 
   132: // Signed saturate.
   133: def HexagonSSAT: SDNode<"HexagonISD::SSAT", SDTSaturate>;
   134: 
   135: // Unsigned saturate.
   136: def HexagonUSAT: SDNode<"HexagonISD::USAT", SDTSaturate>;
   137: 
   138: def ptrue:  PatFrag<(ops), (HexagonPTRUE)>;
   139: def pfalse: PatFrag<(ops), (HexagonPFALSE)>;
   140: def pnot:   PatFrag<(ops node:$Pu), (xor node:$Pu, ptrue)>;
   141: 
   142: def: Pat<(v8i1 (HexagonPFALSE)), (C2_tfrrp (A2_tfrsi (i32 0)))>;
   143: def: Pat<(v4i1 (HexagonPFALSE)), (C2_tfrrp (A2_tfrsi (i32 0)))>;
   144: def: Pat<(v2i1 (HexagonPFALSE)), (C2_tfrrp (A2_tfrsi (i32 0)))>;
   145: 
   146: def: Pat<(v8i1 (HexagonPTRUE)), (C2_tfrrp (A2_tfrsi (i32 -1)))>;
   147: def: Pat<(v4i1 (HexagonPTRUE)), (C2_tfrrp (A2_tfrsi (i32 -1)))>;
   148: def: Pat<(v2i1 (HexagonPTRUE)), (C2_tfrrp (A2_tfrsi (i32 -1)))>;
   149: 
   150: def valign: PatFrag<(ops node:$Vt, node:$Vs, node:$Ru),
   151:                     (HexagonVALIGN node:$Vt, node:$Vs, node:$Ru)>;
   152: def valignaddr: PatFrag<(ops node:$Addr), (HexagonVALIGNADDR node:$Addr)>;
   153: 
   154: def ssat: PatFrag<(ops node:$V, node:$Ty), (HexagonSSAT node:$V, node:$Ty)>;
   155: def usat: PatFrag<(ops node:$V, node:$Ty), (HexagonUSAT node:$V, node:$Ty)>;
   156: 
   157: // Pattern fragments to extract the low and high subregisters from a
   158: // 64-bit value.
   159: def LoReg: OutPatFrag<(ops node:$Rs), (EXTRACT_SUBREG $Rs, isub_lo)>;
   160: def HiReg: OutPatFrag<(ops node:$Rs), (EXTRACT_SUBREG $Rs, isub_hi)>;
   161: 
   162: def IsOrAdd: PatFrag<(ops node:$A, node:$B), (or node:$A, node:$B), [{
   163:   return isOrEquivalentToAdd(N);
   164: }]>;
   165: 
   166: def IsPow2_32: PatLeaf<(i32 imm), [{
   167:   uint32_t V = N->getZExtValue();
   168:   return isPowerOf2_32(V);
   169: }]>;
   170: 
   171: def IsPow2_64: PatLeaf<(i64 imm), [{
   172:   uint64_t V = N->getZExtValue();
   173:   return isPowerOf2_64(V);
   174: }]>;
   175: 
   176: def IsNPow2_32: PatLeaf<(i32 imm), [{
   177:   uint32_t NV = ~N->getZExtValue();
   178:   return isPowerOf2_32(NV);
   179: }]>;
   180: 
   181: def IsPow2_64L: PatLeaf<(i64 imm), [{
   182:   uint64_t V = N->getZExtValue();
   183:   return isPowerOf2_64(V) && Log2_64(V) < 32;
   184: }]>;
   185: 
   186: def IsPow2_64H: PatLeaf<(i64 imm), [{
   187:   uint64_t V = N->getZExtValue();
   188:   return isPowerOf2_64(V) && Log2_64(V) >= 32;
   189: }]>;
   190: 
   191: def IsNPow2_64L: PatLeaf<(i64 imm), [{
   192:   uint64_t NV = ~N->getZExtValue();
   193:   return isPowerOf2_64(NV) && Log2_64(NV) < 32;
   194: }]>;
   195: 
   196: def IsNPow2_64H: PatLeaf<(i64 imm), [{
   197:   uint64_t NV = ~N->getZExtValue();
   198:   return isPowerOf2_64(NV) && Log2_64(NV) >= 32;
   199: }]>;
   200: 
```
- EN: It defines declarative TableGen records like AddrFI, AddrGA, AddrGP, AnyImm, AnyInt, ... (50 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as isOrEquivalentToAdd, getZExtValue, isPowerOf2_32, isPowerOf2_64, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPatterns, HexagonCONST32, HexagonCONST32_GP, HexagonADDC, ... (16 total), showing how the code connects to sibling backend components.
- CN: 这里定义了 AddrFI, AddrGA, AddrGP, AnyImm, AnyInt, ... (50 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 isOrEquivalentToAdd, getZExtValue, isPowerOf2_32, isPowerOf2_64 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPatterns, HexagonCONST32, HexagonCONST32_GP, HexagonADDC, ... (16 total)，说明了它与同级后端组件的连接关系。

### Lines 201-400 / 第 201-400 行

```tablegen
   201: class IsULE<int Width, int Arg>: PatLeaf<(i32 imm),
   202:   "uint64_t V = N->getZExtValue();" #
   203:   "return isUInt<" # Width # ">(V) && V <= " # Arg # ";"
   204: >;
   205: 
   206: class IsUGT<int Width, int Arg>: PatLeaf<(i32 imm),
   207:   "uint64_t V = N->getZExtValue();" #
   208:   "return isUInt<" # Width # ">(V) && V > " # Arg # ";"
   209: >;
   210: 
   211: def SDEC1: SDNodeXForm<imm, [{
   212:   int32_t V = N->getSExtValue();
   213:   return CurDAG->getSignedTargetConstant(V-1, SDLoc(N), MVT::i32);
   214: }]>;
   215: 
   216: def UDEC1: SDNodeXForm<imm, [{
   217:   uint32_t V = N->getZExtValue();
   218:   assert(V >= 1);
   219:   return CurDAG->getTargetConstant(V-1, SDLoc(N), MVT::i32);
   220: }]>;
   221: 
   222: def UDEC32: SDNodeXForm<imm, [{
   223:   uint32_t V = N->getZExtValue();
   224:   assert(V >= 32);
   225:   return CurDAG->getTargetConstant(V-32, SDLoc(N), MVT::i32);
   226: }]>;
   227: 
   228: class Subi<int From>: SDNodeXForm<imm,
   229:   "int32_t V = " # From # " - N->getSExtValue();" #
   230:   "return CurDAG->getTargetConstant(V, SDLoc(N), MVT::i32);"
   231: >;
   232: 
   233: def Log2_32: SDNodeXForm<imm, [{
   234:   uint32_t V = N->getZExtValue();
   235:   return CurDAG->getTargetConstant(Log2_32(V), SDLoc(N), MVT::i32);
   236: }]>;
   237: 
   238: def Log2_64: SDNodeXForm<imm, [{
   239:   uint64_t V = N->getZExtValue();
   240:   return CurDAG->getTargetConstant(Log2_64(V), SDLoc(N), MVT::i32);
   241: }]>;
   242: 
   243: def LogN2_32: SDNodeXForm<imm, [{
   244:   uint32_t NV = ~N->getZExtValue();
   245:   return CurDAG->getTargetConstant(Log2_32(NV), SDLoc(N), MVT::i32);
   246: }]>;
   247: 
   248: def LogN2_64: SDNodeXForm<imm, [{
   249:   uint64_t NV = ~N->getZExtValue();
   250:   return CurDAG->getTargetConstant(Log2_64(NV), SDLoc(N), MVT::i32);
   251: }]>;
   252: 
   253: def NegImm8: SDNodeXForm<imm, [{
   254:   int8_t NV = -N->getSExtValue();
   255:   return CurDAG->getTargetConstant(NV, SDLoc(N), MVT::i32);
   256: }]>;
   257: 
   258: def NegImm16: SDNodeXForm<imm, [{
   259:   int16_t NV = -N->getSExtValue();
   260:   return CurDAG->getTargetConstant(NV, SDLoc(N), MVT::i32);
   261: }]>;
   262: 
   263: def NegImm32: SDNodeXForm<imm, [{
   264:   int32_t NV = -N->getSExtValue();
   265:   return CurDAG->getTargetConstant(NV, SDLoc(N), MVT::i32);
   266: }]>;
   267: 
   268: def SplatB: SDNodeXForm<imm, [{
   269:   uint32_t V = N->getZExtValue();
   270:   assert(isUInt<8>(V) || V >> 8 == 0xFFFFFF);
   271:   V &= 0xFF;
   272:   uint32_t S = V << 24 | V << 16 | V << 8 | V;
   273:   return CurDAG->getTargetConstant(S, SDLoc(N), MVT::i32);
   274: }]>;
   275: 
   276: def SplatH: SDNodeXForm<imm, [{
   277:   uint32_t V = N->getZExtValue();
   278:   assert(isUInt<16>(V) || V >> 16 == 0xFFFF);
   279:   V &= 0xFFFF;
   280:   return CurDAG->getTargetConstant(V << 16 | V, SDLoc(N), MVT::i32);
   281: }]>;
   282: 
   283: 
   284: // Helpers for type promotions/contractions.
   285: def I1toI32:  OutPatFrag<(ops node:$Rs), (C2_muxii (i1 $Rs), 1, 0)>;
   286: def I32toI1:  OutPatFrag<(ops node:$Rs), (i1 (C2_cmpgtui (i32 $Rs), (i32 0)))>;
   287: def ToZext64: OutPatFrag<(ops node:$Rs), (i64 (A4_combineir 0, (i32 $Rs)))>;
   288: def ToSext64: OutPatFrag<(ops node:$Rs), (i64 (A2_sxtw (i32 $Rs)))>;
   289: def ToAext64: OutPatFrag<(ops node:$Rs),
   290:   (REG_SEQUENCE DoubleRegs, (i32 (IMPLICIT_DEF)), isub_hi, (i32 $Rs), isub_lo)>;
   291: 
   292: def Combinew: OutPatFrag<(ops node:$Rs, node:$Rt),
   293:   (REG_SEQUENCE DoubleRegs, $Rs, isub_hi, $Rt, isub_lo)>;
   294: 
   295: def addrga: PatLeaf<(i32 AddrGA:$Addr)>;
   296: def addrgp: PatLeaf<(i32 AddrGP:$Addr)>;
   297: def anyimm: PatLeaf<(i32 AnyImm:$Imm)>;
   298: def anyint: PatLeaf<(i32 AnyInt:$Imm)>;
   299: 
   300: // Global address or an aligned constant.
   301: def anyimm0: PatLeaf<(i32 AnyImm0:$Addr)>;
   302: def anyimm1: PatLeaf<(i32 AnyImm1:$Addr)>;
   303: def anyimm2: PatLeaf<(i32 AnyImm2:$Addr)>;
   304: def anyimm3: PatLeaf<(i32 AnyImm3:$Addr)>;
   305: 
   306: def f32ImmPred : PatLeaf<(f32 fpimm:$F)>;
   307: def f64ImmPred : PatLeaf<(f64 fpimm:$F)>;
   308: def f32zero: PatLeaf<(f32 fpimm:$F), [{
   309:   return N->isExactlyValue(APFloat::getZero(APFloat::IEEEsingle(), false));
   310: }]>;
   311: 
   312: // This complex pattern is really only to detect various forms of
   313: // sign-extension i32->i64. The selected value will be of type i64
   314: // whose low word is the value being extended. The high word is
   315: // unspecified.
   316: def Usxtw:  ComplexPattern<i64, 1, "DetectUseSxtw", [], []>;
   317: 
   318: def Aext64: PatFrag<(ops node:$Rs), (i64 (anyext node:$Rs))>;
   319: def Zext64: PatFrag<(ops node:$Rs), (i64 (zext node:$Rs))>;
   320: def Sext64: PatLeaf<(i64 Usxtw:$Rs)>;
   321: 
   322: def azext: PatFrags<(ops node:$Rs), [(zext node:$Rs), (anyext node:$Rs)]>;
   323: def asext: PatFrags<(ops node:$Rs), [(sext node:$Rs), (anyext node:$Rs)]>;
   324: 
   325: def: Pat<(IsOrAdd (i32 AddrFI:$Rs), s32_0ImmPred:$off),
   326:          (PS_fi (i32 AddrFI:$Rs), imm:$off)>;
   327: 
   328: 
   329: // Converters from unary/binary SDNode to PatFrag.
   330: class pf1<SDNode Op> : PatFrag<(ops node:$a), (Op node:$a)>;
   331: class pf2<SDNode Op> : PatFrag<(ops node:$a, node:$b), (Op node:$a, node:$b)>;
   332: 
   333: class Not2<PatFrag P>
   334:   : PatFrag<(ops node:$A, node:$B), (P node:$A, (not node:$B))>;
   335: class VNot2<PatFrag P, PatFrag Not>
   336:   : PatFrag<(ops node:$A, node:$B), (P node:$A, (Not node:$B))>;
   337: 
   338: // If there is a constant operand that feeds the and/or instruction,
   339: // do not generate the compound instructions.
   340: // It is not always profitable, as some times we end up with a transfer.
   341: // Check the below example.
   342: // ra = #65820; rb = lsr(rb, #8); rc ^= and (rb, ra)
   343: // Instead this is preferable.
   344: // ra = and (#65820, lsr(ra, #8)); rb = xor(rb, ra)
   345: class Su_ni1<PatFrag Op>
   346:   : PatFrag<Op.Operands, !head(Op.Fragments), [{
   347:             if (hasOneUse(N)){
   348:               // Check if Op1 is an immediate operand.
   349:               SDValue Op1 = N->getOperand(1);
   350:               return !isa<ConstantSDNode>(Op1);
   351:             }
   352:             return false;}],
   353:             Op.OperandTransform>;
   354: 
   355: class Su<PatFrag Op>
   356:   : PatFrag<Op.Operands, !head(Op.Fragments), [{ return hasOneUse(N); }],
   357:             Op.OperandTransform>;
   358: 
   359: // Main selection macros.
   360: 
   361: class OpR_R_pat<InstHexagon MI, PatFrag Op, ValueType ResVT, PatFrag RegPred>
   362:   : Pat<(ResVT (Op RegPred:$Rs)), (MI RegPred:$Rs)>;
   363: 
   364: class OpR_RI_pat<InstHexagon MI, PatFrag Op, ValueType ResType,
   365:                  PatFrag RegPred, PatFrag ImmPred>
   366:   : Pat<(ResType (Op RegPred:$Rs, ImmPred:$I)),
   367:         (MI RegPred:$Rs, imm:$I)>;
   368: 
   369: class OpR_RR_pat<InstHexagon MI, PatFrag Op, ValueType ResType,
   370:                  PatFrag RsPred, PatFrag RtPred = RsPred>
   371:   : Pat<(ResType (Op RsPred:$Rs, RtPred:$Rt)),
   372:         (MI RsPred:$Rs, RtPred:$Rt)>;
   373: 
   374: class AccRRI_pat<InstHexagon MI, PatFrag AccOp, PatFrag Op,
   375:                  PatFrag RegPred, PatFrag ImmPred>
   376:   : Pat<(AccOp RegPred:$Rx, (Op RegPred:$Rs, ImmPred:$I)),
   377:         (MI RegPred:$Rx, RegPred:$Rs, imm:$I)>;
   378: 
   379: class AccRRR_pat<InstHexagon MI, PatFrag AccOp, PatFrag Op,
   380:                  PatFrag RxPred, PatFrag RsPred, PatFrag RtPred>
   381:   : Pat<(AccOp RxPred:$Rx, (Op RsPred:$Rs, RtPred:$Rt)),
   382:         (MI RxPred:$Rx, RsPred:$Rs, RtPred:$Rt)>;
   383: 
   384: multiclass SelMinMax_pats<PatFrag CmpOp, PatFrag Val,
   385:                           InstHexagon InstA, InstHexagon InstB> {
   386:   def: Pat<(select (i1 (CmpOp Val:$A, Val:$B)), Val:$A, Val:$B),
   387:            (InstA Val:$A, Val:$B)>;
   388:   def: Pat<(select (i1 (CmpOp Val:$A, Val:$B)), Val:$B, Val:$A),
   389:            (InstB Val:$A, Val:$B)>;
   390: }
   391: 
   392: multiclass MinMax_pats<InstHexagon PickT, InstHexagon PickS,
   393:                        SDPatternOperator Sel, SDPatternOperator CmpOp,
   394:                        ValueType CmpType, PatFrag CmpPred> {
   395:   def: Pat<(Sel (CmpType (CmpOp CmpPred:$Vs, CmpPred:$Vt)),
   396:                 CmpPred:$Vt, CmpPred:$Vs),
   397:            (PickT CmpPred:$Vs, CmpPred:$Vt)>;
   398:   def: Pat<(Sel (CmpType (CmpOp CmpPred:$Vs, CmpPred:$Vt)),
   399:                 CmpPred:$Vs, CmpPred:$Vt),
   400:            (PickS CmpPred:$Vs, CmpPred:$Vt)>;
```
- EN: It declares types such as IsULE, IsUGT, Subi, pf1, ... (14 total), which carry the state or API of this component. It defines declarative TableGen records like IsULE, IsUGT, SDEC1, UDEC1, UDEC32, ... (51 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as PatLeaf<, getSExtValue, getSignedTargetConstant, getZExtValue, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 IsULE, IsUGT, Subi, pf1, ... (14 total) 等类型，用来承载该组件的状态或接口。 这里定义了 IsULE, IsUGT, SDEC1, UDEC1, UDEC32, ... (51 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 PatLeaf<, getSExtValue, getSignedTargetConstant, getZExtValue, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 401-600 / 第 401-600 行

```tablegen
   401: }
   402: 
   403: // Bitcasts between same-size vector types are no-ops, except for the
   404: // actual type change.
   405: multiclass NopCast_pat<ValueType Ty1, ValueType Ty2, RegisterClass RC> {
   406:   def: Pat<(Ty1 (bitconvert (Ty2 RC:$Val))), (Ty1 RC:$Val)>;
   407:   def: Pat<(Ty2 (bitconvert (Ty1 RC:$Val))), (Ty2 RC:$Val)>;
   408: }
   409: 
   410: // Frags for commonly used SDNodes.
   411: def Add: pf2<add>;    def And: pf2<and>;    def Sra: pf2<sra>;
   412: def Sub: pf2<sub>;    def Or:  pf2<or>;     def Srl: pf2<srl>;
   413: def Mul: pf2<mul>;    def Xor: pf2<xor>;    def Shl: pf2<shl>;
   414: def Sext: pf1<sext>;  def Zext: pf1<zext>;
   415: 
   416: def Smin: pf2<smin>;  def Smax: pf2<smax>;
   417: def Umin: pf2<umin>;  def Umax: pf2<umax>;
   418: 
   419: def Rol: pf2<rotl>;
   420: 
   421: def Fptosi: pf1<fp_to_sint>;
   422: def Fptoui: pf1<fp_to_uint>;
   423: def Sitofp: pf1<sint_to_fp>;
   424: def Uitofp: pf1<uint_to_fp>;
   425: 
   426: // --(1) Immediate -------------------------------------------------------
   427: //
   428: 
   429: def Imm64Lo: SDNodeXForm<imm, [{
   430:   return CurDAG->getSignedTargetConstant(int32_t(N->getSExtValue()),
   431:                                          SDLoc(N), MVT::i32);
   432: }]>;
   433: def Imm64Hi: SDNodeXForm<imm, [{
   434:   return CurDAG->getSignedTargetConstant(int32_t(N->getSExtValue()>>32),
   435:                                          SDLoc(N), MVT::i32);
   436: }]>;
   437: 
   438: 
   439: def SDTHexagonCONST32
   440:   : SDTypeProfile<1, 1, [SDTCisVT<0, i32>, SDTCisVT<1, i32>, SDTCisPtrTy<0>]>;
   441: 
   442: // Jump table.
   443: def HexagonJT:          SDNode<"HexagonISD::JT",          SDTIntUnaryOp>;
   444: 
   445: // Constant pool.
   446: def HexagonCP:          SDNode<"HexagonISD::CP",          SDTIntUnaryOp>;
   447: 
   448: def HexagonCONST32:     SDNode<"HexagonISD::CONST32",     SDTHexagonCONST32>;
   449: 
   450: // For marking data present in GP.
   451: def HexagonCONST32_GP:  SDNode<"HexagonISD::CONST32_GP",  SDTHexagonCONST32>;
   452: 
   453: def TruncI64ToI32: SDNodeXForm<imm, [{
   454:   return CurDAG->getSignedTargetConstant(N->getSExtValue(), SDLoc(N), MVT::i32);
   455: }]>;
   456: 
   457: def: Pat<(s32_0ImmPred:$s16), (A2_tfrsi imm:$s16)>;
   458: def: Pat<(s8_0Imm64Pred:$s8), (A2_tfrpi (TruncI64ToI32 $s8))>;
   459: 
   460: def: Pat<(HexagonCONST32    tglobaltlsaddr:$A), (A2_tfrsi imm:$A)>;
   461: def: Pat<(HexagonCONST32    bbl:$A),            (A2_tfrsi imm:$A)>;
   462: def: Pat<(HexagonCONST32    tglobaladdr:$A),    (A2_tfrsi imm:$A)>;
   463: def: Pat<(HexagonCONST32_GP tblockaddress:$A),  (A2_tfrsi imm:$A)>;
   464: def: Pat<(HexagonCONST32_GP tglobaladdr:$A),    (A2_tfrsi imm:$A)>;
   465: def: Pat<(HexagonJT         tjumptable:$A),     (A2_tfrsi imm:$A)>;
   466: def: Pat<(HexagonCP         tconstpool:$A),     (A2_tfrsi imm:$A)>;
   467: // The HVX load patterns also match CP directly. Make sure that if
   468: // the selection of this opcode changes, it's updated in all places.
   469: 
   470: def: Pat<(i1 0),        (PS_false)>;
   471: def: Pat<(i1 1),        (PS_true)>;
   472: def: Pat<(i64 imm:$v),  (CONST64 imm:$v)>,
   473:      Requires<[UseSmallData,NotOptTinyCore]>;
   474: def: Pat<(i64 imm:$v),
   475:          (Combinew (A2_tfrsi (Imm64Hi $v)), (A2_tfrsi (Imm64Lo $v)))>;
   476: 
   477: def ftoi : SDNodeXForm<fpimm, [{
   478:   APInt I = N->getValueAPF().bitcastToAPInt();
   479:   return CurDAG->getTargetConstant(I.getZExtValue(), SDLoc(N),
   480:                                    MVT::getIntegerVT(I.getBitWidth()));
   481: }]>;
   482: 
   483: def: Pat<(f32ImmPred:$f), (A2_tfrsi (ftoi $f))>;
   484: def: Pat<(f64ImmPred:$f), (CONST64  (ftoi $f))>;
   485: 
   486: def ToI32: OutPatFrag<(ops node:$V), (A2_tfrsi $V)>;
   487: 
   488: // --(2) Type cast -------------------------------------------------------
   489: //
   490: 
   491: def: OpR_R_pat<F2_conv_sf2df,      pf1<fpextend>,   f64, F32>;
   492: def: OpR_R_pat<F2_conv_df2sf,      pf1<fpround>,    f32, F64>;
   493: 
   494: def: OpR_R_pat<F2_conv_w2sf,       pf1<sint_to_fp>, f32, I32>;
   495: def: OpR_R_pat<F2_conv_d2sf,       pf1<sint_to_fp>, f32, I64>;
   496: def: OpR_R_pat<F2_conv_w2df,       pf1<sint_to_fp>, f64, I32>;
   497: def: OpR_R_pat<F2_conv_d2df,       pf1<sint_to_fp>, f64, I64>;
   498: 
   499: def: OpR_R_pat<F2_conv_uw2sf,      pf1<uint_to_fp>, f32, I32>;
   500: def: OpR_R_pat<F2_conv_ud2sf,      pf1<uint_to_fp>, f32, I64>;
   501: def: OpR_R_pat<F2_conv_uw2df,      pf1<uint_to_fp>, f64, I32>;
   502: def: OpR_R_pat<F2_conv_ud2df,      pf1<uint_to_fp>, f64, I64>;
   503: 
   504: def: OpR_R_pat<F2_conv_sf2w_chop,  pf1<fp_to_sint>, i32, F32>;
   505: def: OpR_R_pat<F2_conv_df2w_chop,  pf1<fp_to_sint>, i32, F64>;
   506: def: OpR_R_pat<F2_conv_sf2d_chop,  pf1<fp_to_sint>, i64, F32>;
   507: def: OpR_R_pat<F2_conv_df2d_chop,  pf1<fp_to_sint>, i64, F64>;
   508: 
   509: def: OpR_R_pat<F2_conv_sf2uw_chop, pf1<fp_to_uint>, i32, F32>;
   510: def: OpR_R_pat<F2_conv_df2uw_chop, pf1<fp_to_uint>, i32, F64>;
   511: def: OpR_R_pat<F2_conv_sf2ud_chop, pf1<fp_to_uint>, i64, F32>;
   512: def: OpR_R_pat<F2_conv_df2ud_chop, pf1<fp_to_uint>, i64, F64>;
   513: 
   514: def: Pat<(i32 (fp_to_bf16 F32:$v)),
   515:          (C2_mux (F2_sfclass F32:$v, 0x10), (A2_tfrsi(i32 0x7fff)),
   516:            (C2_mux
   517:              (C2_cmpeq
   518:                (A2_and F32:$v, (A2_tfrsi (i32 0x1FFFF))),
   519:                (A2_tfrsi (i32 0x08000))),
   520:              (A2_and (A2_asrh F32:$v), (A2_tfrsi (i32 65535))),
   521:              (A2_and
   522:                (A2_asrh
   523:                  (A2_add F32:$v, (A2_and F32:$v, (A2_tfrsi (i32 0x8000))))),
   524:                  (A2_tfrsi (i32 65535))))
   525:          )>;
   526: // Bitcast is different than [fp|sint|uint]_to_[sint|uint|fp].
   527: def: Pat<(i32 (bitconvert F32:$v)), (I32:$v)>;
   528: def: Pat<(f32 (bitconvert I32:$v)), (F32:$v)>;
   529: def: Pat<(i64 (bitconvert F64:$v)), (I64:$v)>;
   530: def: Pat<(f64 (bitconvert I64:$v)), (F64:$v)>;
   531: 
   532: // Bit convert 32- and 64-bit types.
   533: // All of these are bitcastable to one another: i32, v2i16, v4i8.
   534: defm: NopCast_pat<i32,   v2i16, IntRegs>;
   535: defm: NopCast_pat<i32,    v4i8, IntRegs>;
   536: defm: NopCast_pat<v2i16,  v4i8, IntRegs>;
   537: // All of these are bitcastable to one another: i64, v2i32, v4i16, v8i8.
   538: defm: NopCast_pat<i64,   v2i32, DoubleRegs>;
   539: defm: NopCast_pat<i64,   v4i16, DoubleRegs>;
   540: defm: NopCast_pat<i64,    v8i8, DoubleRegs>;
   541: defm: NopCast_pat<v2i32, v4i16, DoubleRegs>;
   542: defm: NopCast_pat<v2i32,  v8i8, DoubleRegs>;
   543: defm: NopCast_pat<v4i16,  v8i8, DoubleRegs>;
   544: 
   545: 
   546: // --(3) Extend/truncate/saturate ----------------------------------------
   547: //
   548: 
   549: def: Pat<(sext_inreg I32:$Rs, i8),  (A2_sxtb I32:$Rs)>;
   550: def: Pat<(sext_inreg I32:$Rs, i16), (A2_sxth I32:$Rs)>;
   551: def: Pat<(sext_inreg I64:$Rs, i32), (A2_sxtw (LoReg $Rs))>;
   552: def: Pat<(sext_inreg I64:$Rs, i16), (A2_sxtw (A2_sxth (LoReg $Rs)))>;
   553: def: Pat<(sext_inreg I64:$Rs, i8),  (A2_sxtw (A2_sxtb (LoReg $Rs)))>;
   554: 
   555: def: Pat<(i64 (sext I32:$Rs)), (A2_sxtw I32:$Rs)>;
   556: def: Pat<(Zext64 I32:$Rs),     (ToZext64 $Rs)>;
   557: def: Pat<(Aext64 I32:$Rs),     (ToZext64 $Rs)>;
   558: 
   559: def: Pat<(i32 (trunc I64:$Rs)), (LoReg $Rs)>;
   560: def: Pat<(i1 (trunc I32:$Rs)),  (S2_tstbit_i I32:$Rs, 0)>;
   561: def: Pat<(i1 (trunc I64:$Rs)),  (S2_tstbit_i (LoReg $Rs), 0)>;
   562: 
   563: let AddedComplexity = 20 in {
   564:   def: Pat<(and I32:$Rs, 255),   (A2_zxtb I32:$Rs)>;
   565:   def: Pat<(and I32:$Rs, 65535), (A2_zxth I32:$Rs)>;
   566: }
   567: 
   568: // Extensions from i1 or vectors of i1.
   569: def: Pat<(i32 (azext I1:$Pu)), (C2_muxii I1:$Pu, 1, 0)>;
   570: def: Pat<(i64 (azext I1:$Pu)), (ToZext64 (C2_muxii I1:$Pu, 1, 0))>;
   571: def: Pat<(i32  (sext I1:$Pu)), (C2_muxii I1:$Pu, -1, 0)>;
   572: def: Pat<(i64  (sext I1:$Pu)), (Combinew (C2_muxii PredRegs:$Pu, -1, 0),
   573:                                          (C2_muxii PredRegs:$Pu, -1, 0))>;
   574: 
   575: def: Pat<(v2i16 (sext V2I1:$Pu)), (S2_vtrunehb (C2_mask V2I1:$Pu))>;
   576: def: Pat<(v2i32 (sext V2I1:$Pu)), (C2_mask V2I1:$Pu)>;
   577: def: Pat<(v4i8  (sext V4I1:$Pu)), (S2_vtrunehb (C2_mask V4I1:$Pu))>;
   578: def: Pat<(v4i16 (sext V4I1:$Pu)), (C2_mask V4I1:$Pu)>;
   579: def: Pat<(v8i8  (sext V8I1:$Pu)), (C2_mask V8I1:$Pu)>;
   580: 
   581: def Vsplatpi: OutPatFrag<(ops node:$V),
   582:                          (Combinew (A2_tfrsi $V), (A2_tfrsi $V))>;
   583: 
   584: def: Pat<(v2i16 (azext V2I1:$Pu)),
   585:          (A2_andir (S2_vtrunehb (C2_mask V2I1:$Pu)), (i32 0x00010001))>;
   586: def: Pat<(v2i32 (azext V2I1:$Pu)),
   587:          (A2_andp (C2_mask V2I1:$Pu), (A2_combineii (i32 1), (i32 1)))>;
   588: def: Pat<(v4i8 (azext V4I1:$Pu)),
   589:          (A2_andir (S2_vtrunehb (C2_mask V4I1:$Pu)), (i32 0x01010101))>;
   590: def: Pat<(v4i16 (azext V4I1:$Pu)),
   591:          (A2_andp (C2_mask V4I1:$Pu), (Vsplatpi (i32 0x00010001)))>;
   592: def: Pat<(v8i8 (azext V8I1:$Pu)),
   593:          (A2_andp (C2_mask V8I1:$Pu), (Vsplatpi (i32 0x01010101)))>;
   594: 
   595: def: Pat<(v4i16 (azext  V4I8:$Rs)),  (S2_vzxtbh V4I8:$Rs)>;
   596: def: Pat<(v2i32 (azext  V2I16:$Rs)), (S2_vzxthw V2I16:$Rs)>;
   597: def: Pat<(v4i16 (sext   V4I8:$Rs)),  (S2_vsxtbh V4I8:$Rs)>;
   598: def: Pat<(v2i32 (sext   V2I16:$Rs)), (S2_vsxthw V2I16:$Rs)>;
   599: 
   600: def: Pat<(v2i32 (sext_inreg V2I32:$Rs, v2i8)),
```
- EN: It defines declarative TableGen records like NopCast_pat, Add, Sub, Mul, Sext, ... (23 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getSignedTargetConstant, getValueAPF, getTargetConstant, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonJT, HexagonISD, HexagonCP, HexagonCONST32, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里定义了 NopCast_pat, Add, Sub, Mul, Sext, ... (23 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getSignedTargetConstant, getValueAPF, getTargetConstant 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonJT, HexagonISD, HexagonCP, HexagonCONST32, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 601-800 / 第 601-800 行

```tablegen
   601:          (Combinew (A2_sxtb (HiReg $Rs)), (A2_sxtb (LoReg $Rs)))>;
   602: 
   603: def: Pat<(v2i32 (sext_inreg V2I32:$Rs, v2i16)),
   604:          (Combinew (A2_sxth (HiReg $Rs)), (A2_sxth (LoReg $Rs)))>;
   605: 
   606: // Truncate: from vector B copy all 'E'ven 'B'yte elements:
   607: // A[0] = B[0];  A[1] = B[2];  A[2] = B[4];  A[3] = B[6];
   608: def: Pat<(v4i8 (trunc V4I16:$Rs)),
   609:          (S2_vtrunehb V4I16:$Rs)>;
   610: 
   611: // Truncate: from vector B copy all 'O'dd 'B'yte elements:
   612: // A[0] = B[1];  A[1] = B[3];  A[2] = B[5];  A[3] = B[7];
   613: // S2_vtrunohb
   614: 
   615: // Truncate: from vectors B and C copy all 'E'ven 'H'alf-word elements:
   616: // A[0] = B[0];  A[1] = B[2];  A[2] = C[0];  A[3] = C[2];
   617: // S2_vtruneh
   618: 
   619: def: Pat<(v2i16 (trunc V2I32:$Rs)),
   620:          (A2_combine_ll (HiReg $Rs), (LoReg $Rs))>;
   621: 
   622: // Truncate to vNi1
   623: def: Pat<(v2i1 (trunc V2I32:$Rs)),
   624:          (A4_vcmpweqi (A2_andp V2I32:$Rs, (A2_combineii (i32 1), (i32 1))),
   625:                       (i32 1))>;
   626: def: Pat<(v4i1 (trunc V4I16:$Rs)),
   627:          (A4_vcmpheqi (Combinew (A2_andir (HiReg $Rs), (i32 0x00010001)),
   628:                                 (A2_andir (LoReg $Rs), (i32 0x00010001))),
   629:                       (i32 1))>;
   630: def: Pat<(v8i1 (trunc V8I8:$Rs)),
   631:          (A4_vcmpbeqi (Combinew (A2_andir (HiReg $Rs), (i32 0x01010101)),
   632:                                 (A2_andir (LoReg $Rs), (i32 0x01010101))),
   633:                       (i32 1))>;
   634: def : Pat<(v4i1 (trunc V4I8:$Rs)),
   635:           (A4_vcmpheqi (Combinew (A2_andir (HiReg (S2_vzxtbh $Rs)), 0x00010001),
   636:                                  (A2_andir (LoReg (S2_vzxtbh $Rs)), 0x00010001)),
   637:                        (i32 1))>;
   638: def: Pat<(v2i1 (trunc V2I16:$Rs)),
   639:           (A4_vcmpweqi (A2_andp (S2_vzxthw $Rs), (A2_combineii (i32 1), (i32 1))),
   640:                       (i32 1))>;
   641: 
   642: 
   643: // Saturation:
   644: // Note: saturation assumes the same signed-ness for the input and the
   645: // output.
   646: def: Pat<(i32 (ssat I32:$Rs, i8)),  (A2_satb  I32:$Rs)>;
   647: def: Pat<(i32 (ssat I32:$Rs, i16)), (A2_sath  I32:$Rs)>;
   648: def: Pat<(i32 (ssat I64:$Rs, i32)), (A2_sat   I64:$Rs)>;
   649: def: Pat<(i32 (usat I32:$Rs, i8)),  (A2_satub I32:$Rs)>;
   650: def: Pat<(i32 (usat I32:$Rs, i16)), (A2_satuh I32:$Rs)>;
   651: def: Pat<(i32 (usat I64:$Rs, i32)),
   652:          (C2_mux (C2_cmpeqi (HiReg $Rs), (i32 0)), (LoReg $Rs), (i32 -1))>;
   653: 
   654: def: Pat<(v4i8  (ssat V4I16:$Rs, v4i8)),  (S2_vsathb  V4I16:$Rs)>;
   655: def: Pat<(v2i16 (ssat V2I32:$Rs, v2i16)), (S2_vsatwh  V2I32:$Rs)>;
   656: def: Pat<(v4i8  (usat V4I16:$Rs, v4i8)),  (S2_vsathub V4I16:$Rs)>;
   657: def: Pat<(v2i16 (usat V2I32:$Rs, v2i16)), (S2_vsatwuh V2I32:$Rs)>;
   658: 
   659: 
   660: // --(4) Logical ---------------------------------------------------------
   661: //
   662: 
   663: def: Pat<(not I1:$Ps),      (C2_not I1:$Ps)>;
   664: def: Pat<(pnot V2I1:$Ps),   (C2_not V2I1:$Ps)>;
   665: def: Pat<(pnot V4I1:$Ps),   (C2_not V4I1:$Ps)>;
   666: def: Pat<(pnot V8I1:$Ps),   (C2_not V8I1:$Ps)>;
   667: def: Pat<(add I1:$Ps, -1),  (C2_not I1:$Ps)>;
   668: 
   669: def: OpR_RR_pat<C2_and,         And, i1, I1>;
   670: def: OpR_RR_pat<C2_or,           Or, i1, I1>;
   671: def: OpR_RR_pat<C2_xor,         Xor, i1, I1>;
   672: def: OpR_RR_pat<C2_andn,  Not2<And>, i1, I1>;
   673: def: OpR_RR_pat<C2_orn,    Not2<Or>, i1, I1>;
   674: 
   675: def: AccRRR_pat<C4_and_and,   And,       Su<And>, I1, I1, I1>;
   676: def: AccRRR_pat<C4_and_or,    And,       Su< Or>, I1, I1, I1>;
   677: def: AccRRR_pat<C4_or_and,     Or,       Su<And>, I1, I1, I1>;
   678: def: AccRRR_pat<C4_or_or,      Or,       Su< Or>, I1, I1, I1>;
   679: def: AccRRR_pat<C4_and_andn,  And, Su<Not2<And>>, I1, I1, I1>;
   680: def: AccRRR_pat<C4_and_orn,   And, Su<Not2< Or>>, I1, I1, I1>;
   681: def: AccRRR_pat<C4_or_andn,    Or, Su<Not2<And>>, I1, I1, I1>;
   682: def: AccRRR_pat<C4_or_orn,     Or, Su<Not2< Or>>, I1, I1, I1>;
   683: 
   684: multiclass BoolvOpR_RR_pat<InstHexagon MI, PatFrag VOp> {
   685:   def: OpR_RR_pat<MI, VOp, v2i1, V2I1>;
   686:   def: OpR_RR_pat<MI, VOp, v4i1, V4I1>;
   687:   def: OpR_RR_pat<MI, VOp, v8i1, V8I1>;
   688: }
   689: 
   690: multiclass BoolvAccRRR_pat<InstHexagon MI, PatFrag AccOp, PatFrag VOp> {
   691:   def: AccRRR_pat<MI, AccOp, VOp, V2I1, V2I1, V2I1>;
   692:   def: AccRRR_pat<MI, AccOp, VOp, V4I1, V4I1, V4I1>;
   693:   def: AccRRR_pat<MI, AccOp, VOp, V8I1, V8I1, V8I1>;
   694: }
   695: 
   696: defm: BoolvOpR_RR_pat<C2_and,                    And>;
   697: defm: BoolvOpR_RR_pat<C2_or,                      Or>;
   698: defm: BoolvOpR_RR_pat<C2_xor,                    Xor>;
   699: defm: BoolvOpR_RR_pat<C2_andn,      VNot2<And, pnot>>;
   700: defm: BoolvOpR_RR_pat<C2_orn,       VNot2< Or, pnot>>;
   701: 
   702: // op(Ps, op(Pt, Pu))
   703: defm: BoolvAccRRR_pat<C4_and_and,   And, Su<And>>;
   704: defm: BoolvAccRRR_pat<C4_and_or,    And, Su<Or>>;
   705: defm: BoolvAccRRR_pat<C4_or_and,    Or,  Su<And>>;
   706: defm: BoolvAccRRR_pat<C4_or_or,     Or,  Su<Or>>;
   707: 
   708: // op(Ps, op(Pt, !Pu))
   709: defm: BoolvAccRRR_pat<C4_and_andn,  And, Su<VNot2<And, pnot>>>;
   710: defm: BoolvAccRRR_pat<C4_and_orn,   And, Su<VNot2< Or, pnot>>>;
   711: defm: BoolvAccRRR_pat<C4_or_andn,   Or,  Su<VNot2<And, pnot>>>;
   712: defm: BoolvAccRRR_pat<C4_or_orn,    Or,  Su<VNot2< Or, pnot>>>;
   713: 
   714: 
   715: // --(5) Compare ---------------------------------------------------------
   716: //
   717: 
   718: // Avoid negated comparisons, i.e. those of form "Pd = !cmp(...)".
   719: // These cannot form compounds (e.g. J4_cmpeqi_tp0_jump_nt).
   720: 
   721: def: OpR_RI_pat<C2_cmpeqi,    seteq,          i1, I32,  anyimm>;
   722: def: OpR_RI_pat<C2_cmpgti,    setgt,          i1, I32,  anyimm>;
   723: def: OpR_RI_pat<C2_cmpgtui,   setugt,         i1, I32,  anyimm>;
   724: 
   725: def: Pat<(i1 (setge I32:$Rs, s32_0ImmPred:$s10)),
   726:          (C2_cmpgti I32:$Rs, (SDEC1 imm:$s10))>;
   727: def: Pat<(i1 (setuge I32:$Rs, u32_0ImmPred:$u9)),
   728:          (C2_cmpgtui I32:$Rs, (UDEC1 imm:$u9))>;
   729: 
   730: def: Pat<(i1 (setlt I32:$Rs, s32_0ImmPred:$s10)),
   731:          (C2_not (C2_cmpgti I32:$Rs, (SDEC1 imm:$s10)))>;
   732: def: Pat<(i1 (setult I32:$Rs, u32_0ImmPred:$u9)),
   733:          (C2_not (C2_cmpgtui I32:$Rs, (UDEC1 imm:$u9)))>;
   734: 
   735: // Patfrag to convert the usual comparison patfrags (e.g. setlt) to ones
   736: // that reverse the order of the operands.
   737: class RevCmp<PatFrag F>
   738:   : PatFrag<(ops node:$rhs, node:$lhs), !head(F.Fragments), F.PredicateCode,
   739:             F.OperandTransform>;
   740: 
   741: def: OpR_RR_pat<C2_cmpeq,     seteq,          i1,   I32>;
   742: def: OpR_RR_pat<C2_cmpgt,     setgt,          i1,   I32>;
   743: def: OpR_RR_pat<C2_cmpgtu,    setugt,         i1,   I32>;
   744: def: OpR_RR_pat<C2_cmpgt,     RevCmp<setlt>,  i1,   I32>;
   745: def: OpR_RR_pat<C2_cmpgtu,    RevCmp<setult>, i1,   I32>;
   746: def: OpR_RR_pat<C2_cmpeqp,    seteq,          i1,   I64>;
   747: def: OpR_RR_pat<C2_cmpgtp,    setgt,          i1,   I64>;
   748: def: OpR_RR_pat<C2_cmpgtup,   setugt,         i1,   I64>;
   749: def: OpR_RR_pat<C2_cmpgtp,    RevCmp<setlt>,  i1,   I64>;
   750: def: OpR_RR_pat<C2_cmpgtup,   RevCmp<setult>, i1,   I64>;
   751: def: OpR_RR_pat<A2_vcmpbeq,   seteq,          v8i1, V8I8>;
   752: def: OpR_RR_pat<A4_vcmpbgt,   RevCmp<setlt>,  v8i1, V8I8>;
   753: def: OpR_RR_pat<A4_vcmpbgt,   setgt,          v8i1, V8I8>;
   754: def: OpR_RR_pat<A2_vcmpbgtu,  RevCmp<setult>, v8i1, V8I8>;
   755: def: OpR_RR_pat<A2_vcmpbgtu,  setugt,         v8i1, V8I8>;
   756: def: OpR_RR_pat<A2_vcmpheq,   seteq,          v4i1, V4I16>;
   757: def: OpR_RR_pat<A2_vcmphgt,   RevCmp<setlt>,  v4i1, V4I16>;
   758: def: OpR_RR_pat<A2_vcmphgt,   setgt,          v4i1, V4I16>;
   759: def: OpR_RR_pat<A2_vcmphgtu,  RevCmp<setult>, v4i1, V4I16>;
   760: def: OpR_RR_pat<A2_vcmphgtu,  setugt,         v4i1, V4I16>;
   761: def: OpR_RR_pat<A2_vcmpweq,   seteq,          v2i1, V2I32>;
   762: def: OpR_RR_pat<A2_vcmpwgt,   RevCmp<setlt>,  v2i1, V2I32>;
   763: def: OpR_RR_pat<A2_vcmpwgt,   setgt,          v2i1, V2I32>;
   764: def: OpR_RR_pat<A2_vcmpwgtu,  RevCmp<setult>, v2i1, V2I32>;
   765: def: OpR_RR_pat<A2_vcmpwgtu,  setugt,         v2i1, V2I32>;
   766: 
   767: def: OpR_RR_pat<F2_sfcmpeq,   seteq,          i1, F32>;
   768: def: OpR_RR_pat<F2_sfcmpgt,   setgt,          i1, F32>;
   769: def: OpR_RR_pat<F2_sfcmpge,   setge,          i1, F32>;
   770: def: OpR_RR_pat<F2_sfcmpgt,   RevCmp<setlt>,  i1, F32>;
   771: def: OpR_RR_pat<F2_sfcmpge,   RevCmp<setle>,  i1, F32>;
   772: def: OpR_RR_pat<F2_sfcmpuo,   setuo,          i1, F32>;
   773: 
   774: def: OpR_RR_pat<F2_dfcmpeq,   seteq,          i1, F64>;
   775: def: OpR_RR_pat<F2_dfcmpgt,   setgt,          i1, F64>;
   776: def: OpR_RR_pat<F2_dfcmpge,   setge,          i1, F64>;
   777: def: OpR_RR_pat<F2_dfcmpgt,   RevCmp<setlt>,  i1, F64>;
   778: def: OpR_RR_pat<F2_dfcmpge,   RevCmp<setle>,  i1, F64>;
   779: def: OpR_RR_pat<F2_dfcmpuo,   setuo,          i1, F64>;
   780: 
   781: // Avoid C4_cmpneqi, C4_cmpltei, C4_cmplteui, since they cannot form compounds.
   782: 
   783: def: Pat<(i1 (setne I32:$Rs, anyimm:$u5)),
   784:          (C2_not (C2_cmpeqi I32:$Rs, imm:$u5))>;
   785: def: Pat<(i1 (setle I32:$Rs, anyimm:$u5)),
   786:          (C2_not (C2_cmpgti I32:$Rs, imm:$u5))>;
   787: def: Pat<(i1 (setule I32:$Rs, anyimm:$u5)),
   788:          (C2_not (C2_cmpgtui I32:$Rs, imm:$u5))>;
   789: 
   790: class OpmR_RR_pat<PatFrag Output, PatFrag Op, ValueType ResType,
   791:                   PatFrag RsPred, PatFrag RtPred = RsPred>
   792:   : Pat<(ResType (Op RsPred:$Rs, RtPred:$Rt)),
   793:         (Output RsPred:$Rs, RtPred:$Rt)>;
   794: 
   795: class Outn<InstHexagon MI>
   796:   : OutPatFrag<(ops node:$Rs, node:$Rt),
   797:                (C2_not (MI $Rs, $Rt))>;
   798: 
   799: def: OpmR_RR_pat<Outn<C2_cmpeq>,    setne,          i1,   I32>;
   800: def: OpmR_RR_pat<Outn<C2_cmpgt>,    setle,          i1,   I32>;
```
- EN: It declares types such as RevCmp, OpmR_RR_pat, Outn, which carry the state or API of this component. It defines declarative TableGen records like BoolvOpR_RR_pat, BoolvAccRRR_pat, RevCmp, OpmR_RR_pat, Outn; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 RevCmp, OpmR_RR_pat, Outn 等类型，用来承载该组件的状态或接口。 这里定义了 BoolvOpR_RR_pat, BoolvAccRRR_pat, RevCmp, OpmR_RR_pat, Outn 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 801-1000 / 第 801-1000 行

```tablegen
   801: def: OpmR_RR_pat<Outn<C2_cmpgtu>,   setule,         i1,   I32>;
   802: def: OpmR_RR_pat<Outn<C2_cmpgt>,    RevCmp<setge>,  i1,   I32>;
   803: def: OpmR_RR_pat<Outn<C2_cmpgtu>,   RevCmp<setuge>, i1,   I32>;
   804: def: OpmR_RR_pat<Outn<C2_cmpeqp>,   setne,          i1,   I64>;
   805: def: OpmR_RR_pat<Outn<C2_cmpgtp>,   setle,          i1,   I64>;
   806: def: OpmR_RR_pat<Outn<C2_cmpgtup>,  setule,         i1,   I64>;
   807: def: OpmR_RR_pat<Outn<C2_cmpgtp>,   RevCmp<setge>,  i1,   I64>;
   808: def: OpmR_RR_pat<Outn<C2_cmpgtup>,  RevCmp<setuge>, i1,   I64>;
   809: def: OpmR_RR_pat<Outn<A2_vcmpbeq>,  setne,          v8i1, V8I8>;
   810: def: OpmR_RR_pat<Outn<A4_vcmpbgt>,  setle,          v8i1, V8I8>;
   811: def: OpmR_RR_pat<Outn<A2_vcmpbgtu>, setule,         v8i1, V8I8>;
   812: def: OpmR_RR_pat<Outn<A4_vcmpbgt>,  RevCmp<setge>,  v8i1, V8I8>;
   813: def: OpmR_RR_pat<Outn<A2_vcmpbgtu>, RevCmp<setuge>, v8i1, V8I8>;
   814: def: OpmR_RR_pat<Outn<A2_vcmpheq>,  setne,          v4i1, V4I16>;
   815: def: OpmR_RR_pat<Outn<A2_vcmphgt>,  setle,          v4i1, V4I16>;
   816: def: OpmR_RR_pat<Outn<A2_vcmphgtu>, setule,         v4i1, V4I16>;
   817: def: OpmR_RR_pat<Outn<A2_vcmphgt>,  RevCmp<setge>,  v4i1, V4I16>;
   818: def: OpmR_RR_pat<Outn<A2_vcmphgtu>, RevCmp<setuge>, v4i1, V4I16>;
   819: def: OpmR_RR_pat<Outn<A2_vcmpweq>,  setne,          v2i1, V2I32>;
   820: def: OpmR_RR_pat<Outn<A2_vcmpwgt>,  setle,          v2i1, V2I32>;
   821: def: OpmR_RR_pat<Outn<A2_vcmpwgtu>, setule,         v2i1, V2I32>;
   822: def: OpmR_RR_pat<Outn<A2_vcmpwgt>,  RevCmp<setge>,  v2i1, V2I32>;
   823: def: OpmR_RR_pat<Outn<A2_vcmpwgtu>, RevCmp<setuge>, v2i1, V2I32>;
   824: 
   825: let AddedComplexity = 100 in {
   826:   def: Pat<(i1 (seteq (and (xor I32:$Rs, I32:$Rt), 255), 0)),
   827:            (A4_cmpbeq IntRegs:$Rs, IntRegs:$Rt)>;
   828:   def: Pat<(i1 (setne (and (xor I32:$Rs, I32:$Rt), 255), 0)),
   829:            (C2_not (A4_cmpbeq IntRegs:$Rs, IntRegs:$Rt))>;
   830:   def: Pat<(i1 (seteq (and (xor I32:$Rs, I32:$Rt), 65535), 0)),
   831:            (A4_cmpheq IntRegs:$Rs, IntRegs:$Rt)>;
   832:   def: Pat<(i1 (setne (and (xor I32:$Rs, I32:$Rt), 65535), 0)),
   833:            (C2_not (A4_cmpheq IntRegs:$Rs, IntRegs:$Rt))>;
   834: }
   835: 
   836: // PatFrag for AsserZext which takes the original type as a parameter.
   837: def SDTAssertZext: SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisSameAs<0,1>]>;
   838: def AssertZextSD: SDNode<"ISD::AssertZext", SDTAssertZext>;
   839: class AssertZext<ValueType T>: PatFrag<(ops node:$A), (AssertZextSD $A, T)>;
   840: 
   841: multiclass Cmpb_pat<InstHexagon MI, PatFrag Op, PatFrag AssertExt,
   842:                       PatLeaf ImmPred, int Mask> {
   843:   def: Pat<(i1 (Op (and I32:$Rs, Mask), ImmPred:$I)),
   844:            (MI I32:$Rs, imm:$I)>;
   845:   def: Pat<(i1 (Op (AssertExt I32:$Rs), ImmPred:$I)),
   846:            (MI I32:$Rs, imm:$I)>;
   847: }
   848: 
   849: multiclass CmpbN_pat<InstHexagon MI, PatFrag Op, PatFrag AssertExt,
   850:                      PatLeaf ImmPred, int Mask> {
   851:   def: Pat<(i1 (Op (and I32:$Rs, Mask), ImmPred:$I)),
   852:            (C2_not (MI I32:$Rs, imm:$I))>;
   853:   def: Pat<(i1 (Op (AssertExt I32:$Rs), ImmPred:$I)),
   854:            (C2_not (MI I32:$Rs, imm:$I))>;
   855: }
   856: 
   857: multiclass CmpbND_pat<InstHexagon MI, PatFrag Op, PatFrag AssertExt,
   858:                       PatLeaf ImmPred, int Mask> {
   859:   def: Pat<(i1 (Op (and I32:$Rs, Mask), ImmPred:$I)),
   860:            (C2_not (MI I32:$Rs, (UDEC1 imm:$I)))>;
   861:   def: Pat<(i1 (Op (AssertExt I32:$Rs), ImmPred:$I)),
   862:            (C2_not (MI I32:$Rs, (UDEC1 imm:$I)))>;
   863: }
   864: 
   865: let AddedComplexity = 200 in {
   866:   defm: Cmpb_pat  <A4_cmpbeqi,  seteq,  AssertZext<i8>,  IsUGT<8,31>,  255>;
   867:   defm: CmpbN_pat <A4_cmpbeqi,  setne,  AssertZext<i8>,  IsUGT<8,31>,  255>;
   868:   defm: Cmpb_pat  <A4_cmpbgtui, setugt, AssertZext<i8>,  IsUGT<32,31>, 255>;
   869:   defm: CmpbN_pat <A4_cmpbgtui, setule, AssertZext<i8>,  IsUGT<32,31>, 255>;
   870:   defm: Cmpb_pat  <A4_cmphgtui, setugt, AssertZext<i16>, IsUGT<32,31>, 65535>;
   871:   defm: CmpbN_pat <A4_cmphgtui, setule, AssertZext<i16>, IsUGT<32,31>, 65535>;
   872:   defm: CmpbND_pat<A4_cmpbgtui, setult, AssertZext<i8>,  IsUGT<32,32>, 255>;
   873:   defm: CmpbND_pat<A4_cmphgtui, setult, AssertZext<i16>, IsUGT<32,32>, 65535>;
   874: }
   875: 
   876: def: Pat<(i32 (zext (i1 (seteq I32:$Rs, I32:$Rt)))),
   877:          (A4_rcmpeq I32:$Rs, I32:$Rt)>;
   878: def: Pat<(i32 (zext (i1 (setne I32:$Rs, I32:$Rt)))),
   879:          (A4_rcmpneq I32:$Rs, I32:$Rt)>;
   880: def: Pat<(i32 (zext (i1 (seteq I32:$Rs, anyimm:$s8)))),
   881:          (A4_rcmpeqi I32:$Rs, imm:$s8)>;
   882: def: Pat<(i32 (zext (i1 (setne I32:$Rs, anyimm:$s8)))),
   883:          (A4_rcmpneqi I32:$Rs, imm:$s8)>;
   884: 
   885: def: Pat<(i1 (seteq I1:$Ps, (i1 -1))), (I1:$Ps)>;
   886: def: Pat<(i1 (setne I1:$Ps, (i1 -1))), (C2_not I1:$Ps)>;
   887: def: Pat<(i1 (seteq I1:$Ps, I1:$Pt)),  (C2_not (C2_xor I1:$Ps, I1:$Pt))>;
   888: def: Pat<(i1 (setne I1:$Ps, I1:$Pt)),  (C2_xor I1:$Ps, I1:$Pt)>;
   889: 
   890: multiclass BoolE_pat<PatFrag OpPred, ValueType ResTy> {
   891:   def: Pat<(ResTy (seteq OpPred:$Ps, OpPred:$Pt)), (C2_not (C2_xor $Ps, $Pt))>;
   892:   def: Pat<(ResTy (setne OpPred:$Ps, OpPred:$Pt)), (C2_xor $Ps, $Pt)>;
   893: }
   894: 
   895: defm: BoolE_pat<I1,   i1>;
   896: defm: BoolE_pat<V2I1, v2i1>;
   897: defm: BoolE_pat<V4I1, v4i1>;
   898: defm: BoolE_pat<V8I1, v8i1>;
   899: 
   900: multiclass BoolL_pat<PatFrag OpPred, ValueType ResTy> {
   901:   // Signed "true" == -1
   902:   def: Pat<(ResTy (setlt  OpPred:$Ps, OpPred:$Pt)), (C2_andn $Ps, $Pt)>;
   903:   def: Pat<(ResTy (setle  OpPred:$Ps, OpPred:$Pt)), (C2_orn  $Ps, $Pt)>;
   904:   def: Pat<(ResTy (setult OpPred:$Ps, OpPred:$Pt)), (C2_andn $Pt, $Ps)>;
   905:   def: Pat<(ResTy (setule OpPred:$Ps, OpPred:$Pt)), (C2_orn  $Pt, $Ps)>;
   906: }
   907: 
   908: defm: BoolL_pat<I1,   i1>;
   909: defm: BoolL_pat<V2I1, v2i1>;
   910: defm: BoolL_pat<V4I1, v4i1>;
   911: defm: BoolL_pat<V8I1, v8i1>;
   912: 
   913: // Floating-point comparisons with checks for ordered/unordered status.
   914: 
   915: class T3<InstHexagon MI1, InstHexagon MI2, InstHexagon MI3>
   916:   : OutPatFrag<(ops node:$Rs, node:$Rt),
   917:                (MI1 (MI2 $Rs, $Rt), (MI3 $Rs, $Rt))>;
   918: 
   919: class Cmpuf<InstHexagon MI>:  T3<C2_or,  F2_sfcmpuo, MI>;
   920: class Cmpud<InstHexagon MI>:  T3<C2_or,  F2_dfcmpuo, MI>;
   921: 
   922: class Cmpufn<InstHexagon MI>: T3<C2_orn, F2_sfcmpuo, MI>;
   923: class Cmpudn<InstHexagon MI>: T3<C2_orn, F2_dfcmpuo, MI>;
   924: 
   925: def: OpmR_RR_pat<Cmpuf<F2_sfcmpeq>,  setueq,         i1, F32>;
   926: def: OpmR_RR_pat<Cmpuf<F2_sfcmpge>,  setuge,         i1, F32>;
   927: def: OpmR_RR_pat<Cmpuf<F2_sfcmpgt>,  setugt,         i1, F32>;
   928: def: OpmR_RR_pat<Cmpuf<F2_sfcmpge>,  RevCmp<setule>, i1, F32>;
   929: def: OpmR_RR_pat<Cmpuf<F2_sfcmpgt>,  RevCmp<setult>, i1, F32>;
   930: def: OpmR_RR_pat<Cmpufn<F2_sfcmpeq>, setune,         i1, F32>;
   931: 
   932: def: OpmR_RR_pat<Cmpud<F2_dfcmpeq>,  setueq,         i1, F64>;
   933: def: OpmR_RR_pat<Cmpud<F2_dfcmpge>,  setuge,         i1, F64>;
   934: def: OpmR_RR_pat<Cmpud<F2_dfcmpgt>,  setugt,         i1, F64>;
   935: def: OpmR_RR_pat<Cmpud<F2_dfcmpge>,  RevCmp<setule>, i1, F64>;
   936: def: OpmR_RR_pat<Cmpud<F2_dfcmpgt>,  RevCmp<setult>, i1, F64>;
   937: def: OpmR_RR_pat<Cmpudn<F2_dfcmpeq>, setune,         i1, F64>;
   938: 
   939: class T4<InstHexagon MI1, InstHexagon MI2, InstHexagon MI3, InstHexagon MI4>
   940:   : OutPatFrag<(ops node:$Rs, node:$Rt),
   941:                (MI1 (MI2 (MI3 $Rs, $Rt), (MI4 $Rs, $Rt)))>;
   942: 
   943: class Cmpof<InstHexagon MI>: T3<C2_andn, MI,  F2_sfcmpuo>;
   944: class Cmpod<InstHexagon MI>: T3<C2_andn, MI,  F2_dfcmpuo>;
   945: 
   946: class Cmpofn<InstHexagon MI>: T4<C2_not,  C2_or, MI,  F2_sfcmpuo>;
   947: class Cmpodn<InstHexagon MI>: T4<C2_not,  C2_or, MI,  F2_dfcmpuo>;
   948: 
   949: def: OpmR_RR_pat<Cmpof<F2_sfcmpeq>,  setoeq,         i1, F32>;
   950: def: OpmR_RR_pat<Cmpof<F2_sfcmpge>,  setoge,         i1, F32>;
   951: def: OpmR_RR_pat<Cmpof<F2_sfcmpgt>,  setogt,         i1, F32>;
   952: def: OpmR_RR_pat<Cmpof<F2_sfcmpge>,  RevCmp<setole>, i1, F32>;
   953: def: OpmR_RR_pat<Cmpof<F2_sfcmpgt>,  RevCmp<setolt>, i1, F32>;
   954: def: OpmR_RR_pat<Cmpofn<F2_sfcmpeq>, setone,         i1, F32>;
   955: 
   956: def: OpmR_RR_pat<Cmpod<F2_dfcmpeq>,  setoeq,         i1, F64>;
   957: def: OpmR_RR_pat<Cmpod<F2_dfcmpge>,  setoge,         i1, F64>;
   958: def: OpmR_RR_pat<Cmpod<F2_dfcmpgt>,  setogt,         i1, F64>;
   959: def: OpmR_RR_pat<Cmpod<F2_dfcmpge>,  RevCmp<setole>, i1, F64>;
   960: def: OpmR_RR_pat<Cmpod<F2_dfcmpgt>,  RevCmp<setolt>, i1, F64>;
   961: def: OpmR_RR_pat<Cmpodn<F2_dfcmpeq>, setone,         i1, F64>;
   962: 
   963: def: OpmR_RR_pat<Outn<F2_sfcmpuo>, seto,   i1, F32>;
   964: def: OpmR_RR_pat<Outn<F2_dfcmpuo>, seto,   i1, F64>;
   965: 
   966: def: OpmR_RR_pat<Outn<F2_sfcmpeq>, setne,  i1, F32>;
   967: def: OpmR_RR_pat<Outn<F2_dfcmpeq>, setne,  i1, F64>;
   968: 
   969: // --(6) Select ----------------------------------------------------------
   970: //
   971: 
   972: def: Pat<(select I1:$Pu, I32:$Rs, I32:$Rt),
   973:          (C2_mux I1:$Pu, I32:$Rs, I32:$Rt)>;
   974: def: Pat<(select I1:$Pu, v4i8:$Rs, v4i8:$Rt),
   975:          (C2_mux I1:$Pu, v4i8:$Rs, v4i8:$Rt)>;
   976: def: Pat<(select I1:$Pu, v2i16:$Rs, v2i16:$Rt),
   977:          (C2_mux I1:$Pu, v2i16:$Rs, v2i16:$Rt)>;
   978: def: Pat<(select I1:$Pu, anyimm:$s8, I32:$Rs),
   979:          (C2_muxri I1:$Pu, imm:$s8, I32:$Rs)>;
   980: def: Pat<(select I1:$Pu, I32:$Rs, anyimm:$s8),
   981:          (C2_muxir I1:$Pu, I32:$Rs, imm:$s8)>;
   982: def: Pat<(select I1:$Pu, anyimm:$s8, s8_0ImmPred:$S8),
   983:          (C2_muxii I1:$Pu, imm:$s8, imm:$S8)>;
   984: 
   985: def: Pat<(select (not I1:$Pu), I32:$Rs, I32:$Rt),
   986:          (C2_mux I1:$Pu, I32:$Rt, I32:$Rs)>;
   987: def: Pat<(select (not I1:$Pu), s8_0ImmPred:$S8, anyimm:$s8),
   988:          (C2_muxii I1:$Pu, imm:$s8, imm:$S8)>;
   989: def: Pat<(select (not I1:$Pu), anyimm:$s8, I32:$Rs),
   990:          (C2_muxir I1:$Pu, I32:$Rs, imm:$s8)>;
   991: def: Pat<(select (not I1:$Pu), I32:$Rs, anyimm:$s8),
   992:          (C2_muxri I1:$Pu, imm:$s8, I32:$Rs)>;
   993: 
   994: // Map from a 64-bit select to an emulated 64-bit mux.
   995: // Hexagon does not support 64-bit MUXes; so emulate with combines.
   996: def: Pat<(select I1:$Pu, I64:$Rs, I64:$Rt),
   997:          (Combinew (C2_mux I1:$Pu, (HiReg $Rs), (HiReg $Rt)),
   998:                    (C2_mux I1:$Pu, (LoReg $Rs), (LoReg $Rt)))>;
   999: 
  1000: def: Pat<(select I1:$Pu, v2i32:$Rs, v2i32:$Rt),
```
- EN: It declares types such as AssertZext, T3, Cmpuf, Cmpud, ... (11 total), which carry the state or API of this component. It defines declarative TableGen records like SDTAssertZext, AssertZextSD, AssertZext, Cmpb_pat, CmpbN_pat, ... (18 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 AssertZext, T3, Cmpuf, Cmpud, ... (11 total) 等类型，用来承载该组件的状态或接口。 这里定义了 SDTAssertZext, AssertZextSD, AssertZext, Cmpb_pat, CmpbN_pat, ... (18 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1001-1200 / 第 1001-1200 行

```tablegen
  1001:          (Combinew (C2_mux I1:$Pu, (HiReg $Rs), (HiReg $Rt)),
  1002:                    (C2_mux I1:$Pu, (LoReg $Rs), (LoReg $Rt)))>;
  1003: 
  1004: def: Pat<(select I1:$Pu, F32:$Rs, f32ImmPred:$I),
  1005:          (C2_muxir I1:$Pu, F32:$Rs, (ftoi $I))>;
  1006: def: Pat<(select I1:$Pu, f32ImmPred:$I, F32:$Rt),
  1007:          (C2_muxri I1:$Pu, (ftoi $I), F32:$Rt)>;
  1008: def: Pat<(select I1:$Pu, F32:$Rs, F32:$Rt),
  1009:          (C2_mux I1:$Pu, F32:$Rs, F32:$Rt)>;
  1010: def: Pat<(select I1:$Pu, F64:$Rs, F64:$Rt),
  1011:          (Combinew (C2_mux I1:$Pu, (HiReg $Rs), (HiReg $Rt)),
  1012:                    (C2_mux I1:$Pu, (LoReg $Rs), (LoReg $Rt)))>;
  1013: 
  1014: def: Pat<(select (i1 (setult F32:$Ra, F32:$Rb)), F32:$Rs, F32:$Rt),
  1015:          (C2_mux (F2_sfcmpgt F32:$Rb, F32:$Ra), F32:$Rs, F32:$Rt)>;
  1016: def: Pat<(select (i1 (setult F64:$Ra, F64:$Rb)), F64:$Rs, F64:$Rt),
  1017:          (C2_vmux (F2_dfcmpgt F64:$Rb, F64:$Ra), F64:$Rs, F64:$Rt)>;
  1018: 
  1019: def: Pat<(select (not I1:$Pu), f32ImmPred:$I, F32:$Rs),
  1020:          (C2_muxir I1:$Pu, F32:$Rs, (ftoi $I))>;
  1021: def: Pat<(select (not I1:$Pu), F32:$Rt, f32ImmPred:$I),
  1022:          (C2_muxri I1:$Pu, (ftoi $I), F32:$Rt)>;
  1023: 
  1024: def: Pat<(vselect V8I1:$Pu, V8I8:$Rs, V8I8:$Rt),
  1025:          (C2_vmux V8I1:$Pu, V8I8:$Rs, V8I8:$Rt)>;
  1026: def: Pat<(vselect V4I1:$Pu, V4I16:$Rs, V4I16:$Rt),
  1027:          (C2_vmux V4I1:$Pu, V4I16:$Rs, V4I16:$Rt)>;
  1028: def: Pat<(vselect V2I1:$Pu, V2I32:$Rs, V2I32:$Rt),
  1029:          (C2_vmux V2I1:$Pu, V2I32:$Rs, V2I32:$Rt)>;
  1030: 
  1031: def: Pat<(vselect (pnot V8I1:$Pu), V8I8:$Rs, V8I8:$Rt),
  1032:          (C2_vmux V8I1:$Pu, V8I8:$Rt, V8I8:$Rs)>;
  1033: def: Pat<(vselect (pnot V4I1:$Pu), V4I16:$Rs, V4I16:$Rt),
  1034:          (C2_vmux V4I1:$Pu, V4I16:$Rt, V4I16:$Rs)>;
  1035: def: Pat<(vselect (pnot V2I1:$Pu), V2I32:$Rs, V2I32:$Rt),
  1036:          (C2_vmux V2I1:$Pu, V2I32:$Rt, V2I32:$Rs)>;
  1037: 
  1038: 
  1039: // From LegalizeDAG.cpp: (Pu ? Pv : Pw) <=> (Pu & Pv) | (!Pu & Pw).
  1040: def: Pat<(select I1:$Pu, I1:$Ps, I1:$Pt),
  1041:          (C4_or_andn (C2_and $Ps, $Pu), $Pt, $Pu)>;
  1042: 
  1043: def: Pat<(vselect V2I1:$Pu, V2I1:$Ps, V2I1:$Pt),
  1044:          (C4_or_andn (C2_and $Ps, $Pu), $Pt, $Pu)>;
  1045: def: Pat<(vselect V4I1:$Pu, V4I1:$Ps, V4I1:$Pt),
  1046:          (C4_or_andn (C2_and $Ps, $Pu), $Pt, $Pu)>;
  1047: def: Pat<(vselect V8I1:$Pu, V8I1:$Ps, V8I1:$Pt),
  1048:          (C4_or_andn (C2_and $Ps, $Pu), $Pt, $Pu)>;
  1049: 
  1050: def: Pat<(select I1:$Pu, V2I1:$Ps, V2I1:$Pt),
  1051:          (C2_tfrrp (C2_mux $Pu, (C2_tfrpr $Ps), (C2_tfrpr $Pt)))>;
  1052: def: Pat<(select I1:$Pu, V4I1:$Ps, V4I1:$Pt),
  1053:          (C2_tfrrp (C2_mux $Pu, (C2_tfrpr $Ps), (C2_tfrpr $Pt)))>;
  1054: def: Pat<(select I1:$Pu, V8I1:$Ps, V8I1:$Pt),
  1055:          (C2_tfrrp (C2_mux $Pu, (C2_tfrpr $Ps), (C2_tfrpr $Pt)))>;
  1056: 
  1057: def IsPosHalf : PatLeaf<(i32 IntRegs:$a), [{
  1058:   return isPositiveHalfWord(N);
  1059: }]>;
  1060: 
  1061: multiclass SelMinMax16_pats<PatFrag CmpOp, InstHexagon InstA,
  1062:                             InstHexagon InstB> {
  1063:   def: Pat<(sext_inreg (select (i1 (CmpOp IsPosHalf:$Rs, IsPosHalf:$Rt)),
  1064:                                IsPosHalf:$Rs, IsPosHalf:$Rt), i16),
  1065:            (InstA IntRegs:$Rs, IntRegs:$Rt)>;
  1066:   def: Pat<(sext_inreg (select (i1 (CmpOp IsPosHalf:$Rs, IsPosHalf:$Rt)),
  1067:                                IsPosHalf:$Rt, IsPosHalf:$Rs), i16),
  1068:            (InstB IntRegs:$Rs, IntRegs:$Rt)>;
  1069: }
  1070: 
  1071: let AddedComplexity = 200 in {
  1072:   defm: SelMinMax16_pats<setge,  A2_max,  A2_min>;
  1073:   defm: SelMinMax16_pats<setgt,  A2_max,  A2_min>;
  1074:   defm: SelMinMax16_pats<setle,  A2_min,  A2_max>;
  1075:   defm: SelMinMax16_pats<setlt,  A2_min,  A2_max>;
  1076:   defm: SelMinMax16_pats<setuge, A2_maxu, A2_minu>;
  1077:   defm: SelMinMax16_pats<setugt, A2_maxu, A2_minu>;
  1078:   defm: SelMinMax16_pats<setule, A2_minu, A2_maxu>;
  1079:   defm: SelMinMax16_pats<setult, A2_minu, A2_maxu>;
  1080: }
  1081: 
  1082: def: OpR_RR_pat<A2_min,   Smin, i32, I32, I32>;
  1083: def: OpR_RR_pat<A2_max,   Smax, i32, I32, I32>;
  1084: def: OpR_RR_pat<A2_minu,  Umin, i32, I32, I32>;
  1085: def: OpR_RR_pat<A2_maxu,  Umax, i32, I32, I32>;
  1086: def: OpR_RR_pat<A2_minp,  Smin, i64, I64, I64>;
  1087: def: OpR_RR_pat<A2_maxp,  Smax, i64, I64, I64>;
  1088: def: OpR_RR_pat<A2_minup, Umin, i64, I64, I64>;
  1089: def: OpR_RR_pat<A2_maxup, Umax, i64, I64, I64>;
  1090: 
  1091: let AddedComplexity = 100 in {
  1092:   defm: MinMax_pats<F2_sfmin, F2_sfmax, select, setogt, i1, F32>;
  1093:   defm: MinMax_pats<F2_sfmin, F2_sfmax, select, setoge, i1, F32>;
  1094:   defm: MinMax_pats<F2_sfmax, F2_sfmin, select, setolt, i1, F32>;
  1095:   defm: MinMax_pats<F2_sfmax, F2_sfmin, select, setole, i1, F32>;
  1096: }
  1097: 
  1098: let AddedComplexity = 100, Predicates = [HasV67] in {
  1099:   defm: MinMax_pats<F2_dfmin, F2_dfmax, select, setogt, i1, F64>;
  1100:   defm: MinMax_pats<F2_dfmin, F2_dfmax, select, setoge, i1, F64>;
  1101:   defm: MinMax_pats<F2_dfmax, F2_dfmin, select, setolt, i1, F64>;
  1102:   defm: MinMax_pats<F2_dfmax, F2_dfmin, select, setole, i1, F64>;
  1103: }
  1104: 
  1105: def: OpR_RR_pat<A2_vminb,  Smin, v8i8,  V8I8>;
  1106: def: OpR_RR_pat<A2_vmaxb,  Smax, v8i8,  V8I8>;
  1107: def: OpR_RR_pat<A2_vminub, Umin, v8i8,  V8I8>;
  1108: def: OpR_RR_pat<A2_vmaxub, Umax, v8i8,  V8I8>;
  1109: 
  1110: def: OpR_RR_pat<A2_vminh,  Smin, v4i16, V4I16>;
  1111: def: OpR_RR_pat<A2_vmaxh,  Smax, v4i16, V4I16>;
  1112: def: OpR_RR_pat<A2_vminuh, Umin, v4i16, V4I16>;
  1113: def: OpR_RR_pat<A2_vmaxuh, Umax, v4i16, V4I16>;
  1114: 
  1115: def: OpR_RR_pat<A2_vminw,  Smin, v2i32, V2I32>;
  1116: def: OpR_RR_pat<A2_vmaxw,  Smax, v2i32, V2I32>;
  1117: def: OpR_RR_pat<A2_vminuw, Umin, v2i32, V2I32>;
  1118: def: OpR_RR_pat<A2_vmaxuw, Umax, v2i32, V2I32>;
  1119: 
  1120: // --(7) Insert/extract --------------------------------------------------
  1121: //
  1122: 
  1123: def SDTHexagonINSERT:
  1124:   SDTypeProfile<1, 4, [SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>,
  1125:                        SDTCisInt<0>, SDTCisVT<3, i32>, SDTCisVT<4, i32>]>;
  1126: def HexagonINSERT:    SDNode<"HexagonISD::INSERT",   SDTHexagonINSERT>;
  1127: 
  1128: let AddedComplexity = 10 in {
  1129:   def: Pat<(HexagonINSERT I32:$Rs, I32:$Rt, u5_0ImmPred:$u1, u5_0ImmPred:$u2),
  1130:            (S2_insert I32:$Rs, I32:$Rt, imm:$u1, imm:$u2)>;
  1131:   def: Pat<(HexagonINSERT I64:$Rs, I64:$Rt, u6_0ImmPred:$u1, u6_0ImmPred:$u2),
  1132:            (S2_insertp I64:$Rs, I64:$Rt, imm:$u1, imm:$u2)>;
  1133: }
  1134: def: Pat<(HexagonINSERT I32:$Rs, I32:$Rt, I32:$Width, I32:$Off),
  1135:          (S2_insert_rp I32:$Rs, I32:$Rt, (Combinew $Width, $Off))>;
  1136: def: Pat<(HexagonINSERT I64:$Rs, I64:$Rt, I32:$Width, I32:$Off),
  1137:          (S2_insertp_rp I64:$Rs, I64:$Rt, (Combinew $Width, $Off))>;
  1138: 
  1139: def SDTHexagonEXTRACTU
  1140:   : SDTypeProfile<1, 3, [SDTCisSameAs<0, 1>, SDTCisInt<0>, SDTCisInt<1>,
  1141:                   SDTCisVT<2, i32>, SDTCisVT<3, i32>]>;
  1142: def HexagonEXTRACTU:   SDNode<"HexagonISD::EXTRACTU",   SDTHexagonEXTRACTU>;
  1143: 
  1144: let AddedComplexity = 10 in {
  1145:   def: Pat<(HexagonEXTRACTU I32:$Rs, u5_0ImmPred:$u5, u5_0ImmPred:$U5),
  1146:            (S2_extractu I32:$Rs, imm:$u5, imm:$U5)>;
  1147:   def: Pat<(HexagonEXTRACTU I64:$Rs, u6_0ImmPred:$u6, u6_0ImmPred:$U6),
  1148:            (S2_extractup I64:$Rs, imm:$u6, imm:$U6)>;
  1149: }
  1150: def: Pat<(HexagonEXTRACTU I32:$Rs, I32:$Width, I32:$Off),
  1151:          (S2_extractu_rp I32:$Rs, (Combinew $Width, $Off))>;
  1152: def: Pat<(HexagonEXTRACTU I64:$Rs, I32:$Width, I32:$Off),
  1153:          (S2_extractup_rp I64:$Rs, (Combinew $Width, $Off))>;
  1154: 
  1155: def: Pat<(v4i8  (splat_vector anyint:$V)), (ToI32 (SplatB $V))>;
  1156: def: Pat<(v2i16 (splat_vector anyint:$V)), (ToI32 (SplatH $V))>;
  1157: def: Pat<(v8i8  (splat_vector anyint:$V)),
  1158:           (Combinew (ToI32 (SplatB $V)), (ToI32 (SplatB $V)))>;
  1159: def: Pat<(v4i16 (splat_vector anyint:$V)),
  1160:           (Combinew (ToI32 (SplatH $V)), (ToI32 (SplatH $V)))>;
  1161: let AddedComplexity = 10 in
  1162: def: Pat<(v2i32 (splat_vector s8_0ImmPred:$s8)),
  1163:          (A2_combineii imm:$s8, imm:$s8)>;
  1164: def: Pat<(v2i32 (splat_vector anyimm:$V)), (Combinew (ToI32 $V), (ToI32 $V))>;
  1165: 
  1166: def: Pat<(v4i8  (splat_vector I32:$Rs)), (S2_vsplatrb I32:$Rs)>;
  1167: def: Pat<(v2i16 (splat_vector I32:$Rs)), (LoReg (S2_vsplatrh I32:$Rs))>;
  1168: def: Pat<(v4i16 (splat_vector I32:$Rs)), (S2_vsplatrh I32:$Rs)>;
  1169: def: Pat<(v2i32 (splat_vector I32:$Rs)), (Combinew I32:$Rs, I32:$Rs)>;
  1170: 
  1171: let AddedComplexity = 10 in
  1172: def: Pat<(v8i8 (splat_vector I32:$Rs)), (S6_vsplatrbp I32:$Rs)>,
  1173:      Requires<[HasV62]>;
  1174: def: Pat<(v8i8 (splat_vector I32:$Rs)),
  1175:          (Combinew (S2_vsplatrb I32:$Rs), (S2_vsplatrb I32:$Rs))>;
  1176: 
  1177: let AddedComplexity = 10 in {
  1178:   def: Pat<(sext_inreg (HexagonEXTRACTU I32:$Rs,  8, u5_0ImmPred:$U5),  i8),
  1179:            (S4_extract  I32:$Rs,  8, imm:$U5)>;
  1180:   def: Pat<(sext_inreg (HexagonEXTRACTU I32:$Rs, 16, u5_0ImmPred:$U5), i16),
  1181:            (S4_extract  I32:$Rs, 16, imm:$U5)>;
  1182:   def: Pat<(sext_inreg (HexagonEXTRACTU I64:$Rs,  8, u6_0ImmPred:$U6),  i8),
  1183:            (S4_extractp I64:$Rs,  8, imm:$U6)>;
  1184:   def: Pat<(sext_inreg (HexagonEXTRACTU I64:$Rs, 16, u6_0ImmPred:$U6), i16),
  1185:            (S4_extractp I64:$Rs, 16, imm:$U6)>;
  1186:   def: Pat<(sext_inreg (HexagonEXTRACTU I64:$Rs, 32, u6_0ImmPred:$U6), i32),
  1187:            (S4_extractp I64:$Rs, 32, imm:$U6)>;
  1188: }
  1189: 
  1190: def: Pat<(sext_inreg (HexagonEXTRACTU I32:$Rs,  8, I32:$Off),  i8),
  1191:          (S4_extract_rp  I32:$Rs, (Combinew (ToI32 8), I32:$Off))>;
  1192: def: Pat<(sext_inreg (HexagonEXTRACTU I32:$Rs, 16, I32:$Off), i16),
  1193:          (S4_extract_rp  I32:$Rs, (Combinew (ToI32 16), I32:$Off))>;
  1194: def: Pat<(sext_inreg (HexagonEXTRACTU I64:$Rs,  8, I32:$Off),  i8),
  1195:          (S4_extractp_rp I64:$Rs, (Combinew (ToI32 8), I32:$Off))>;
  1196: def: Pat<(sext_inreg (HexagonEXTRACTU I64:$Rs, 16, I32:$Off), i16),
  1197:          (S4_extractp_rp I64:$Rs, (Combinew (ToI32 16), I32:$Off))>;
  1198: def: Pat<(sext_inreg (HexagonEXTRACTU I64:$Rs, 32, I32:$Off), i32),
  1199:          (S4_extractp_rp I64:$Rs, (Combinew (ToI32 32), I32:$Off))>;
  1200: 
```
- EN: It defines declarative TableGen records like IsPosHalf, SelMinMax16_pats, SDTHexagonINSERT, HexagonINSERT, SDTHexagonEXTRACTU, ... (6 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as isPositiveHalfWord, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonINSERT, HexagonISD, HexagonEXTRACTU, showing how the code connects to sibling backend components.
- CN: 这里定义了 IsPosHalf, SelMinMax16_pats, SDTHexagonINSERT, HexagonINSERT, SDTHexagonEXTRACTU, ... (6 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 isPositiveHalfWord 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonINSERT, HexagonISD, HexagonEXTRACTU，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```tablegen
  1201: 
  1202: // --(8) Shift/permute ---------------------------------------------------
  1203: //
  1204: 
  1205: def SDTHexagonI64I32I32: SDTypeProfile<1, 2,
  1206:   [SDTCisVT<0, i64>, SDTCisVT<1, i32>, SDTCisSameAs<1, 2>]>;
  1207: 
  1208: def HexagonCOMBINE:  SDNode<"HexagonISD::COMBINE",  SDTHexagonI64I32I32>;
  1209: 
  1210: def: Pat<(HexagonCOMBINE I32:$Rs, I32:$Rt), (Combinew $Rs, $Rt)>;
  1211: 
  1212: // The complexity of the combines involving immediates should be greater
  1213: // than the complexity of the combine with two registers.
  1214: let AddedComplexity = 50 in {
  1215:   def: Pat<(HexagonCOMBINE I32:$Rs, anyimm:$s8),
  1216:            (A4_combineri IntRegs:$Rs, imm:$s8)>;
  1217:   def: Pat<(HexagonCOMBINE anyimm:$s8, I32:$Rs),
  1218:            (A4_combineir imm:$s8, IntRegs:$Rs)>;
  1219: }
  1220: 
  1221: // The complexity of the combine with two immediates should be greater than
  1222: // the complexity of a combine involving a register.
  1223: let AddedComplexity = 75 in {
  1224:   def: Pat<(HexagonCOMBINE s8_0ImmPred:$s8, anyimm:$u6),
  1225:            (A4_combineii imm:$s8, imm:$u6)>;
  1226:   def: Pat<(HexagonCOMBINE anyimm:$s8, s8_0ImmPred:$S8),
  1227:            (A2_combineii imm:$s8, imm:$S8)>;
  1228: }
  1229: 
  1230: def: Pat<(bswap I32:$Rs),  (A2_swiz I32:$Rs)>;
  1231: def: Pat<(bswap I64:$Rss), (Combinew (A2_swiz (LoReg $Rss)),
  1232:                                      (A2_swiz (HiReg $Rss)))>;
  1233: 
  1234: def: Pat<(bswap V2I16:$Rs), (A2_combine_lh (A2_swiz $Rs), (A2_swiz $Rs))>;
  1235: def: Pat<(bswap V2I32:$Rs), (Combinew (A2_swiz (HiReg $Rs)),
  1236:                                       (A2_swiz (LoReg $Rs)))>;
  1237: def: Pat<(bswap V4I16:$Rs), (A2_orp (S2_lsr_i_vh $Rs, 8),
  1238:                                     (S2_asl_i_vh $Rs, 8))>;
  1239: 
  1240: def: Pat<(shl s6_0ImmPred:$s6, I32:$Rt),  (S4_lsli imm:$s6, I32:$Rt)>;
  1241: def: Pat<(shl I32:$Rs, (i32 16)),         (A2_aslh I32:$Rs)>;
  1242: def: Pat<(sra I32:$Rs, (i32 16)),         (A2_asrh I32:$Rs)>;
  1243: 
  1244: def: OpR_RI_pat<S2_asr_i_r,  Sra, i32,   I32,   u5_0ImmPred>;
  1245: def: OpR_RI_pat<S2_lsr_i_r,  Srl, i32,   I32,   u5_0ImmPred>;
  1246: def: OpR_RI_pat<S2_asl_i_r,  Shl, i32,   I32,   u5_0ImmPred>;
  1247: def: OpR_RI_pat<S2_asr_i_p,  Sra, i64,   I64,   u6_0ImmPred>;
  1248: def: OpR_RI_pat<S2_lsr_i_p,  Srl, i64,   I64,   u6_0ImmPred>;
  1249: def: OpR_RI_pat<S2_asl_i_p,  Shl, i64,   I64,   u6_0ImmPred>;
  1250: 
  1251: def: OpR_RR_pat<S2_asr_r_r, Sra, i32, I32, I32>;
  1252: def: OpR_RR_pat<S2_lsr_r_r, Srl, i32, I32, I32>;
  1253: def: OpR_RR_pat<S2_asl_r_r, Shl, i32, I32, I32>;
  1254: def: OpR_RR_pat<S2_asr_r_p, Sra, i64, I64, I32>;
  1255: def: OpR_RR_pat<S2_lsr_r_p, Srl, i64, I64, I32>;
  1256: def: OpR_RR_pat<S2_asl_r_p, Shl, i64, I64, I32>;
  1257: 
  1258: // Funnel shifts.
  1259: def IsMul8_U3: PatLeaf<(i32 imm), [{
  1260:   uint64_t V = N->getZExtValue();
  1261:   return V % 8 == 0 && isUInt<3>(V / 8);
  1262: }]>;
  1263: 
  1264: def Divu8: SDNodeXForm<imm, [{
  1265:   return CurDAG->getTargetConstant(N->getZExtValue() / 8, SDLoc(N), MVT::i32);
  1266: }]>;
  1267: 
  1268: // Funnel shift-left.
  1269: def FShl32i: OutPatFrag<(ops node:$Rs, node:$Rt, node:$S),
  1270:   (HiReg (S2_asl_i_p (Combinew $Rs, $Rt), $S))>;
  1271: def FShl32r: OutPatFrag<(ops node:$Rs, node:$Rt, node:$Ru),
  1272:   (HiReg (S2_asl_r_p (Combinew $Rs, $Rt), $Ru))>;
  1273: 
  1274: def FShl64i: OutPatFrag<(ops node:$Rs, node:$Rt, node:$S),
  1275:   (S2_lsr_i_p_or (S2_asl_i_p $Rs, $S),  $Rt, (Subi<64> $S))>;
  1276: def FShl64r: OutPatFrag<(ops node:$Rs, node:$Rt, node:$Ru),
  1277:   (S2_lsl_r_p_or (S2_asl_r_p $Rs, (A2_andir $Ru, 63)),
  1278:                   $Rt,
  1279:                   (A2_addi (A2_andir $Ru, 63), -64))>;
  1280: 
  1281: // Combined SDNodeXForm: (Divu8 (Subi<64> $S))
  1282: def Divu64_8: SDNodeXForm<imm, [{
  1283:   return CurDAG->getTargetConstant((64 - N->getSExtValue()) / 8,
  1284:                                    SDLoc(N), MVT::i32);
  1285: }]>;
  1286: 
  1287: // Special cases:
  1288: let AddedComplexity = 100 in {
  1289:   def: Pat<(fshl I32:$Rs, I32:$Rt, (i32 16)),
  1290:            (A2_combine_lh I32:$Rs, I32:$Rt)>;
  1291:   def: Pat<(fshl I64:$Rs, I64:$Rt, IsMul8_U3:$S),
  1292:            (S2_valignib I64:$Rs, I64:$Rt, (Divu64_8 $S))>;
  1293: }
  1294: 
  1295: let Predicates = [HasV60], AddedComplexity = 50 in {
  1296:   def: OpR_RI_pat<S6_rol_i_r, Rol, i32, I32, u5_0ImmPred>;
  1297:   def: OpR_RI_pat<S6_rol_i_p, Rol, i64, I64, u6_0ImmPred>;
  1298: }
  1299: let AddedComplexity = 30 in {
  1300:   def: Pat<(rotl I32:$Rs, u5_0ImmPred:$S),          (FShl32i $Rs, $Rs, imm:$S)>;
  1301:   def: Pat<(rotl I64:$Rs, u6_0ImmPred:$S),          (FShl64i $Rs, $Rs, imm:$S)>;
  1302:   def: Pat<(fshl I32:$Rs, I32:$Rt, u5_0ImmPred:$S), (FShl32i $Rs, $Rt, imm:$S)>;
  1303:   def: Pat<(fshl I64:$Rs, I64:$Rt, u6_0ImmPred:$S), (FShl64i $Rs, $Rt, imm:$S)>;
  1304: }
  1305: def: Pat<(rotl I32:$Rs, I32:$Rt),           (FShl32r $Rs, $Rs, $Rt)>;
  1306: def: Pat<(rotl I64:$Rs, I32:$Rt),           (FShl64r $Rs, $Rs, $Rt)>;
  1307: def: Pat<(fshl I32:$Rs, I32:$Rt, I32:$Ru),  (FShl32r $Rs, $Rt, $Ru)>;
  1308: def: Pat<(fshl I64:$Rs, I64:$Rt, I32:$Ru),  (FShl64r $Rs, $Rt, $Ru)>;
  1309: 
  1310: // Funnel shift-right.
  1311: def FShr32i: OutPatFrag<(ops node:$Rs, node:$Rt, node:$S),
  1312:   (LoReg (S2_lsr_i_p (Combinew $Rs, $Rt), $S))>;
  1313: def FShr32r: OutPatFrag<(ops node:$Rs, node:$Rt, node:$Ru),
  1314:   (LoReg (S2_lsr_r_p (Combinew $Rs, $Rt), $Ru))>;
  1315: 
  1316: def FShr64i: OutPatFrag<(ops node:$Rs, node:$Rt, node:$S),
  1317:   (S2_asl_i_p_or (S2_lsr_i_p $Rt, $S),  $Rs, (Subi<64> $S))>;
  1318: def FShr64r: OutPatFrag<(ops node:$Rs, node:$Rt, node:$Ru),
  1319:   (S2_lsr_r_p_or (S2_lsr_r_p $Rt, (A2_andir $Ru, 63)),
  1320:                   $Rs,
  1321:                   (A2_addi (A2_andir $Ru, 63), -64))>;
  1322: 
  1323: // Special cases:
  1324: let AddedComplexity = 100 in {
  1325:   def: Pat<(fshr I32:$Rs, I32:$Rt, (i32 16)),
  1326:            (A2_combine_lh I32:$Rs, I32:$Rt)>;
  1327:   def: Pat<(fshr I64:$Rs, I64:$Rt, IsMul8_U3:$S),
  1328:            (S2_valignib I64:$Rs, I64:$Rt, (Divu8 $S))>;
  1329: }
  1330: 
  1331: let Predicates = [HasV60], AddedComplexity = 50 in {
  1332:   def: Pat<(rotr I32:$Rs, u5_0ImmPred:$S), (S6_rol_i_r I32:$Rs, (Subi<32> $S))>;
  1333:   def: Pat<(rotr I64:$Rs, u6_0ImmPred:$S), (S6_rol_i_p I64:$Rs, (Subi<64> $S))>;
  1334: }
  1335: let AddedComplexity = 30 in {
  1336:   def: Pat<(rotr I32:$Rs, u5_0ImmPred:$S),          (FShr32i $Rs, $Rs, imm:$S)>;
  1337:   def: Pat<(rotr I64:$Rs, u6_0ImmPred:$S),          (FShr64i $Rs, $Rs, imm:$S)>;
  1338:   def: Pat<(fshr I32:$Rs, I32:$Rt, u5_0ImmPred:$S), (FShr32i $Rs, $Rt, imm:$S)>;
  1339:   def: Pat<(fshr I64:$Rs, I64:$Rt, u6_0ImmPred:$S), (FShr64i $Rs, $Rt, imm:$S)>;
  1340: }
  1341: def: Pat<(rotr I32:$Rs, I32:$Rt),           (FShr32r $Rs, $Rs, $Rt)>;
  1342: def: Pat<(rotr I64:$Rs, I32:$Rt),           (FShr64r $Rs, $Rs, $Rt)>;
  1343: def: Pat<(fshr I32:$Rs, I32:$Rt, I32:$Ru),  (FShr32r $Rs, $Rt, $Ru)>;
  1344: def: Pat<(fshr I64:$Rs, I64:$Rt, I32:$Ru),  (FShr64r $Rs, $Rt, $Ru)>;
  1345: 
  1346: 
  1347: def: Pat<(sra (add (sra I32:$Rs, u5_0ImmPred:$u5), 1), (i32 1)),
  1348:          (S2_asr_i_r_rnd I32:$Rs, imm:$u5)>;
  1349: def: Pat<(sra (add (sra I64:$Rs, u6_0ImmPred:$u6), 1), (i32 1)),
  1350:          (S2_asr_i_p_rnd I64:$Rs, imm:$u6)>;
  1351: 
  1352: // Prefer S2_addasl_rrri over S2_asl_i_r_acc.
  1353: let AddedComplexity = 120 in
  1354: def: Pat<(add I32:$Rt, (shl I32:$Rs, u3_0ImmPred:$u3)),
  1355:          (S2_addasl_rrri IntRegs:$Rt, IntRegs:$Rs, imm:$u3)>;
  1356: 
  1357: let AddedComplexity = 100 in {
  1358:   def: AccRRI_pat<S2_asr_i_r_acc,   Add, Su<Sra>, I32, u5_0ImmPred>;
  1359:   def: AccRRI_pat<S2_asr_i_r_nac,   Sub, Su<Sra>, I32, u5_0ImmPred>;
  1360:   def: AccRRI_pat<S2_asr_i_r_and,   And, Su<Sra>, I32, u5_0ImmPred>;
  1361:   def: AccRRI_pat<S2_asr_i_r_or,    Or,  Su<Sra>, I32, u5_0ImmPred>;
  1362: 
  1363:   def: AccRRI_pat<S2_asr_i_p_acc,   Add, Su<Sra>, I64, u6_0ImmPred>;
  1364:   def: AccRRI_pat<S2_asr_i_p_nac,   Sub, Su<Sra>, I64, u6_0ImmPred>;
  1365:   def: AccRRI_pat<S2_asr_i_p_and,   And, Su<Sra>, I64, u6_0ImmPred>;
  1366:   def: AccRRI_pat<S2_asr_i_p_or,    Or,  Su<Sra>, I64, u6_0ImmPred>;
  1367: 
  1368:   def: AccRRI_pat<S2_lsr_i_r_acc,   Add, Su<Srl>, I32, u5_0ImmPred>;
  1369:   def: AccRRI_pat<S2_lsr_i_r_nac,   Sub, Su<Srl>, I32, u5_0ImmPred>;
  1370:   def: AccRRI_pat<S2_lsr_i_r_and,   And, Su<Srl>, I32, u5_0ImmPred>;
  1371:   def: AccRRI_pat<S2_lsr_i_r_or,    Or,  Su<Srl>, I32, u5_0ImmPred>;
  1372:   def: AccRRI_pat<S2_lsr_i_r_xacc,  Xor, Su<Srl>, I32, u5_0ImmPred>;
  1373: 
  1374:   def: AccRRI_pat<S2_lsr_i_p_acc,   Add, Su<Srl>, I64, u6_0ImmPred>;
  1375:   def: AccRRI_pat<S2_lsr_i_p_nac,   Sub, Su<Srl>, I64, u6_0ImmPred>;
  1376:   def: AccRRI_pat<S2_lsr_i_p_and,   And, Su<Srl>, I64, u6_0ImmPred>;
  1377:   def: AccRRI_pat<S2_lsr_i_p_or,    Or,  Su<Srl>, I64, u6_0ImmPred>;
  1378:   def: AccRRI_pat<S2_lsr_i_p_xacc,  Xor, Su<Srl>, I64, u6_0ImmPred>;
  1379: 
  1380:   def: AccRRI_pat<S2_asl_i_r_acc,   Add, Su<Shl>, I32, u5_0ImmPred>;
  1381:   def: AccRRI_pat<S2_asl_i_r_nac,   Sub, Su<Shl>, I32, u5_0ImmPred>;
  1382:   def: AccRRI_pat<S2_asl_i_r_and,   And, Su<Shl>, I32, u5_0ImmPred>;
  1383:   def: AccRRI_pat<S2_asl_i_r_or,    Or,  Su<Shl>, I32, u5_0ImmPred>;
  1384:   def: AccRRI_pat<S2_asl_i_r_xacc,  Xor, Su<Shl>, I32, u5_0ImmPred>;
  1385: 
  1386:   def: AccRRI_pat<S2_asl_i_p_acc,   Add, Su<Shl>, I64, u6_0ImmPred>;
  1387:   def: AccRRI_pat<S2_asl_i_p_nac,   Sub, Su<Shl>, I64, u6_0ImmPred>;
  1388:   def: AccRRI_pat<S2_asl_i_p_and,   And, Su<Shl>, I64, u6_0ImmPred>;
  1389:   def: AccRRI_pat<S2_asl_i_p_or,    Or,  Su<Shl>, I64, u6_0ImmPred>;
  1390:   def: AccRRI_pat<S2_asl_i_p_xacc,  Xor, Su<Shl>, I64, u6_0ImmPred>;
  1391: 
  1392:   let Predicates = [HasV60] in {
  1393:     def: AccRRI_pat<S6_rol_i_r_acc,   Add, Su<Rol>, I32, u5_0ImmPred>;
  1394:     def: AccRRI_pat<S6_rol_i_r_nac,   Sub, Su<Rol>, I32, u5_0ImmPred>;
  1395:     def: AccRRI_pat<S6_rol_i_r_and,   And, Su<Rol>, I32, u5_0ImmPred>;
  1396:     def: AccRRI_pat<S6_rol_i_r_or,    Or,  Su<Rol>, I32, u5_0ImmPred>;
  1397:     def: AccRRI_pat<S6_rol_i_r_xacc,  Xor, Su<Rol>, I32, u5_0ImmPred>;
  1398: 
  1399:     def: AccRRI_pat<S6_rol_i_p_acc,   Add, Su<Rol>, I64, u6_0ImmPred>;
  1400:     def: AccRRI_pat<S6_rol_i_p_nac,   Sub, Su<Rol>, I64, u6_0ImmPred>;
```
- EN: It defines declarative TableGen records like SDTHexagonI64I32I32, HexagonCOMBINE, IsMul8_U3, Divu8, FShl32i, ... (13 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getZExtValue, isUInt<3>, getTargetConstant, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonCOMBINE, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里定义了 SDTHexagonI64I32I32, HexagonCOMBINE, IsMul8_U3, Divu8, FShl32i, ... (13 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getZExtValue, isUInt<3>, getTargetConstant 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonCOMBINE, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 1401-1600 / 第 1401-1600 行

```tablegen
  1401:     def: AccRRI_pat<S6_rol_i_p_and,   And, Su<Rol>, I64, u6_0ImmPred>;
  1402:     def: AccRRI_pat<S6_rol_i_p_or,    Or,  Su<Rol>, I64, u6_0ImmPred>;
  1403:     def: AccRRI_pat<S6_rol_i_p_xacc,  Xor, Su<Rol>, I64, u6_0ImmPred>;
  1404:   }
  1405: }
  1406: 
  1407: let AddedComplexity = 100 in {
  1408:   def: AccRRR_pat<S2_asr_r_r_acc,   Add, Su<Sra>, I32, I32, I32>;
  1409:   def: AccRRR_pat<S2_asr_r_r_nac,   Sub, Su<Sra>, I32, I32, I32>;
  1410:   def: AccRRR_pat<S2_asr_r_r_and,   And, Su<Sra>, I32, I32, I32>;
  1411:   def: AccRRR_pat<S2_asr_r_r_or,    Or,  Su<Sra>, I32, I32, I32>;
  1412: 
  1413:   def: AccRRR_pat<S2_asr_r_p_acc,   Add, Su<Sra>, I64, I64, I32>;
  1414:   def: AccRRR_pat<S2_asr_r_p_nac,   Sub, Su<Sra>, I64, I64, I32>;
  1415:   def: AccRRR_pat<S2_asr_r_p_and,   And, Su<Sra>, I64, I64, I32>;
  1416:   def: AccRRR_pat<S2_asr_r_p_or,    Or,  Su<Sra>, I64, I64, I32>;
  1417:   def: AccRRR_pat<S2_asr_r_p_xor,   Xor, Su<Sra>, I64, I64, I32>;
  1418: 
  1419:   def: AccRRR_pat<S2_lsr_r_r_acc,   Add, Su<Srl>, I32, I32, I32>;
  1420:   def: AccRRR_pat<S2_lsr_r_r_nac,   Sub, Su<Srl>, I32, I32, I32>;
  1421:   def: AccRRR_pat<S2_lsr_r_r_and,   And, Su<Srl>, I32, I32, I32>;
  1422:   def: AccRRR_pat<S2_lsr_r_r_or,    Or,  Su<Srl>, I32, I32, I32>;
  1423: 
  1424:   def: AccRRR_pat<S2_lsr_r_p_acc,   Add, Su<Srl>, I64, I64, I32>;
  1425:   def: AccRRR_pat<S2_lsr_r_p_nac,   Sub, Su<Srl>, I64, I64, I32>;
  1426:   def: AccRRR_pat<S2_lsr_r_p_and,   And, Su<Srl>, I64, I64, I32>;
  1427:   def: AccRRR_pat<S2_lsr_r_p_or,    Or,  Su<Srl>, I64, I64, I32>;
  1428:   def: AccRRR_pat<S2_lsr_r_p_xor,   Xor, Su<Srl>, I64, I64, I32>;
  1429: 
  1430:   def: AccRRR_pat<S2_asl_r_r_acc,   Add, Su<Shl>, I32, I32, I32>;
  1431:   def: AccRRR_pat<S2_asl_r_r_nac,   Sub, Su<Shl>, I32, I32, I32>;
  1432:   def: AccRRR_pat<S2_asl_r_r_and,   And, Su<Shl>, I32, I32, I32>;
  1433:   def: AccRRR_pat<S2_asl_r_r_or,    Or,  Su<Shl>, I32, I32, I32>;
  1434: 
  1435:   def: AccRRR_pat<S2_asl_r_p_acc,   Add, Su<Shl>, I64, I64, I32>;
  1436:   def: AccRRR_pat<S2_asl_r_p_nac,   Sub, Su<Shl>, I64, I64, I32>;
  1437:   def: AccRRR_pat<S2_asl_r_p_and,   And, Su<Shl>, I64, I64, I32>;
  1438:   def: AccRRR_pat<S2_asl_r_p_or,    Or,  Su<Shl>, I64, I64, I32>;
  1439:   def: AccRRR_pat<S2_asl_r_p_xor,   Xor, Su<Shl>, I64, I64, I32>;
  1440: }
  1441: 
  1442: 
  1443: class OpshIRI_pat<InstHexagon MI, PatFrag Op, PatFrag ShOp,
  1444:                   PatFrag RegPred, PatFrag ImmPred>
  1445:   : Pat<(Op anyimm:$u8, (ShOp RegPred:$Rs, ImmPred:$U5)),
  1446:         (MI anyimm:$u8, RegPred:$Rs, imm:$U5)>;
  1447: 
  1448: let AddedComplexity = 200, Predicates = [UseCompound] in {
  1449:   def: OpshIRI_pat<S4_addi_asl_ri,  Add, Su<Shl>, I32, u5_0ImmPred>;
  1450:   def: OpshIRI_pat<S4_addi_lsr_ri,  Add, Su<Srl>, I32, u5_0ImmPred>;
  1451:   def: OpshIRI_pat<S4_subi_asl_ri,  Sub, Su<Shl>, I32, u5_0ImmPred>;
  1452:   def: OpshIRI_pat<S4_subi_lsr_ri,  Sub, Su<Srl>, I32, u5_0ImmPred>;
  1453:   def: OpshIRI_pat<S4_andi_asl_ri,  And, Su<Shl>, I32, u5_0ImmPred>;
  1454:   def: OpshIRI_pat<S4_andi_lsr_ri,  And, Su<Srl>, I32, u5_0ImmPred>;
  1455:   def: OpshIRI_pat<S4_ori_asl_ri,   Or,  Su<Shl>, I32, u5_0ImmPred>;
  1456:   def: OpshIRI_pat<S4_ori_lsr_ri,   Or,  Su<Srl>, I32, u5_0ImmPred>;
  1457: }
  1458: 
  1459: // Prefer this pattern to S2_asl_i_p_or for the special case of joining
  1460: // two 32-bit words into a 64-bit word.
  1461: let AddedComplexity = 200 in
  1462: def: Pat<(or (shl (Aext64 I32:$a), (i32 32)), (Zext64 I32:$b)),
  1463:          (Combinew I32:$a, I32:$b)>;
  1464: 
  1465: def: Pat<(or (or (or (shl (Zext64 (and I32:$b, (i32 65535))), (i32 16)),
  1466:                      (Zext64 (and I32:$a, (i32 65535)))),
  1467:                  (shl (Aext64 (and I32:$c, (i32 65535))), (i32 32))),
  1468:              (shl (Aext64 I32:$d), (i32 48))),
  1469:          (Combinew (A2_combine_ll I32:$d, I32:$c),
  1470:                    (A2_combine_ll I32:$b, I32:$a))>;
  1471: 
  1472: let AddedComplexity = 200 in {
  1473:   def: Pat<(or (shl I32:$Rt, (i32 16)), (and I32:$Rs, (i32 65535))),
  1474:            (A2_combine_ll I32:$Rt, I32:$Rs)>;
  1475:   def: Pat<(or (shl I32:$Rt, (i32 16)), (srl I32:$Rs, (i32 16))),
  1476:            (A2_combine_lh I32:$Rt, I32:$Rs)>;
  1477:   def: Pat<(or (and I32:$Rt, (i32 268431360)), (and I32:$Rs, (i32 65535))),
  1478:            (A2_combine_hl I32:$Rt, I32:$Rs)>;
  1479:   def: Pat<(or (and I32:$Rt, (i32 268431360)), (srl I32:$Rs, (i32 16))),
  1480:            (A2_combine_hh I32:$Rt, I32:$Rs)>;
  1481: }
  1482: 
  1483: def SDTHexagonVShift
  1484:   : SDTypeProfile<1, 2, [SDTCisSameAs<0, 1>, SDTCisVec<0>, SDTCisVT<2, i32>]>;
  1485: 
  1486: // Vector shifts by a scalar value
  1487: def HexagonVASL: SDNode<"HexagonISD::VASL", SDTHexagonVShift>;
  1488: def HexagonVASR: SDNode<"HexagonISD::VASR", SDTHexagonVShift>;
  1489: def HexagonVLSR: SDNode<"HexagonISD::VLSR", SDTHexagonVShift>;
  1490: 
  1491: // Funnel shifts with the shift amount module element bit width.
  1492: def HexagonMFSHL: SDNode<"HexagonISD::MFSHL", SDTIntShiftDOp>;
  1493: def HexagonMFSHR: SDNode<"HexagonISD::MFSHR", SDTIntShiftDOp>;
  1494: 
  1495: def: OpR_RI_pat<S2_asl_i_vw, pf2<HexagonVASL>, v2i32, V2I32, u5_0ImmPred>;
  1496: def: OpR_RI_pat<S2_asl_i_vh, pf2<HexagonVASL>, v4i16, V4I16, u4_0ImmPred>;
  1497: def: OpR_RI_pat<S2_asr_i_vw, pf2<HexagonVASR>, v2i32, V2I32, u5_0ImmPred>;
  1498: def: OpR_RI_pat<S2_asr_i_vh, pf2<HexagonVASR>, v4i16, V4I16, u4_0ImmPred>;
  1499: def: OpR_RI_pat<S2_lsr_i_vw, pf2<HexagonVLSR>, v2i32, V2I32, u5_0ImmPred>;
  1500: def: OpR_RI_pat<S2_lsr_i_vh, pf2<HexagonVLSR>, v4i16, V4I16, u4_0ImmPred>;
  1501: 
  1502: def: OpR_RR_pat<S2_asl_r_vw, pf2<HexagonVASL>, v2i32, V2I32, I32>;
  1503: def: OpR_RR_pat<S2_asl_r_vh, pf2<HexagonVASL>, v4i16, V4I16, I32>;
  1504: def: OpR_RR_pat<S2_asr_r_vw, pf2<HexagonVASR>, v2i32, V2I32, I32>;
  1505: def: OpR_RR_pat<S2_asr_r_vh, pf2<HexagonVASR>, v4i16, V4I16, I32>;
  1506: def: OpR_RR_pat<S2_lsr_r_vw, pf2<HexagonVLSR>, v2i32, V2I32, I32>;
  1507: def: OpR_RR_pat<S2_lsr_r_vh, pf2<HexagonVLSR>, v4i16, V4I16, I32>;
  1508: 
  1509: def: Pat<(sra V2I32:$b, (v2i32 (splat_vector u5_0ImmPred:$c))),
  1510:          (S2_asr_i_vw V2I32:$b, imm:$c)>;
  1511: def: Pat<(srl V2I32:$b, (v2i32 (splat_vector u5_0ImmPred:$c))),
  1512:          (S2_lsr_i_vw V2I32:$b, imm:$c)>;
  1513: def: Pat<(shl V2I32:$b, (v2i32 (splat_vector u5_0ImmPred:$c))),
  1514:          (S2_asl_i_vw V2I32:$b, imm:$c)>;
  1515: def: Pat<(sra V4I16:$b, (v4i16 (splat_vector u4_0ImmPred:$c))),
  1516:          (S2_asr_i_vh V4I16:$b, imm:$c)>;
  1517: def: Pat<(srl V4I16:$b, (v4i16 (splat_vector u4_0ImmPred:$c))),
  1518:          (S2_lsr_i_vh V4I16:$b, imm:$c)>;
  1519: def: Pat<(shl V4I16:$b, (v4i16 (splat_vector u4_0ImmPred:$c))),
  1520:          (S2_asl_i_vh V4I16:$b, imm:$c)>;
  1521: 
  1522: def: Pat<(HexagonVASR V2I16:$Rs, u4_0ImmPred:$S),
  1523:          (LoReg (S2_asr_i_vh (ToAext64 $Rs), imm:$S))>;
  1524: def: Pat<(HexagonVASL V2I16:$Rs, u4_0ImmPred:$S),
  1525:          (LoReg (S2_asl_i_vh (ToAext64 $Rs), imm:$S))>;
  1526: def: Pat<(HexagonVLSR V2I16:$Rs, u4_0ImmPred:$S),
  1527:          (LoReg (S2_lsr_i_vh (ToAext64 $Rs), imm:$S))>;
  1528: def: Pat<(HexagonVASR V2I16:$Rs, I32:$Rt),
  1529:          (LoReg (S2_asr_i_vh (ToAext64 $Rs), I32:$Rt))>;
  1530: def: Pat<(HexagonVASL V2I16:$Rs, I32:$Rt),
  1531:          (LoReg (S2_asl_i_vh (ToAext64 $Rs), I32:$Rt))>;
  1532: def: Pat<(HexagonVLSR V2I16:$Rs, I32:$Rt),
  1533:          (LoReg (S2_lsr_i_vh (ToAext64 $Rs), I32:$Rt))>;
  1534: 
  1535: 
  1536: // --(9) Arithmetic/bitwise ----------------------------------------------
  1537: //
  1538: 
  1539: def: Pat<(abs  I32:$Rs), (A2_abs   I32:$Rs)>;
  1540: def: Pat<(abs  I64:$Rs), (A2_absp  I64:$Rs)>;
  1541: def: Pat<(not  I32:$Rs), (A2_subri -1, I32:$Rs)>;
  1542: def: Pat<(not  I64:$Rs), (A2_notp  I64:$Rs)>;
  1543: def: Pat<(ineg I64:$Rs), (A2_negp  I64:$Rs)>;
  1544: 
  1545: def: Pat<(fabs F32:$Rs), (S2_clrbit_i    F32:$Rs, 31)>;
  1546: def: Pat<(fneg F32:$Rs), (S2_togglebit_i F32:$Rs, 31)>;
  1547: 
  1548: def: Pat<(fabs F64:$Rs),
  1549:          (Combinew (S2_clrbit_i (HiReg $Rs), 31),
  1550:                    (i32 (LoReg $Rs)))>;
  1551: def: Pat<(fneg F64:$Rs),
  1552:          (Combinew (S2_togglebit_i (HiReg $Rs), 31),
  1553:                    (i32 (LoReg $Rs)))>;
  1554: 
  1555: def: Pat<(add I32:$Rs, anyimm:$s16),   (A2_addi   I32:$Rs,  imm:$s16)>;
  1556: def: Pat<(or  I32:$Rs, anyimm:$s10),   (A2_orir   I32:$Rs,  imm:$s10)>;
  1557: def: Pat<(and I32:$Rs, anyimm:$s10),   (A2_andir  I32:$Rs,  imm:$s10)>;
  1558: def: Pat<(sub anyimm:$s10, I32:$Rs),   (A2_subri  imm:$s10, I32:$Rs)>;
  1559: 
  1560: class OpR_RR_pat_sat<InstHexagon MI, SDNode Op, ValueType ResType,
  1561:                      PatFrag RxPred>
  1562:   : Pat<(ResType (Op RxPred:$Rs, RxPred:$Rt)),
  1563:         (MI RxPred:$Rs, RxPred:$Rt)>;
  1564: 
  1565: def: OpR_RR_pat_sat<A2_addsat,  saddsat, i32, I32>;
  1566: def: OpR_RR_pat_sat<A2_addpsat, saddsat, i64, I64>;
  1567: 
  1568: def: OpR_RR_pat<A2_add,       Add,        i32,   I32>;
  1569: def: OpR_RR_pat<A2_sub,       Sub,        i32,   I32>;
  1570: def: OpR_RR_pat<A2_and,       And,        i32,   I32>;
  1571: def: OpR_RR_pat<A2_or,        Or,         i32,   I32>;
  1572: def: OpR_RR_pat<A2_xor,       Xor,        i32,   I32>;
  1573: def: OpR_RR_pat<A2_addp,      Add,        i64,   I64>;
  1574: def: OpR_RR_pat<A2_subp,      Sub,        i64,   I64>;
  1575: def: OpR_RR_pat<A2_andp,      And,        i64,   I64>;
  1576: def: OpR_RR_pat<A2_orp,       Or,         i64,   I64>;
  1577: def: OpR_RR_pat<A2_xorp,      Xor,        i64,   I64>;
  1578: def: OpR_RR_pat<A4_andnp,     Not2<And>,  i64,   I64>;
  1579: def: OpR_RR_pat<A4_ornp,      Not2<Or>,   i64,   I64>;
  1580: 
  1581: def: OpR_RR_pat<A2_svaddh,    Add,        v2i16, V2I16>;
  1582: def: OpR_RR_pat<A2_svsubh,    Sub,        v2i16, V2I16>;
  1583: 
  1584: def: OpR_RR_pat<A2_vaddub,    Add,        v8i8,  V8I8>;
  1585: def: OpR_RR_pat<A2_vaddh,     Add,        v4i16, V4I16>;
  1586: def: OpR_RR_pat<A2_vaddw,     Add,        v2i32, V2I32>;
  1587: def: OpR_RR_pat<A2_vsubub,    Sub,        v8i8,  V8I8>;
  1588: def: OpR_RR_pat<A2_vsubh,     Sub,        v4i16, V4I16>;
  1589: def: OpR_RR_pat<A2_vsubw,     Sub,        v2i32, V2I32>;
  1590: 
  1591: def: OpR_RR_pat<A2_and,       And,        v4i8,  V4I8>;
  1592: def: OpR_RR_pat<A2_xor,       Xor,        v4i8,  V4I8>;
  1593: def: OpR_RR_pat<A2_or,        Or,         v4i8,  V4I8>;
  1594: def: OpR_RR_pat<A2_and,       And,        v2i16, V2I16>;
  1595: def: OpR_RR_pat<A2_xor,       Xor,        v2i16, V2I16>;
  1596: def: OpR_RR_pat<A2_or,        Or,         v2i16, V2I16>;
  1597: def: OpR_RR_pat<A2_andp,      And,        v8i8,  V8I8>;
  1598: def: OpR_RR_pat<A2_orp,       Or,         v8i8,  V8I8>;
  1599: def: OpR_RR_pat<A2_xorp,      Xor,        v8i8,  V8I8>;
  1600: def: OpR_RR_pat<A2_andp,      And,        v4i16, V4I16>;
```
- EN: It declares types such as OpshIRI_pat, OpR_RR_pat_sat, which carry the state or API of this component. It defines declarative TableGen records like OpshIRI_pat, SDTHexagonVShift, HexagonVASL, HexagonVASR, HexagonVLSR, ... (8 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVASL, HexagonISD, HexagonVASR, HexagonVLSR, ... (6 total), showing how the code connects to sibling backend components.
- CN: 这里声明了 OpshIRI_pat, OpR_RR_pat_sat 等类型，用来承载该组件的状态或接口。 这里定义了 OpshIRI_pat, SDTHexagonVShift, HexagonVASL, HexagonVASR, HexagonVLSR, ... (8 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVASL, HexagonISD, HexagonVASR, HexagonVLSR, ... (6 total)，说明了它与同级后端组件的连接关系。

### Lines 1601-1800 / 第 1601-1800 行

```tablegen
  1601: def: OpR_RR_pat<A2_orp,       Or,         v4i16, V4I16>;
  1602: def: OpR_RR_pat<A2_xorp,      Xor,        v4i16, V4I16>;
  1603: def: OpR_RR_pat<A2_andp,      And,        v2i32, V2I32>;
  1604: def: OpR_RR_pat<A2_orp,       Or,         v2i32, V2I32>;
  1605: def: OpR_RR_pat<A2_xorp,      Xor,        v2i32, V2I32>;
  1606: 
  1607: def: OpR_RR_pat<M2_mpyi,      Mul,        i32,   I32>;
  1608: def: OpR_RR_pat<M2_mpy_up,    pf2<mulhs>, i32,   I32>;
  1609: def: OpR_RR_pat<M2_mpyu_up,   pf2<mulhu>, i32,   I32>;
  1610: def: OpR_RI_pat<M2_mpysip,    Mul,        i32,   I32, u32_0ImmPred>;
  1611: def: OpR_RI_pat<M2_mpysmi,    Mul,        i32,   I32, s32_0ImmPred>;
  1612: 
  1613: // Arithmetic on predicates.
  1614: def: OpR_RR_pat<C2_xor,       Add,        i1,    I1>;
  1615: def: OpR_RR_pat<C2_xor,       Add,        v2i1,  V2I1>;
  1616: def: OpR_RR_pat<C2_xor,       Add,        v4i1,  V4I1>;
  1617: def: OpR_RR_pat<C2_xor,       Add,        v8i1,  V8I1>;
  1618: def: OpR_RR_pat<C2_xor,       Sub,        i1,    I1>;
  1619: def: OpR_RR_pat<C2_xor,       Sub,        v2i1,  V2I1>;
  1620: def: OpR_RR_pat<C2_xor,       Sub,        v4i1,  V4I1>;
  1621: def: OpR_RR_pat<C2_xor,       Sub,        v8i1,  V8I1>;
  1622: def: OpR_RR_pat<C2_and,       Mul,        i1,    I1>;
  1623: def: OpR_RR_pat<C2_and,       Mul,        v2i1,  V2I1>;
  1624: def: OpR_RR_pat<C2_and,       Mul,        v4i1,  V4I1>;
  1625: def: OpR_RR_pat<C2_and,       Mul,        v8i1,  V8I1>;
  1626: 
  1627: def: OpR_RR_pat<F2_sfadd,     pf2<fadd>,    f32, F32>;
  1628: def: OpR_RR_pat<F2_sfsub,     pf2<fsub>,    f32, F32>;
  1629: def: OpR_RR_pat<F2_sfmpy,     pf2<fmul>,    f32, F32>;
  1630: def: OpR_RR_pat<F2_sfmin,     pf2<fminimumnum>, f32, F32>;
  1631: def: OpR_RR_pat<F2_sfmax,     pf2<fmaximumnum>, f32, F32>;
  1632: 
  1633: let Predicates = [HasV66] in {
  1634:   def: OpR_RR_pat<F2_dfadd,     pf2<fadd>,    f64, F64>;
  1635:   def: OpR_RR_pat<F2_dfsub,     pf2<fsub>,    f64, F64>;
  1636: }
  1637: 
  1638: def DfMpy: OutPatFrag<(ops node:$Rs, node:$Rt),
  1639:   (F2_dfmpyhh
  1640:     (F2_dfmpylh
  1641:       (F2_dfmpylh
  1642:         (F2_dfmpyll $Rs, $Rt),
  1643:       $Rs, $Rt),
  1644:     $Rt, $Rs),
  1645:   $Rs, $Rt)>;
  1646: 
  1647: def fmul_afn : PatFrag<(ops node:$a, node:$b), (fmul node:$a, node:$b), [{
  1648:   return N->getFlags().hasApproximateFuncs();
  1649: }]>;
  1650: let Predicates = [HasV67], AddedComplexity = 50 in {
  1651:   def : Pat<(fmul_afn F64:$Rs, F64:$Rt), (DfMpy $Rs, $Rt)>;
  1652: }
  1653: let Predicates = [HasV67] in {
  1654:   def: OpR_RR_pat<F2_dfmin,     pf2<fminimumnum>, f64, F64>;
  1655:   def: OpR_RR_pat<F2_dfmax,     pf2<fmaximumnum>, f64, F64>;
  1656: 
  1657:   def: Pat<(fmul F64:$Rs, F64:$Rt), (DfMpy (F2_dfmpyfix $Rs, $Rt),
  1658:                                            (F2_dfmpyfix $Rt, $Rs))>;
  1659: }
  1660: 
  1661: // In expressions like a0*b0 + a1*b1 + ..., prefer to generate multiply-add,
  1662: // over add-add with individual multiplies as inputs.
  1663: let AddedComplexity = 10 in {
  1664:   def: AccRRI_pat<M2_macsip,    Add, Su<Mul>, I32, u32_0ImmPred>;
  1665:   def: AccRRI_pat<M2_macsin,    Sub, Su<Mul>, I32, u32_0ImmPred>;
  1666:   def: AccRRR_pat<M2_maci,      Add, Su<Mul>, I32, I32, I32>;
  1667:   let Predicates = [HasV66] in
  1668:   def: AccRRR_pat<M2_mnaci,     Sub, Su<Mul>, I32, I32, I32>;
  1669: }
  1670: 
  1671: def: AccRRI_pat<M2_naccii,    Sub, Su<Add>, I32, s32_0ImmPred>;
  1672: def: AccRRI_pat<M2_accii,     Add, Su<Add>, I32, s32_0ImmPred>;
  1673: def: AccRRR_pat<M2_acci,      Add, Su<Add>, I32, I32, I32>;
  1674: 
  1675: // Mulh for vectors
  1676: //
  1677: def: Pat<(v2i32 (mulhu V2I32:$Rss, V2I32:$Rtt)),
  1678:          (Combinew (M2_mpyu_up (HiReg $Rss), (HiReg $Rtt)),
  1679:                    (M2_mpyu_up (LoReg $Rss), (LoReg $Rtt)))>;
  1680: 
  1681: def: Pat<(v2i32 (mulhs V2I32:$Rss, V2I32:$Rtt)),
  1682:          (Combinew (M2_mpy_up (HiReg $Rss), (HiReg $Rtt)),
  1683:                    (M2_mpy_up (LoReg $Rss), (LoReg $Rtt)))>;
  1684: 
  1685: def Mulhub4:
  1686:   OutPatFrag<(ops node:$Rs, node:$Rt), (S2_vtrunohb (M5_vmpybuu $Rs, $Rt))>;
  1687: def Mulhub8:
  1688:   OutPatFrag<(ops node:$Rss, node:$Rtt),
  1689:              (Combinew (Mulhub4 (HiReg $Rss), (HiReg $Rtt)),
  1690:                        (Mulhub4 (LoReg $Rss), (LoReg $Rtt)))>;
  1691: 
  1692: // (mux (x >= 0), 0, y)
  1693: def Negbytes8:
  1694:   OutPatFrag<(ops node:$Rss, node:$Rtt),
  1695:              (C2_vmux (A4_vcmpbgti $Rss, -1), (A2_tfrpi 0), $Rtt)>;
  1696: 
  1697: def: Pat<(v4i8 (mulhu  V4I8:$Rs,  V4I8:$Rt)), (Mulhub4  $Rs,  $Rt)>;
  1698: def: Pat<(v8i8 (mulhu V8I8:$Rss, V8I8:$Rtt)), (Mulhub8 $Rss, $Rtt)>;
  1699: 
  1700: // (Mulhs x, y) = (Mulhu x, y) - (x < 0 ? y : 0) - (y < 0 ? x : 0)
  1701: def Mulhsb8:
  1702:   OutPatFrag<(ops node:$Rss, node:$Rtt),
  1703:              (A2_vsubub (Mulhub8 $Rss, $Rtt),
  1704:                         (A2_vaddub (Negbytes8 $Rss, $Rtt),
  1705:                                    (Negbytes8 $Rtt, $Rss)))>;
  1706: 
  1707: def: Pat<(v4i8 (mulhs V4I8:$Rs, V4I8:$Rt)),
  1708:          (LoReg (Mulhsb8 (v8i8 (ToAext64 $Rs)), (v8i8 (ToAext64 $Rt))))>;
  1709: def: Pat<(v8i8 (mulhs V8I8:$Rss, V8I8:$Rtt)), (Mulhsb8 $Rss, $Rtt)>;
  1710: 
  1711: // v2i16 *s v2i16 -> v2i32
  1712: def Muli16:
  1713:   OutPatFrag<(ops node:$Rs, node:$Rt), (M2_vmpy2s_s0 $Rs, $Rt)>;
  1714: 
  1715: def Mulhsh2:
  1716:   OutPatFrag<(ops node:$Rs, node:$Rt),
  1717:              (A2_combine_hh (HiReg (Muli16 $Rs, $Rt)),
  1718:                             (LoReg (Muli16 $Rs, $Rt)))>;
  1719: def Mulhsh4:
  1720:   OutPatFrag<(ops node:$Rss, node:$Rtt),
  1721:              (Combinew (Mulhsh2 (HiReg $Rss), (HiReg $Rtt)),
  1722:                        (Mulhsh2 (LoReg $Rss), (LoReg $Rtt)))>;
  1723: 
  1724: def: Pat<(v2i16 (mulhs  V2I16:$Rs,  V2I16:$Rt)), (Mulhsh2  $Rs,  $Rt)>;
  1725: def: Pat<(v4i16 (mulhs V4I16:$Rss, V4I16:$Rtt)), (Mulhsh4 $Rss, $Rtt)>;
  1726: 
  1727: def: Pat<(v2i16 (mulhu V2I16:$Rs, V2I16:$Rt)),
  1728:   (A2_svaddh
  1729:      (Mulhsh2 $Rs, $Rt),
  1730:      (A2_svaddh (LoReg (A2_andp (Combinew $Rt, $Rs),
  1731:                                 (S2_asr_i_vh (Combinew $Rs, $Rt), 15))),
  1732:                 (HiReg (A2_andp (Combinew $Rt, $Rs),
  1733:                                 (S2_asr_i_vh (Combinew $Rs, $Rt), 15)))))>;
  1734: 
  1735: def: Pat<(v4i16 (mulhu V4I16:$Rss, V4I16:$Rtt)),
  1736:          (A2_vaddh
  1737:            (Mulhsh4 $Rss, $Rtt),
  1738:            (A2_vaddh (A2_andp V4I16:$Rss, (S2_asr_i_vh $Rtt, 15)),
  1739:                      (A2_andp V4I16:$Rtt, (S2_asr_i_vh $Rss, 15))))>;
  1740: 
  1741: 
  1742: def: Pat<(ineg (mul I32:$Rs, u8_0ImmPred:$u8)),
  1743:          (M2_mpysin IntRegs:$Rs, imm:$u8)>;
  1744: 
  1745: def n8_0ImmPred: PatLeaf<(i32 imm), [{
  1746:   int64_t V = N->getSExtValue();
  1747:   return -255 <= V && V <= 0;
  1748: }]>;
  1749: 
  1750: def mulnsw : PatFrag<(ops node:$lhs, node:$rhs),
  1751:                      (mul node:$lhs, node:$rhs), [{
  1752:   return N->getFlags().hasNoSignedWrap();
  1753: }]>;
  1754: 
  1755: // Change the sign of the immediate for Rd=-mpyi(Rs,#u8)
  1756: def: Pat<(mul I32:$Rs, n8_0ImmPred:$n8),
  1757:          (M2_mpysin I32:$Rs, (NegImm8 imm:$n8))>;
  1758: 
  1759: def: Pat<(v2i32 (mulnsw (sext V2I16:$Rs), (sext V2I16:$Rt))),
  1760:          (M2_vmpy2s_s0 V2I16:$Rs, V2I16:$Rt)>;
  1761: 
  1762: def: Pat<(add Sext64:$Rs, I64:$Rt),
  1763:          (A2_addsp (LoReg Sext64:$Rs), I64:$Rt)>;
  1764: 
  1765: def: AccRRR_pat<M4_and_and,   And, Su_ni1<And>,  I32,  I32,  I32>;
  1766: def: AccRRR_pat<M4_and_or,    And, Su_ni1<Or>,   I32,  I32,  I32>;
  1767: def: AccRRR_pat<M4_and_xor,   And, Su<Xor>,      I32,  I32,  I32>;
  1768: def: AccRRR_pat<M4_or_and,    Or,  Su_ni1<And>,  I32,  I32,  I32>;
  1769: def: AccRRR_pat<M4_or_or,     Or,  Su_ni1<Or>,   I32,  I32,  I32>;
  1770: def: AccRRR_pat<M4_or_xor,    Or,  Su<Xor>,      I32,  I32,  I32>;
  1771: def: AccRRR_pat<M4_xor_and,   Xor, Su_ni1<And>,  I32,  I32,  I32>;
  1772: def: AccRRR_pat<M4_xor_or,    Xor, Su_ni1<Or>,   I32,  I32,  I32>;
  1773: def: AccRRR_pat<M2_xor_xacc,  Xor, Su<Xor>,      I32,  I32,  I32>;
  1774: def: AccRRR_pat<M4_xor_xacc,  Xor, Su<Xor>,      I64,  I64,  I64>;
  1775: 
  1776: // For dags like (or (and (not _), _), (shl _, _)) where the "or" with
  1777: // one argument matches the patterns below, and with the other argument
  1778: // matches S2_asl_r_r_or, etc, prefer the patterns below.
  1779: let AddedComplexity = 110 in {  // greater than S2_asl_r_r_and/or/xor.
  1780:   def: AccRRR_pat<M4_and_andn,  And, Su<Not2<And>>, I32,  I32,  I32>;
  1781:   def: AccRRR_pat<M4_or_andn,   Or,  Su<Not2<And>>, I32,  I32,  I32>;
  1782:   def: AccRRR_pat<M4_xor_andn,  Xor, Su<Not2<And>>, I32,  I32,  I32>;
  1783: }
  1784: 
  1785: // S4_addaddi and S4_subaddi don't have tied operands, so give them
  1786: // a bit of preference.
  1787: let AddedComplexity = 30, Predicates = [UseCompound] in {
  1788:   def: Pat<(add I32:$Rs, (Su<Add> I32:$Ru, anyimm:$s6)),
  1789:            (S4_addaddi IntRegs:$Rs, IntRegs:$Ru, imm:$s6)>;
  1790:   def: Pat<(add anyimm:$s6, (Su<Add> I32:$Rs, I32:$Ru)),
  1791:            (S4_addaddi IntRegs:$Rs, IntRegs:$Ru, imm:$s6)>;
  1792:   def: Pat<(add I32:$Rs, (Su<Sub> anyimm:$s6, I32:$Ru)),
  1793:            (S4_subaddi IntRegs:$Rs, imm:$s6, IntRegs:$Ru)>;
  1794:   def: Pat<(sub (Su<Add> I32:$Rs, anyimm:$s6), I32:$Ru),
  1795:            (S4_subaddi IntRegs:$Rs, imm:$s6, IntRegs:$Ru)>;
  1796:   def: Pat<(add (Su<Sub> I32:$Rs, I32:$Ru), anyimm:$s6),
  1797:            (S4_subaddi IntRegs:$Rs, imm:$s6, IntRegs:$Ru)>;
  1798: }
  1799: 
  1800: let Predicates = [UseCompound] in
```
- EN: It defines declarative TableGen records like DfMpy, fmul_afn, Mulhub4, Mulhub8, Negbytes8, ... (11 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getFlags, getSExtValue, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里定义了 DfMpy, fmul_afn, Mulhub4, Mulhub8, Negbytes8, ... (11 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getFlags, getSExtValue 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1801-2000 / 第 1801-2000 行

```tablegen
  1801: def: Pat<(or I32:$Ru, (Su<And> I32:$Rx, anyimm:$s10)),
  1802:          (S4_or_andix IntRegs:$Ru, IntRegs:$Rx, imm:$s10)>;
  1803: 
  1804: def: Pat<(or I32:$Rx, (Su<And> I32:$Rs, anyimm:$s10)),
  1805:          (S4_or_andi IntRegs:$Rx, IntRegs:$Rs, imm:$s10)>;
  1806: def: Pat<(or I32:$Rx, (Su<Or> I32:$Rs, anyimm:$s10)),
  1807:          (S4_or_ori IntRegs:$Rx, IntRegs:$Rs, imm:$s10)>;
  1808: 
  1809: 
  1810: def: Pat<(i32 (trunc (sra (Su<Mul> Sext64:$Rs, Sext64:$Rt), (i32 32)))),
  1811:          (M2_mpy_up (LoReg Sext64:$Rs), (LoReg Sext64:$Rt))>;
  1812: def: Pat<(i32 (trunc (srl (Su<Mul> Sext64:$Rs, Sext64:$Rt), (i32 32)))),
  1813:          (M2_mpy_up (LoReg Sext64:$Rs), (LoReg Sext64:$Rt))>;
  1814: 
  1815: def: Pat<(mul (Zext64 I32:$Rs), (Zext64 I32:$Rt)),
  1816:          (M2_dpmpyuu_s0 I32:$Rs, I32:$Rt)>;
  1817: def: Pat<(mul (Aext64 I32:$Rs), (Aext64 I32:$Rt)),
  1818:          (M2_dpmpyuu_s0 I32:$Rs, I32:$Rt)>;
  1819: def: Pat<(mul Sext64:$Rs, Sext64:$Rt),
  1820:          (M2_dpmpyss_s0 (LoReg Sext64:$Rs), (LoReg Sext64:$Rt))>;
  1821: 
  1822: def: Pat<(add I64:$Rx, (Su<Mul> Sext64:$Rs, Sext64:$Rt)),
  1823:          (M2_dpmpyss_acc_s0 I64:$Rx, (LoReg Sext64:$Rs), (LoReg Sext64:$Rt))>;
  1824: def: Pat<(sub I64:$Rx, (Su<Mul> Sext64:$Rs, Sext64:$Rt)),
  1825:          (M2_dpmpyss_nac_s0 I64:$Rx, (LoReg Sext64:$Rs), (LoReg Sext64:$Rt))>;
  1826: def: Pat<(add I64:$Rx, (Su<Mul> (Aext64 I32:$Rs), (Aext64 I32:$Rt))),
  1827:          (M2_dpmpyuu_acc_s0 I64:$Rx, I32:$Rs, I32:$Rt)>;
  1828: def: Pat<(add I64:$Rx, (Su<Mul> (Zext64 I32:$Rs), (Zext64 I32:$Rt))),
  1829:          (M2_dpmpyuu_acc_s0 I64:$Rx, I32:$Rs, I32:$Rt)>;
  1830: def: Pat<(sub I64:$Rx, (Su<Mul> (Aext64 I32:$Rs), (Aext64 I32:$Rt))),
  1831:          (M2_dpmpyuu_nac_s0 I64:$Rx, I32:$Rs, I32:$Rt)>;
  1832: def: Pat<(sub I64:$Rx, (Su<Mul> (Zext64 I32:$Rs), (Zext64 I32:$Rt))),
  1833:          (M2_dpmpyuu_nac_s0 I64:$Rx, I32:$Rs, I32:$Rt)>;
  1834: 
  1835: // Add halfword.
  1836: def: Pat<(sext_inreg (add I32:$Rt, I32:$Rs), i16),
  1837:          (A2_addh_l16_ll I32:$Rt, I32:$Rs)>;
  1838: def: Pat<(sra (add (shl I32:$Rt, (i32 16)), I32:$Rs), (i32 16)),
  1839:          (A2_addh_l16_hl I32:$Rt, I32:$Rs)>;
  1840: def: Pat<(shl (add I32:$Rt, I32:$Rs), (i32 16)),
  1841:          (A2_addh_h16_ll I32:$Rt, I32:$Rs)>;
  1842: 
  1843: // Subtract halfword.
  1844: def: Pat<(sext_inreg (sub I32:$Rt, I32:$Rs), i16),
  1845:          (A2_subh_l16_ll I32:$Rt, I32:$Rs)>;
  1846: def: Pat<(sra (add (shl I32:$Rt, (i32 16)), I32:$Rs), (i32 16)),
  1847:          (A2_addh_l16_hl I32:$Rt, I32:$Rs)>;
  1848: def: Pat<(shl (sub I32:$Rt, I32:$Rs), (i32 16)),
  1849:          (A2_subh_h16_ll I32:$Rt, I32:$Rs)>;
  1850: 
  1851: def: Pat<(mul I64:$Rss, I64:$Rtt),
  1852:          (Combinew
  1853:            (M2_maci (M2_maci (HiReg (M2_dpmpyuu_s0 (LoReg $Rss), (LoReg $Rtt))),
  1854:                              (LoReg $Rss),
  1855:                              (HiReg $Rtt)),
  1856:                     (LoReg $Rtt),
  1857:                     (HiReg $Rss)),
  1858:            (i32 (LoReg (M2_dpmpyuu_s0 (LoReg $Rss), (LoReg $Rtt)))))>;
  1859: 
  1860: def MulHU : OutPatFrag<(ops node:$Rss, node:$Rtt),
  1861:   (A2_addp
  1862:     (M2_dpmpyuu_acc_s0
  1863:       (S2_lsr_i_p
  1864:         (A2_addp
  1865:           (M2_dpmpyuu_acc_s0
  1866:             (S2_lsr_i_p (M2_dpmpyuu_s0 (LoReg $Rss), (LoReg $Rtt)), 32),
  1867:             (HiReg $Rss),
  1868:             (LoReg $Rtt)),
  1869:           (A4_combineir 0, (LoReg (M2_dpmpyuu_s0 (LoReg $Rss), (HiReg $Rtt))))),
  1870:         32),
  1871:       (HiReg $Rss),
  1872:       (HiReg $Rtt)),
  1873:     (S2_lsr_i_p (M2_dpmpyuu_s0 (LoReg $Rss), (HiReg $Rtt)), 32))>;
  1874: 
  1875: // Multiply 64-bit unsigned and use upper result.
  1876: def : Pat <(mulhu I64:$Rss, I64:$Rtt), (MulHU $Rss, $Rtt)>;
  1877: 
  1878: // Multiply 64-bit signed and use upper result.
  1879: //
  1880: // For two signed 64-bit integers A and B, let A' and B' denote A and B
  1881: // with the sign bit cleared. Then A = -2^63*s(A) + A', where s(A) is the
  1882: // sign bit of A (and identically for B). With this notation, the signed
  1883: // product A*B can be written as:
  1884: //   AB = (-2^63 s(A) + A') * (-2^63 s(B) + B')
  1885: //      = 2^126 s(A)s(B) - 2^63 [s(A)B'+s(B)A'] + A'B'
  1886: //      = 2^126 s(A)s(B) + 2^63 [s(A)B'+s(B)A'] + A'B' - 2*2^63 [s(A)B'+s(B)A']
  1887: //      = (unsigned product AB) - 2^64 [s(A)B'+s(B)A']
  1888: 
  1889: // Clear the sign bit in a 64-bit register.
  1890: def ClearSign : OutPatFrag<(ops node:$Rss),
  1891:   (Combinew (S2_clrbit_i (HiReg $Rss), 31), (i32 (LoReg $Rss)))>;
  1892: 
  1893: def : Pat <(mulhs I64:$Rss, I64:$Rtt),
  1894:   (A2_subp
  1895:     (MulHU $Rss, $Rtt),
  1896:     (A2_addp
  1897:       (A2_andp (S2_asr_i_p $Rss, 63), (ClearSign $Rtt)),
  1898:       (A2_andp (S2_asr_i_p $Rtt, 63), (ClearSign $Rss))))>;
  1899: 
  1900: // Prefer these instructions over M2_macsip/M2_macsin: the macsi* instructions
  1901: // will put the immediate addend into a register, while these instructions will
  1902: // use it directly. Such a construct does not appear in the middle of a gep,
  1903: // where M2_macsip would be preferable.
  1904: let AddedComplexity = 20, Predicates = [UseCompound] in {
  1905:   def: Pat<(add (Su<Mul> I32:$Rs, u6_0ImmPred:$U6), anyimm:$u6),
  1906:            (M4_mpyri_addi imm:$u6, IntRegs:$Rs, imm:$U6)>;
  1907:   def: Pat<(add (Su<Mul> I32:$Rs, I32:$Rt), anyimm:$u6),
  1908:            (M4_mpyrr_addi imm:$u6, IntRegs:$Rs, IntRegs:$Rt)>;
  1909: }
  1910: 
  1911: // Keep these instructions less preferable to M2_macsip/M2_macsin.
  1912: let Predicates = [UseCompound] in {
  1913:   def: Pat<(add I32:$Ru, (Su<Mul> I32:$Rs, u6_2ImmPred:$u6_2)),
  1914:            (M4_mpyri_addr_u2 IntRegs:$Ru, imm:$u6_2, IntRegs:$Rs)>;
  1915:   def: Pat<(add I32:$Ru, (Su<Mul> I32:$Rs, anyimm:$u6)),
  1916:            (M4_mpyri_addr IntRegs:$Ru, IntRegs:$Rs, imm:$u6)>;
  1917:   def: Pat<(add I32:$Ru, (Su<Mul> I32:$Ry, I32:$Rs)),
  1918:            (M4_mpyrr_addr IntRegs:$Ru, IntRegs:$Ry, IntRegs:$Rs)>;
  1919: }
  1920: 
  1921: def: Pat<(fma F32:$Rs, F32:$Rt, F32:$Rx),
  1922:          (F2_sffma F32:$Rx, F32:$Rs, F32:$Rt)>;
  1923: def: Pat<(fma (fneg F32:$Rs), F32:$Rt, F32:$Rx),
  1924:          (F2_sffms F32:$Rx, F32:$Rs, F32:$Rt)>;
  1925: 
  1926: def: Pat<(mul V2I32:$Rs, V2I32:$Rt),
  1927:          (PS_vmulw V2I32:$Rs, V2I32:$Rt)>;
  1928: def: Pat<(add V2I32:$Rx, (mul V2I32:$Rs, V2I32:$Rt)),
  1929:          (PS_vmulw_acc V2I32:$Rx, V2I32:$Rs, V2I32:$Rt)>;
  1930: 
  1931: // Add/subtract two v4i8: Hexagon does not have an insn for this one, so
  1932: // we use the double add v8i8, and use only the low part of the result.
  1933: def: Pat<(add V4I8:$Rs, V4I8:$Rt),
  1934:          (LoReg (A2_vaddub (ToAext64 $Rs), (ToAext64 $Rt)))>;
  1935: def: Pat<(sub V4I8:$Rs, V4I8:$Rt),
  1936:          (LoReg (A2_vsubub (ToAext64 $Rs), (ToAext64 $Rt)))>;
  1937: 
  1938: // Use M2_vmpy2s_s0 for half-word vector multiply. It multiplies two
  1939: // half-words, and saturates the result to a 32-bit value, except the
  1940: // saturation never happens (it can only occur with scaling).
  1941: def: Pat<(v2i16 (mul V2I16:$Rs, V2I16:$Rt)),
  1942:          (LoReg (S2_vtrunewh (IMPLICIT_DEF),
  1943:                              (M2_vmpy2s_s0 V2I16:$Rs, V2I16:$Rt)))>;
  1944: def: Pat<(v4i16 (mul V4I16:$Rs, V4I16:$Rt)),
  1945:          (S2_vtrunewh (M2_vmpy2s_s0 (HiReg $Rs), (HiReg $Rt)),
  1946:                       (M2_vmpy2s_s0 (LoReg $Rs), (LoReg $Rt)))>;
  1947: 
  1948: // Multiplies two v4i8 vectors.
  1949: def: Pat<(v4i8 (mul V4I8:$Rs, V4I8:$Rt)),
  1950:          (S2_vtrunehb (M5_vmpybuu V4I8:$Rs, V4I8:$Rt))>;
  1951: 
  1952: // Multiplies two v8i8 vectors.
  1953: def: Pat<(v8i8 (mul V8I8:$Rs, V8I8:$Rt)),
  1954:          (Combinew (S2_vtrunehb (M5_vmpybuu (HiReg $Rs), (HiReg $Rt))),
  1955:                    (S2_vtrunehb (M5_vmpybuu (LoReg $Rs), (LoReg $Rt))))>;
  1956: 
  1957: 
  1958: // --(10) Bit ------------------------------------------------------------
  1959: //
  1960: 
  1961: // Count leading zeros.
  1962: def: Pat<(i32 (ctlz I32:$Rs)),                (S2_cl0 I32:$Rs)>;
  1963: def: Pat<(i32 (trunc (ctlz I64:$Rss))),       (S2_cl0p I64:$Rss)>;
  1964: 
  1965: // Count trailing zeros.
  1966: def: Pat<(i32 (cttz I32:$Rs)),                (S2_ct0 I32:$Rs)>;
  1967: def: Pat<(i32 (trunc (cttz I64:$Rss))),       (S2_ct0p I64:$Rss)>;
  1968: 
  1969: // Count leading ones.
  1970: def: Pat<(i32 (ctlz (not I32:$Rs))),          (S2_cl1 I32:$Rs)>;
  1971: def: Pat<(i32 (trunc (ctlz (not I64:$Rss)))), (S2_cl1p I64:$Rss)>;
  1972: 
  1973: // Count trailing ones.
  1974: def: Pat<(i32 (cttz (not I32:$Rs))),           (S2_ct1 I32:$Rs)>;
  1975: def: Pat<(i32 (trunc (cttz (not I64:$Rss)))), (S2_ct1p I64:$Rss)>;
  1976: 
  1977: // Define leading/trailing patterns that require zero-extensions to 64 bits.
  1978: def: Pat<(i64 (ctlz I64:$Rss)),               (ToZext64 (S2_cl0p I64:$Rss))>;
  1979: def: Pat<(i64 (cttz I64:$Rss)),               (ToZext64 (S2_ct0p I64:$Rss))>;
  1980: def: Pat<(i64 (ctlz (not I64:$Rss))),         (ToZext64 (S2_cl1p I64:$Rss))>;
  1981: def: Pat<(i64 (cttz (not I64:$Rss))),         (ToZext64 (S2_ct1p I64:$Rss))>;
  1982: 
  1983: def: Pat<(i64 (ctpop I64:$Rss)),  (ToZext64 (S5_popcountp I64:$Rss))>;
  1984: def: Pat<(i32 (ctpop I32:$Rs)),   (S5_popcountp (A4_combineir 0, I32:$Rs))>;
  1985: 
  1986: def: Pat<(bitreverse I32:$Rs),    (S2_brev I32:$Rs)>;
  1987: def: Pat<(bitreverse I64:$Rss),   (S2_brevp I64:$Rss)>;
  1988: 
  1989: def: Pat<(bitreverse V4I8:$Rs),   (A2_swiz (S2_brev $Rs))>;
  1990: def: Pat<(bitreverse V8I8:$Rs),   (Combinew (A2_swiz (LoReg (S2_brevp $Rs))),
  1991:                                             (A2_swiz (HiReg (S2_brevp $Rs))))>;
  1992: def: Pat<(bitreverse V2I16:$Rs),  (A2_combine_lh (S2_brev $Rs),
  1993:                                                  (S2_brev $Rs))>;
  1994: def: Pat<(bitreverse V4I16:$Rs),
  1995:          (Combinew (A2_combine_lh (LoReg (S2_brevp $Rs)),
  1996:                                   (LoReg (S2_brevp $Rs))),
  1997:                    (A2_combine_lh (HiReg (S2_brevp $Rs)),
  1998:                                   (HiReg (S2_brevp $Rs))))>;
  1999: def: Pat<(bitreverse V2I32:$Rs),
  2000:          (Combinew (i32 (LoReg (S2_brevp $Rs))),
```
- EN: It defines declarative TableGen records like MulHU, ClearSign; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里定义了 MulHU, ClearSign 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 2001-2200 / 第 2001-2200 行

```tablegen
  2001:                    (i32 (HiReg (S2_brevp $Rs))))>;
  2002: 
  2003: let AddedComplexity = 20 in { // Complexity greater than and/or/xor
  2004:   def: Pat<(and I32:$Rs, IsNPow2_32:$V),
  2005:            (S2_clrbit_i IntRegs:$Rs, (LogN2_32 $V))>;
  2006:   def: Pat<(or I32:$Rs, IsPow2_32:$V),
  2007:            (S2_setbit_i IntRegs:$Rs, (Log2_32 $V))>;
  2008:   def: Pat<(xor I32:$Rs, IsPow2_32:$V),
  2009:            (S2_togglebit_i IntRegs:$Rs, (Log2_32 $V))>;
  2010: 
  2011:   def: Pat<(and I32:$Rs, (not (shl 1, I32:$Rt))),
  2012:            (S2_clrbit_r IntRegs:$Rs, IntRegs:$Rt)>;
  2013:   def: Pat<(or I32:$Rs, (shl 1, I32:$Rt)),
  2014:            (S2_setbit_r IntRegs:$Rs, IntRegs:$Rt)>;
  2015:   def: Pat<(xor I32:$Rs, (shl 1, I32:$Rt)),
  2016:            (S2_togglebit_r IntRegs:$Rs, IntRegs:$Rt)>;
  2017: }
  2018: 
  2019: // Clr/set/toggle bit for 64-bit values with immediate bit index.
  2020: let AddedComplexity = 20 in { // Complexity greater than and/or/xor
  2021:   def: Pat<(and I64:$Rss, IsNPow2_64L:$V),
  2022:            (Combinew (i32 (HiReg $Rss)),
  2023:                      (S2_clrbit_i (LoReg $Rss), (LogN2_64 $V)))>;
  2024:   def: Pat<(and I64:$Rss, IsNPow2_64H:$V),
  2025:            (Combinew (S2_clrbit_i (HiReg $Rss), (UDEC32 (i32 (LogN2_64 $V)))),
  2026:                      (i32 (LoReg $Rss)))>;
  2027: 
  2028:   def: Pat<(or I64:$Rss, IsPow2_64L:$V),
  2029:            (Combinew (i32 (HiReg $Rss)),
  2030:                      (S2_setbit_i (LoReg $Rss), (Log2_64 $V)))>;
  2031:   def: Pat<(or I64:$Rss, IsPow2_64H:$V),
  2032:            (Combinew (S2_setbit_i (HiReg $Rss), (UDEC32 (i32 (Log2_64 $V)))),
  2033:                      (i32 (LoReg $Rss)))>;
  2034: 
  2035:   def: Pat<(xor I64:$Rss, IsPow2_64L:$V),
  2036:            (Combinew (i32 (HiReg $Rss)),
  2037:                      (S2_togglebit_i (LoReg $Rss), (Log2_64 $V)))>;
  2038:   def: Pat<(xor I64:$Rss, IsPow2_64H:$V),
  2039:            (Combinew (S2_togglebit_i (HiReg $Rss), (UDEC32 (i32 (Log2_64 $V)))),
  2040:                      (i32 (LoReg $Rss)))>;
  2041: }
  2042: 
  2043: 
  2044: let AddedComplexity = 20 in { // Complexity greater than cmp reg-imm.
  2045:   def: Pat<(i1 (setne (and (shl 1, u5_0ImmPred:$u5), I32:$Rs), 0)),
  2046:            (S2_tstbit_i IntRegs:$Rs, imm:$u5)>;
  2047:   def: Pat<(i1 (setne (and (shl 1, I32:$Rt), I32:$Rs), 0)),
  2048:            (S2_tstbit_r IntRegs:$Rs, IntRegs:$Rt)>;
  2049:   def: Pat<(i1 (trunc I32:$Rs)),
  2050:            (S2_tstbit_i IntRegs:$Rs, 0)>;
  2051:   def: Pat<(i1 (trunc I64:$Rs)),
  2052:            (S2_tstbit_i (LoReg DoubleRegs:$Rs), 0)>;
  2053: }
  2054: 
  2055: def: Pat<(and (srl I32:$Rs, u5_0ImmPred:$u5), 1),
  2056:          (I1toI32 (S2_tstbit_i I32:$Rs, imm:$u5))>;
  2057: def: Pat<(and (srl I64:$Rss, IsULE<32,31>:$u6), 1),
  2058:          (ToZext64 (I1toI32 (S2_tstbit_i (LoReg $Rss), imm:$u6)))>;
  2059: def: Pat<(and (srl I64:$Rss, IsUGT<32,31>:$u6), 1),
  2060:          (ToZext64 (I1toI32 (S2_tstbit_i (HiReg $Rss), (UDEC32 $u6))))>;
  2061: 
  2062: def: Pat<(and (not (srl I32:$Rs, u5_0ImmPred:$u5)), 1),
  2063:          (I1toI32 (S4_ntstbit_i I32:$Rs, imm:$u5))>;
  2064: def: Pat<(and (not (srl I64:$Rss, IsULE<32,31>:$u6)), 1),
  2065:          (ToZext64 (I1toI32 (S4_ntstbit_i (LoReg $Rss), imm:$u6)))>;
  2066: def: Pat<(and (not (srl I64:$Rss, IsUGT<32,31>:$u6)), 1),
  2067:          (ToZext64 (I1toI32 (S4_ntstbit_i (HiReg $Rss), (UDEC32 $u6))))>;
  2068: 
  2069: let AddedComplexity = 20 in { // Complexity greater than compare reg-imm.
  2070:   def: Pat<(i1 (seteq (and I32:$Rs, u6_0ImmPred:$u6), 0)),
  2071:            (C2_bitsclri IntRegs:$Rs, imm:$u6)>;
  2072:   def: Pat<(i1 (seteq (and I32:$Rs, I32:$Rt), 0)),
  2073:            (C2_bitsclr IntRegs:$Rs, IntRegs:$Rt)>;
  2074: }
  2075: 
  2076: let AddedComplexity = 10 in   // Complexity greater than compare reg-reg.
  2077: def: Pat<(i1 (seteq (and I32:$Rs, I32:$Rt), IntRegs:$Rt)),
  2078:          (C2_bitsset IntRegs:$Rs, IntRegs:$Rt)>;
  2079: 
  2080: def SDTTestBit:
  2081:   SDTypeProfile<1, 2, [SDTCisVT<0, i1>, SDTCisVT<1, i32>, SDTCisVT<2, i32>]>;
  2082: def HexagonTSTBIT: SDNode<"HexagonISD::TSTBIT", SDTTestBit>;
  2083: 
  2084: def: Pat<(HexagonTSTBIT I32:$Rs, u5_0ImmPred:$u5),
  2085:          (S2_tstbit_i I32:$Rs, imm:$u5)>;
  2086: def: Pat<(HexagonTSTBIT I32:$Rs, I32:$Rt),
  2087:          (S2_tstbit_r I32:$Rs, I32:$Rt)>;
  2088: 
  2089: // Add extra complexity to prefer these instructions over bitsset/bitsclr.
  2090: // The reason is that tstbit/ntstbit can be folded into a compound instruction:
  2091: //   if ([!]tstbit(...)) jump ...
  2092: let AddedComplexity = 20 in {   // Complexity greater than cmp reg-imm.
  2093:   def: Pat<(i1 (seteq (and I32:$Rs, IsPow2_32:$u5), 0)),
  2094:            (S4_ntstbit_i I32:$Rs, (Log2_32 imm:$u5))>;
  2095:   def: Pat<(i1 (setne (and I32:$Rs, IsPow2_32:$u5), 0)),
  2096:            (S2_tstbit_i I32:$Rs, (Log2_32 imm:$u5))>;
  2097:   def: Pat<(i1 (seteq (and (shl 1, I32:$Rt), I32:$Rs), 0)),
  2098:            (S4_ntstbit_r I32:$Rs, I32:$Rt)>;
  2099:   def: Pat<(i1 (setne (and (shl 1, I32:$Rt), I32:$Rs), 0)),
  2100:            (S2_tstbit_r I32:$Rs, I32:$Rt)>;
  2101: }
  2102: 
  2103: def: Pat<(i1 (seteq (and I64:$Rs, IsPow2_64L:$u6), 0)),
  2104:          (S4_ntstbit_i (LoReg $Rs), (Log2_64 $u6))>;
  2105: def: Pat<(i1 (seteq (and I64:$Rs, IsPow2_64H:$u6), 0)),
  2106:          (S4_ntstbit_i (HiReg $Rs), (UDEC32 (i32 (Log2_64 $u6))))>;
  2107: def: Pat<(i1 (setne (and I64:$Rs, IsPow2_64L:$u6), 0)),
  2108:          (S2_tstbit_i (LoReg $Rs), (Log2_64 imm:$u6))>;
  2109: def: Pat<(i1 (setne (and I64:$Rs, IsPow2_64H:$u6), 0)),
  2110:          (S2_tstbit_i (HiReg $Rs), (UDEC32 (i32 (Log2_64 imm:$u6))))>;
  2111: 
  2112: // Do not increase complexity of these patterns. In the DAG, "cmp i8" may be
  2113: // represented as a compare against "value & 0xFF", which is an exact match
  2114: // for cmpb (same for cmph). The patterns below do not contain any additional
  2115: // complexity that would make them preferable, and if they were actually used
  2116: // instead of cmpb/cmph, they would result in a compare against register that
  2117: // is loaded with the byte/half mask (i.e. 0xFF or 0xFFFF).
  2118: def: Pat<(i1 (setne (and I32:$Rs, u6_0ImmPred:$u6), 0)),
  2119:          (C4_nbitsclri I32:$Rs, imm:$u6)>;
  2120: def: Pat<(i1 (setne (and I32:$Rs, I32:$Rt), 0)),
  2121:          (C4_nbitsclr I32:$Rs, I32:$Rt)>;
  2122: def: Pat<(i1 (setne (and I32:$Rs, I32:$Rt), I32:$Rt)),
  2123:          (C4_nbitsset I32:$Rs, I32:$Rt)>;
  2124: 
  2125: // Special patterns to address certain cases where the "top-down" matching
  2126: // algorithm would cause suboptimal selection.
  2127: 
  2128: let AddedComplexity = 100 in {
  2129:   // Avoid A4_rcmp[n]eqi in these cases:
  2130:   def: Pat<(i32 (zext (i1 (seteq (and (shl 1, I32:$Rt), I32:$Rs), 0)))),
  2131:            (I1toI32 (S4_ntstbit_r IntRegs:$Rs, IntRegs:$Rt))>;
  2132:   def: Pat<(i32 (zext (i1 (setne (and (shl 1, I32:$Rt), I32:$Rs), 0)))),
  2133:            (I1toI32 (S2_tstbit_r IntRegs:$Rs, IntRegs:$Rt))>;
  2134:   def: Pat<(i32 (zext (i1 (seteq (and I32:$Rs, IsPow2_32:$u5), 0)))),
  2135:            (I1toI32 (S4_ntstbit_i I32:$Rs, (Log2_32 imm:$u5)))>;
  2136:   def: Pat<(i32 (zext (i1 (setne (and I32:$Rs, IsPow2_32:$u5), 0)))),
  2137:            (I1toI32 (S2_tstbit_i I32:$Rs, (Log2_32 imm:$u5)))>;
  2138:   def: Pat<(i32 (zext (i1 (seteq (and (shl 1, I32:$Rt), I32:$Rs), 0)))),
  2139:            (I1toI32 (S4_ntstbit_r I32:$Rs, I32:$Rt))>;
  2140:   def: Pat<(i32 (zext (i1 (setne (and (shl 1, I32:$Rt), I32:$Rs), 0)))),
  2141:            (I1toI32 (S2_tstbit_r I32:$Rs, I32:$Rt))>;
  2142: }
  2143: 
  2144: // --(11) PIC ------------------------------------------------------------
  2145: //
  2146: 
  2147: def SDT_HexagonAtGot
  2148:   : SDTypeProfile<1, 3, [SDTCisVT<0, i32>, SDTCisVT<1, i32>, SDTCisVT<2, i32>]>;
  2149: def SDT_HexagonAtPcrel
  2150:   : SDTypeProfile<1, 1, [SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
  2151: 
  2152: // AT_GOT address-of-GOT, address-of-global, offset-in-global
  2153: def HexagonAtGot       : SDNode<"HexagonISD::AT_GOT", SDT_HexagonAtGot>;
  2154: // AT_PCREL address-of-global
  2155: def HexagonAtPcrel     : SDNode<"HexagonISD::AT_PCREL", SDT_HexagonAtPcrel>;
  2156: 
  2157: def: Pat<(HexagonAtGot I32:$got, I32:$addr, (i32 0)),
  2158:          (L2_loadri_io I32:$got, imm:$addr)>;
  2159: def: Pat<(HexagonAtGot I32:$got, I32:$addr, s30_2ImmPred:$off),
  2160:          (A2_addi (L2_loadri_io I32:$got, imm:$addr), imm:$off)>;
  2161: def: Pat<(HexagonAtPcrel I32:$addr),
  2162:          (C4_addipc imm:$addr)>;
  2163: 
  2164: // The HVX load patterns also match AT_PCREL directly. Make sure that
  2165: // if the selection of this opcode changes, it's updated in all places.
  2166: 
  2167: 
  2168: // --(12) Load -----------------------------------------------------------
  2169: //
  2170: 
  2171: def L1toI32:  OutPatFrag<(ops node:$Rs), (A2_subri 0, (i32 $Rs))>;
  2172: def L1toI64:  OutPatFrag<(ops node:$Rs), (ToSext64 (L1toI32 $Rs))>;
  2173: 
  2174: def extloadv2i8: PatFrag<(ops node:$ptr), (extload node:$ptr), [{
  2175:   return cast<LoadSDNode>(N)->getMemoryVT() == MVT::v2i8;
  2176: }]>;
  2177: def extloadv4i8: PatFrag<(ops node:$ptr), (extload node:$ptr), [{
  2178:   return cast<LoadSDNode>(N)->getMemoryVT() == MVT::v4i8;
  2179: }]>;
  2180: 
  2181: def zextloadv2i8: PatFrag<(ops node:$ptr), (zextload node:$ptr), [{
  2182:   return cast<LoadSDNode>(N)->getMemoryVT() == MVT::v2i8;
  2183: }]>;
  2184: def zextloadv4i8: PatFrag<(ops node:$ptr), (zextload node:$ptr), [{
  2185:   return cast<LoadSDNode>(N)->getMemoryVT() == MVT::v4i8;
  2186: }]>;
  2187: 
  2188: def sextloadv2i8: PatFrag<(ops node:$ptr), (sextload node:$ptr), [{
  2189:   return cast<LoadSDNode>(N)->getMemoryVT() == MVT::v2i8;
  2190: }]>;
  2191: def sextloadv4i8: PatFrag<(ops node:$ptr), (sextload node:$ptr), [{
  2192:   return cast<LoadSDNode>(N)->getMemoryVT() == MVT::v4i8;
  2193: }]>;
  2194: 
  2195: // Patterns to select load-indexed: Rs + Off.
  2196: // - frameindex [+ imm],
  2197: multiclass Loadxfi_pat<PatFrags Load, ValueType VT, PatLeaf ImmPred,
  2198:                        InstHexagon MI> {
  2199:   def: Pat<(VT (Load (add (i32 AddrFI:$fi), ImmPred:$Off))),
  2200:            (VT (MI AddrFI:$fi, imm:$Off))>;
```
- EN: It defines declarative TableGen records like SDTTestBit, HexagonTSTBIT, SDT_HexagonAtGot, SDT_HexagonAtPcrel, HexagonAtGot, ... (15 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTSTBIT, HexagonISD, HexagonAtGot, HexagonAtPcrel, showing how the code connects to sibling backend components.
- CN: 这里定义了 SDTTestBit, HexagonTSTBIT, SDT_HexagonAtGot, SDT_HexagonAtPcrel, HexagonAtGot, ... (15 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTSTBIT, HexagonISD, HexagonAtGot, HexagonAtPcrel，说明了它与同级后端组件的连接关系。

### Lines 2201-2400 / 第 2201-2400 行

```tablegen
  2201:   def: Pat<(VT (Load (IsOrAdd (i32 AddrFI:$fi), ImmPred:$Off))),
  2202:            (VT (MI AddrFI:$fi, imm:$Off))>;
  2203:   def: Pat<(VT (Load AddrFI:$fi)), (VT (MI AddrFI:$fi, 0))>;
  2204: }
  2205: 
  2206: // Patterns to select load-indexed: Rs + Off.
  2207: // - base reg [+ imm]
  2208: multiclass Loadxgi_pat<PatFrags Load, ValueType VT, PatLeaf ImmPred,
  2209:                        InstHexagon MI> {
  2210:   def: Pat<(VT (Load (add I32:$Rs, ImmPred:$Off))),
  2211:            (VT (MI IntRegs:$Rs, imm:$Off))>;
  2212:   def: Pat<(VT (Load (IsOrAdd I32:$Rs, ImmPred:$Off))),
  2213:            (VT (MI IntRegs:$Rs, imm:$Off))>;
  2214:   def: Pat<(VT (Load I32:$Rs)), (VT (MI IntRegs:$Rs, 0))>;
  2215: }
  2216: 
  2217: // Patterns to select load-indexed: Rs + Off. Combines Loadxfi + Loadxgi.
  2218: multiclass Loadxi_pat<PatFrags Load, ValueType VT, PatLeaf ImmPred,
  2219:                       InstHexagon MI> {
  2220:   defm: Loadxfi_pat<Load, VT, ImmPred, MI>;
  2221:   defm: Loadxgi_pat<Load, VT, ImmPred, MI>;
  2222: }
  2223: 
  2224: // Patterns to select load reg indexed: Rs + Off with a value modifier.
  2225: // - frameindex [+ imm]
  2226: multiclass Loadxfim_pat<PatFrag Load, ValueType VT, PatFrag ValueMod,
  2227:                         PatLeaf ImmPred, InstHexagon MI> {
  2228:   def: Pat<(VT (Load (add (i32 AddrFI:$fi), ImmPred:$Off))),
  2229:            (VT (ValueMod (MI AddrFI:$fi, imm:$Off)))>;
  2230:   def: Pat<(VT (Load (IsOrAdd (i32 AddrFI:$fi), ImmPred:$Off))),
  2231:            (VT (ValueMod (MI AddrFI:$fi, imm:$Off)))>;
  2232:   def: Pat<(VT (Load AddrFI:$fi)), (VT (ValueMod (MI AddrFI:$fi, 0)))>;
  2233: }
  2234: 
  2235: // Patterns to select load reg indexed: Rs + Off with a value modifier.
  2236: // - base reg [+ imm]
  2237: multiclass Loadxgim_pat<PatFrag Load, ValueType VT, PatFrag ValueMod,
  2238:                         PatLeaf ImmPred, InstHexagon MI> {
  2239:   def: Pat<(VT (Load (add I32:$Rs, ImmPred:$Off))),
  2240:            (VT (ValueMod (MI IntRegs:$Rs, imm:$Off)))>;
  2241:   def: Pat<(VT (Load (IsOrAdd I32:$Rs, ImmPred:$Off))),
  2242:            (VT (ValueMod (MI IntRegs:$Rs, imm:$Off)))>;
  2243:   def: Pat<(VT (Load I32:$Rs)), (VT (ValueMod (MI IntRegs:$Rs, 0)))>;
  2244: }
  2245: 
  2246: // Patterns to select load reg indexed: Rs + Off with a value modifier.
  2247: // Combines Loadxfim + Loadxgim.
  2248: multiclass Loadxim_pat<PatFrag Load, ValueType VT, PatFrag ValueMod,
  2249:                        PatLeaf ImmPred, InstHexagon MI> {
  2250:   defm: Loadxfim_pat<Load, VT, ValueMod, ImmPred, MI>;
  2251:   defm: Loadxgim_pat<Load, VT, ValueMod, ImmPred, MI>;
  2252: }
  2253: 
  2254: // Pattern to select load reg reg-indexed: Rs + Rt<<u2.
  2255: class Loadxr_shl_pat<PatFrag Load, ValueType VT, InstHexagon MI>
  2256:   : Pat<(VT (Load (add I32:$Rs, (i32 (shl I32:$Rt, u2_0ImmPred:$u2))))),
  2257:         (VT (MI IntRegs:$Rs, IntRegs:$Rt, imm:$u2))>;
  2258: 
  2259: // Pattern to select load reg reg-indexed: Rs + Rt<<0.
  2260: class Loadxr_add_pat<PatFrag Load, ValueType VT, InstHexagon MI>
  2261:   : Pat<(VT (Load (add I32:$Rs, I32:$Rt))),
  2262:         (VT (MI IntRegs:$Rs, IntRegs:$Rt, 0))>;
  2263: 
  2264: // Pattern to select load reg reg-indexed: Rs + Rt<<u2 with value modifier.
  2265: class Loadxrm_shl_pat<PatFrag Load, ValueType VT, PatFrag ValueMod,
  2266:                       InstHexagon MI>
  2267:   : Pat<(VT (Load (add I32:$Rs, (i32 (shl I32:$Rt, u2_0ImmPred:$u2))))),
  2268:         (VT (ValueMod (MI IntRegs:$Rs, IntRegs:$Rt, imm:$u2)))>;
  2269: 
  2270: // Pattern to select load reg reg-indexed: Rs + Rt<<0 with value modifier.
  2271: class Loadxrm_add_pat<PatFrag Load, ValueType VT, PatFrag ValueMod,
  2272:                       InstHexagon MI>
  2273:   : Pat<(VT (Load (add I32:$Rs, I32:$Rt))),
  2274:         (VT (ValueMod (MI IntRegs:$Rs, IntRegs:$Rt, 0)))>;
  2275: 
  2276: // Pattern to select load long-offset reg-indexed: Addr + Rt<<u2.
  2277: // Don't match for u2==0, instead use reg+imm for those cases.
  2278: class Loadxu_pat<PatFrag Load, ValueType VT, PatFrag ImmPred, InstHexagon MI>
  2279:   : Pat<(VT (Load (add (shl IntRegs:$Rt, u2_0ImmPred:$u2), ImmPred:$Addr))),
  2280:         (VT (MI IntRegs:$Rt, imm:$u2, ImmPred:$Addr))>;
  2281: 
  2282: class Loadxum_pat<PatFrag Load, ValueType VT, PatFrag ImmPred, PatFrag ValueMod,
  2283:                   InstHexagon MI>
  2284:   : Pat<(VT (Load (add (shl IntRegs:$Rt, u2_0ImmPred:$u2), ImmPred:$Addr))),
  2285:         (VT (ValueMod (MI IntRegs:$Rt, imm:$u2, ImmPred:$Addr)))>;
  2286: 
  2287: // Pattern to select load absolute.
  2288: class Loada_pat<PatFrags Load, ValueType VT, PatFrag Addr, InstHexagon MI>
  2289:   : Pat<(VT (Load Addr:$addr)), (MI Addr:$addr)>;
  2290: 
  2291: // Pattern to select load absolute with value modifier.
  2292: class Loadam_pat<PatFrag Load, ValueType VT, PatFrag Addr, PatFrag ValueMod,
  2293:                  InstHexagon MI>
  2294:   : Pat<(VT (Load Addr:$addr)), (ValueMod (MI Addr:$addr))>;
  2295: 
  2296: 
  2297: let AddedComplexity = 20 in {
  2298:   defm: Loadxi_pat<extloadi1,       i32,   anyimm0, L2_loadrub_io>;
  2299:   defm: Loadxi_pat<extloadi8,       i32,   anyimm0, L2_loadrub_io>;
  2300:   defm: Loadxi_pat<extloadi16,      i32,   anyimm1, L2_loadruh_io>;
  2301:   defm: Loadxi_pat<extloadv2i8,     v2i16, anyimm1, L2_loadbzw2_io>;
  2302:   defm: Loadxi_pat<extloadv4i8,     v4i16, anyimm2, L2_loadbzw4_io>;
  2303:   defm: Loadxi_pat<sextloadi8,      i32,   anyimm0, L2_loadrb_io>;
  2304:   defm: Loadxi_pat<sextloadi16,     i32,   anyimm1, L2_loadrh_io>;
  2305:   defm: Loadxi_pat<sextloadv2i8,    v2i16, anyimm1, L2_loadbsw2_io>;
  2306:   defm: Loadxi_pat<sextloadv4i8,    v4i16, anyimm2, L2_loadbsw4_io>;
  2307:   defm: Loadxi_pat<zextloadi1,      i32,   anyimm0, L2_loadrub_io>;
  2308:   defm: Loadxi_pat<zextloadi8,      i32,   anyimm0, L2_loadrub_io>;
  2309:   defm: Loadxi_pat<zextloadi16,     i32,   anyimm1, L2_loadruh_io>;
  2310:   defm: Loadxi_pat<zextloadv2i8,    v2i16, anyimm1, L2_loadbzw2_io>;
  2311:   defm: Loadxi_pat<zextloadv4i8,    v4i16, anyimm2, L2_loadbzw4_io>;
  2312:   defm: Loadxi_pat<load,            i32,   anyimm2, L2_loadri_io>;
  2313:   defm: Loadxi_pat<load,            v2i16, anyimm2, L2_loadri_io>;
  2314:   defm: Loadxi_pat<load,            v4i8,  anyimm2, L2_loadri_io>;
  2315:   defm: Loadxi_pat<load,            i64,   anyimm3, L2_loadrd_io>;
  2316:   defm: Loadxi_pat<load,            v2i32, anyimm3, L2_loadrd_io>;
  2317:   defm: Loadxi_pat<load,            v4i16, anyimm3, L2_loadrd_io>;
  2318:   defm: Loadxi_pat<load,            v8i8,  anyimm3, L2_loadrd_io>;
  2319:   defm: Loadxi_pat<load,            f32,   anyimm2, L2_loadri_io>;
  2320:   defm: Loadxi_pat<load,            f64,   anyimm3, L2_loadrd_io>;
  2321:   // No sextloadi1.
  2322: 
  2323:   defm: Loadxi_pat<atomic_load_azext_8 ,  i32, anyimm0, L2_loadrub_io>;
  2324:   defm: Loadxi_pat<atomic_load_azext_16,  i32, anyimm1, L2_loadruh_io>;
  2325:   defm: Loadxi_pat<atomic_load_nonext_32,  i32, anyimm2, L2_loadri_io>;
  2326:   defm: Loadxi_pat<atomic_load_nonext_64,  i64, anyimm3, L2_loadrd_io>;
  2327: }
  2328: 
  2329: let AddedComplexity = 30 in {
  2330:   // Loads of i1 are loading a byte, and the byte should be either 0 or 1.
  2331:   // It doesn't matter if it's sign- or zero-extended, so use zero-extension
  2332:   // everywhere.
  2333:   defm: Loadxim_pat<sextloadi1,   i32, L1toI32,  anyimm0, L2_loadrub_io>;
  2334:   defm: Loadxim_pat<extloadi1,    i64, ToAext64, anyimm0, L2_loadrub_io>;
  2335:   defm: Loadxim_pat<sextloadi1,   i64, L1toI64,  anyimm0, L2_loadrub_io>;
  2336:   defm: Loadxim_pat<zextloadi1,   i64, ToZext64, anyimm0, L2_loadrub_io>;
  2337: 
  2338:   defm: Loadxim_pat<extloadi8,    i64, ToAext64, anyimm0, L2_loadrub_io>;
  2339:   defm: Loadxim_pat<extloadi16,   i64, ToAext64, anyimm1, L2_loadruh_io>;
  2340:   defm: Loadxim_pat<extloadi32,   i64, ToAext64, anyimm2, L2_loadri_io>;
  2341:   defm: Loadxim_pat<zextloadi8,   i64, ToZext64, anyimm0, L2_loadrub_io>;
  2342:   defm: Loadxim_pat<zextloadi16,  i64, ToZext64, anyimm1, L2_loadruh_io>;
  2343:   defm: Loadxim_pat<zextloadi32,  i64, ToZext64, anyimm2, L2_loadri_io>;
  2344:   defm: Loadxim_pat<sextloadi8,   i64, ToSext64, anyimm0, L2_loadrb_io>;
  2345:   defm: Loadxim_pat<sextloadi16,  i64, ToSext64, anyimm1, L2_loadrh_io>;
  2346:   defm: Loadxim_pat<sextloadi32,  i64, ToSext64, anyimm2, L2_loadri_io>;
  2347: }
  2348: 
  2349: let AddedComplexity  = 60 in {
  2350:   def: Loadxu_pat<extloadi1,    i32,   anyimm0, L4_loadrub_ur>;
  2351:   def: Loadxu_pat<extloadi8,    i32,   anyimm0, L4_loadrub_ur>;
  2352:   def: Loadxu_pat<extloadi16,   i32,   anyimm1, L4_loadruh_ur>;
  2353:   def: Loadxu_pat<extloadv2i8,  v2i16, anyimm1, L4_loadbzw2_ur>;
  2354:   def: Loadxu_pat<extloadv4i8,  v4i16, anyimm2, L4_loadbzw4_ur>;
  2355:   def: Loadxu_pat<sextloadi8,   i32,   anyimm0, L4_loadrb_ur>;
  2356:   def: Loadxu_pat<sextloadi16,  i32,   anyimm1, L4_loadrh_ur>;
  2357:   def: Loadxu_pat<sextloadv2i8, v2i16, anyimm1, L4_loadbsw2_ur>;
  2358:   def: Loadxu_pat<sextloadv4i8, v4i16, anyimm2, L4_loadbsw4_ur>;
  2359:   def: Loadxu_pat<zextloadi1,   i32,   anyimm0, L4_loadrub_ur>;
  2360:   def: Loadxu_pat<zextloadi8,   i32,   anyimm0, L4_loadrub_ur>;
  2361:   def: Loadxu_pat<zextloadi16,  i32,   anyimm1, L4_loadruh_ur>;
  2362:   def: Loadxu_pat<zextloadv2i8, v2i16, anyimm1, L4_loadbzw2_ur>;
  2363:   def: Loadxu_pat<zextloadv4i8, v4i16, anyimm2, L4_loadbzw4_ur>;
  2364:   def: Loadxu_pat<load,         i32,   anyimm2, L4_loadri_ur>;
  2365:   def: Loadxu_pat<load,         v2i16, anyimm2, L4_loadri_ur>;
  2366:   def: Loadxu_pat<load,         v4i8,  anyimm2, L4_loadri_ur>;
  2367:   def: Loadxu_pat<load,         i64,   anyimm3, L4_loadrd_ur>;
  2368:   def: Loadxu_pat<load,         v2i32, anyimm3, L4_loadrd_ur>;
  2369:   def: Loadxu_pat<load,         v4i16, anyimm3, L4_loadrd_ur>;
  2370:   def: Loadxu_pat<load,         v8i8,  anyimm3, L4_loadrd_ur>;
  2371:   def: Loadxu_pat<load,         f32,   anyimm2, L4_loadri_ur>;
  2372:   def: Loadxu_pat<load,         f64,   anyimm3, L4_loadrd_ur>;
  2373: 
  2374:   def: Loadxum_pat<sextloadi1,  i32, anyimm0, L1toI32,  L4_loadrub_ur>;
  2375:   def: Loadxum_pat<extloadi1,   i64, anyimm0, ToAext64, L4_loadrub_ur>;
  2376:   def: Loadxum_pat<sextloadi1,  i64, anyimm0, L1toI64,  L4_loadrub_ur>;
  2377:   def: Loadxum_pat<zextloadi1,  i64, anyimm0, ToZext64, L4_loadrub_ur>;
  2378: 
  2379:   def: Loadxum_pat<sextloadi8,  i64, anyimm0, ToSext64, L4_loadrb_ur>;
  2380:   def: Loadxum_pat<zextloadi8,  i64, anyimm0, ToZext64, L4_loadrub_ur>;
  2381:   def: Loadxum_pat<extloadi8,   i64, anyimm0, ToAext64, L4_loadrub_ur>;
  2382:   def: Loadxum_pat<sextloadi16, i64, anyimm1, ToSext64, L4_loadrh_ur>;
  2383:   def: Loadxum_pat<zextloadi16, i64, anyimm1, ToZext64, L4_loadruh_ur>;
  2384:   def: Loadxum_pat<extloadi16,  i64, anyimm1, ToAext64, L4_loadruh_ur>;
  2385:   def: Loadxum_pat<sextloadi32, i64, anyimm2, ToSext64, L4_loadri_ur>;
  2386:   def: Loadxum_pat<zextloadi32, i64, anyimm2, ToZext64, L4_loadri_ur>;
  2387:   def: Loadxum_pat<extloadi32,  i64, anyimm2, ToAext64, L4_loadri_ur>;
  2388: }
  2389: 
  2390: let AddedComplexity = 40 in {
  2391:   def: Loadxr_shl_pat<extloadi1,     i32,   L4_loadrub_rr>;
  2392:   def: Loadxr_shl_pat<extloadi8,     i32,   L4_loadrub_rr>;
  2393:   def: Loadxr_shl_pat<zextloadi1,    i32,   L4_loadrub_rr>;
  2394:   def: Loadxr_shl_pat<zextloadi8,    i32,   L4_loadrub_rr>;
  2395:   def: Loadxr_shl_pat<sextloadi8,    i32,   L4_loadrb_rr>;
  2396:   def: Loadxr_shl_pat<extloadi16,    i32,   L4_loadruh_rr>;
  2397:   def: Loadxr_shl_pat<zextloadi16,   i32,   L4_loadruh_rr>;
  2398:   def: Loadxr_shl_pat<sextloadi16,   i32,   L4_loadrh_rr>;
  2399:   def: Loadxr_shl_pat<load,          i32,   L4_loadri_rr>;
  2400:   def: Loadxr_shl_pat<load,          v2i16, L4_loadri_rr>;
```
- EN: It declares types such as Loadxr_shl_pat, Loadxr_add_pat, Loadxrm_shl_pat, Loadxrm_add_pat, ... (8 total), which carry the state or API of this component. It defines declarative TableGen records like Loadxgi_pat, Loadxi_pat, Loadxfim_pat, Loadxgim_pat, Loadxim_pat, ... (13 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 Loadxr_shl_pat, Loadxr_add_pat, Loadxrm_shl_pat, Loadxrm_add_pat, ... (8 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Loadxgi_pat, Loadxi_pat, Loadxfim_pat, Loadxgim_pat, Loadxim_pat, ... (13 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 2401-2600 / 第 2401-2600 行

```tablegen
  2401:   def: Loadxr_shl_pat<load,          v4i8,  L4_loadri_rr>;
  2402:   def: Loadxr_shl_pat<load,          i64,   L4_loadrd_rr>;
  2403:   def: Loadxr_shl_pat<load,          v2i32, L4_loadrd_rr>;
  2404:   def: Loadxr_shl_pat<load,          v4i16, L4_loadrd_rr>;
  2405:   def: Loadxr_shl_pat<load,          v8i8,  L4_loadrd_rr>;
  2406:   def: Loadxr_shl_pat<load,          f32,   L4_loadri_rr>;
  2407:   def: Loadxr_shl_pat<load,          f64,   L4_loadrd_rr>;
  2408: }
  2409: 
  2410: let AddedComplexity = 20 in {
  2411:   def: Loadxr_add_pat<extloadi1,     i32,   L4_loadrub_rr>;
  2412:   def: Loadxr_add_pat<extloadi8,     i32,   L4_loadrub_rr>;
  2413:   def: Loadxr_add_pat<zextloadi8,    i32,   L4_loadrub_rr>;
  2414:   def: Loadxr_add_pat<zextloadi1,    i32,   L4_loadrub_rr>;
  2415:   def: Loadxr_add_pat<sextloadi8,    i32,   L4_loadrb_rr>;
  2416:   def: Loadxr_add_pat<extloadi16,    i32,   L4_loadruh_rr>;
  2417:   def: Loadxr_add_pat<zextloadi16,   i32,   L4_loadruh_rr>;
  2418:   def: Loadxr_add_pat<sextloadi16,   i32,   L4_loadrh_rr>;
  2419:   def: Loadxr_add_pat<load,          i32,   L4_loadri_rr>;
  2420:   def: Loadxr_add_pat<load,          v2i16, L4_loadri_rr>;
  2421:   def: Loadxr_add_pat<load,          v4i8,  L4_loadri_rr>;
  2422:   def: Loadxr_add_pat<load,          i64,   L4_loadrd_rr>;
  2423:   def: Loadxr_add_pat<load,          v2i32, L4_loadrd_rr>;
  2424:   def: Loadxr_add_pat<load,          v4i16, L4_loadrd_rr>;
  2425:   def: Loadxr_add_pat<load,          v8i8,  L4_loadrd_rr>;
  2426:   def: Loadxr_add_pat<load,          f32,   L4_loadri_rr>;
  2427:   def: Loadxr_add_pat<load,          f64,   L4_loadrd_rr>;
  2428: }
  2429: 
  2430: let AddedComplexity = 40 in {
  2431:   def: Loadxrm_shl_pat<sextloadi1,   i32, L1toI32,  L4_loadrub_rr>;
  2432:   def: Loadxrm_shl_pat<extloadi1,    i64, ToAext64, L4_loadrub_rr>;
  2433:   def: Loadxrm_shl_pat<sextloadi1,   i64, L1toI64,  L4_loadrub_rr>;
  2434:   def: Loadxrm_shl_pat<zextloadi1,   i64, ToZext64, L4_loadrub_rr>;
  2435: 
  2436:   def: Loadxrm_shl_pat<extloadi8,    i64, ToAext64, L4_loadrub_rr>;
  2437:   def: Loadxrm_shl_pat<zextloadi8,   i64, ToZext64, L4_loadrub_rr>;
  2438:   def: Loadxrm_shl_pat<sextloadi8,   i64, ToSext64, L4_loadrb_rr>;
  2439:   def: Loadxrm_shl_pat<extloadi16,   i64, ToAext64, L4_loadruh_rr>;
  2440:   def: Loadxrm_shl_pat<zextloadi16,  i64, ToZext64, L4_loadruh_rr>;
  2441:   def: Loadxrm_shl_pat<sextloadi16,  i64, ToSext64, L4_loadrh_rr>;
  2442:   def: Loadxrm_shl_pat<extloadi32,   i64, ToAext64, L4_loadri_rr>;
  2443:   def: Loadxrm_shl_pat<zextloadi32,  i64, ToZext64, L4_loadri_rr>;
  2444:   def: Loadxrm_shl_pat<sextloadi32,  i64, ToSext64, L4_loadri_rr>;
  2445: }
  2446: 
  2447: let AddedComplexity = 30 in {
  2448:   def: Loadxrm_add_pat<sextloadi1,   i32, L1toI32,  L4_loadrub_rr>;
  2449:   def: Loadxrm_add_pat<extloadi1,    i64, ToAext64, L4_loadrub_rr>;
  2450:   def: Loadxrm_add_pat<sextloadi1,   i64, L1toI64,  L4_loadrub_rr>;
  2451:   def: Loadxrm_add_pat<zextloadi1,   i64, ToZext64, L4_loadrub_rr>;
  2452: 
  2453:   def: Loadxrm_add_pat<extloadi8,    i64, ToAext64, L4_loadrub_rr>;
  2454:   def: Loadxrm_add_pat<zextloadi8,   i64, ToZext64, L4_loadrub_rr>;
  2455:   def: Loadxrm_add_pat<sextloadi8,   i64, ToSext64, L4_loadrb_rr>;
  2456:   def: Loadxrm_add_pat<extloadi16,   i64, ToAext64, L4_loadruh_rr>;
  2457:   def: Loadxrm_add_pat<zextloadi16,  i64, ToZext64, L4_loadruh_rr>;
  2458:   def: Loadxrm_add_pat<sextloadi16,  i64, ToSext64, L4_loadrh_rr>;
  2459:   def: Loadxrm_add_pat<extloadi32,   i64, ToAext64, L4_loadri_rr>;
  2460:   def: Loadxrm_add_pat<zextloadi32,  i64, ToZext64, L4_loadri_rr>;
  2461:   def: Loadxrm_add_pat<sextloadi32,  i64, ToSext64, L4_loadri_rr>;
  2462: }
  2463: 
  2464: // Absolute address
  2465: 
  2466: let AddedComplexity  = 60 in {
  2467:   def: Loada_pat<extloadi1,       i32,   anyimm0, PS_loadrubabs>;
  2468:   def: Loada_pat<zextloadi1,      i32,   anyimm0, PS_loadrubabs>;
  2469:   def: Loada_pat<extloadi8,       i32,   anyimm0, PS_loadrubabs>;
  2470:   def: Loada_pat<sextloadi8,      i32,   anyimm0, PS_loadrbabs>;
  2471:   def: Loada_pat<zextloadi8,      i32,   anyimm0, PS_loadrubabs>;
  2472:   def: Loada_pat<extloadi16,      i32,   anyimm1, PS_loadruhabs>;
  2473:   def: Loada_pat<sextloadi16,     i32,   anyimm1, PS_loadrhabs>;
  2474:   def: Loada_pat<zextloadi16,     i32,   anyimm1, PS_loadruhabs>;
  2475:   def: Loada_pat<load,            i32,   anyimm2, PS_loadriabs>;
  2476:   def: Loada_pat<load,            v2i16, anyimm2, PS_loadriabs>;
  2477:   def: Loada_pat<load,            v4i8,  anyimm2, PS_loadriabs>;
  2478:   def: Loada_pat<load,            i64,   anyimm3, PS_loadrdabs>;
  2479:   def: Loada_pat<load,            v2i32, anyimm3, PS_loadrdabs>;
  2480:   def: Loada_pat<load,            v4i16, anyimm3, PS_loadrdabs>;
  2481:   def: Loada_pat<load,            v8i8,  anyimm3, PS_loadrdabs>;
  2482:   def: Loada_pat<load,            f32,   anyimm2, PS_loadriabs>;
  2483:   def: Loada_pat<load,            f64,   anyimm3, PS_loadrdabs>;
  2484: 
  2485:   def: Loada_pat<atomic_load_azext_8,   i32, anyimm0, PS_loadrubabs>;
  2486:   def: Loada_pat<atomic_load_azext_16,  i32, anyimm1, PS_loadruhabs>;
  2487:   def: Loada_pat<atomic_load_nonext_32,  i32, anyimm2, PS_loadriabs>;
  2488:   def: Loada_pat<atomic_load_nonext_64,  i64, anyimm3, PS_loadrdabs>;
  2489: }
  2490: 
  2491: let AddedComplexity  = 30 in {
  2492:   def: Loadam_pat<load,           i1,  anyimm0, I32toI1,  PS_loadrubabs>;
  2493:   def: Loadam_pat<sextloadi1,     i32, anyimm0, L1toI32,  PS_loadrubabs>;
  2494:   def: Loadam_pat<extloadi1,      i64, anyimm0, ToZext64, PS_loadrubabs>;
  2495:   def: Loadam_pat<sextloadi1,     i64, anyimm0, L1toI64,  PS_loadrubabs>;
  2496:   def: Loadam_pat<zextloadi1,     i64, anyimm0, ToZext64, PS_loadrubabs>;
  2497: 
  2498:   def: Loadam_pat<extloadi8,      i64, anyimm0, ToAext64, PS_loadrubabs>;
  2499:   def: Loadam_pat<sextloadi8,     i64, anyimm0, ToSext64, PS_loadrbabs>;
  2500:   def: Loadam_pat<zextloadi8,     i64, anyimm0, ToZext64, PS_loadrubabs>;
  2501:   def: Loadam_pat<extloadi16,     i64, anyimm1, ToAext64, PS_loadruhabs>;
  2502:   def: Loadam_pat<sextloadi16,    i64, anyimm1, ToSext64, PS_loadrhabs>;
  2503:   def: Loadam_pat<zextloadi16,    i64, anyimm1, ToZext64, PS_loadruhabs>;
  2504:   def: Loadam_pat<extloadi32,     i64, anyimm2, ToAext64, PS_loadriabs>;
  2505:   def: Loadam_pat<sextloadi32,    i64, anyimm2, ToSext64, PS_loadriabs>;
  2506:   def: Loadam_pat<zextloadi32,    i64, anyimm2, ToZext64, PS_loadriabs>;
  2507: }
  2508: 
  2509: // GP-relative address
  2510: 
  2511: let AddedComplexity  = 100 in {
  2512:   def: Loada_pat<extloadi1,       i32,   addrgp,  L2_loadrubgp>;
  2513:   def: Loada_pat<zextloadi1,      i32,   addrgp,  L2_loadrubgp>;
  2514:   def: Loada_pat<extloadi8,       i32,   addrgp,  L2_loadrubgp>;
  2515:   def: Loada_pat<sextloadi8,      i32,   addrgp,  L2_loadrbgp>;
  2516:   def: Loada_pat<zextloadi8,      i32,   addrgp,  L2_loadrubgp>;
  2517:   def: Loada_pat<extloadi16,      i32,   addrgp,  L2_loadruhgp>;
  2518:   def: Loada_pat<sextloadi16,     i32,   addrgp,  L2_loadrhgp>;
  2519:   def: Loada_pat<zextloadi16,     i32,   addrgp,  L2_loadruhgp>;
  2520:   def: Loada_pat<load,            i32,   addrgp,  L2_loadrigp>;
  2521:   def: Loada_pat<load,            v2i16, addrgp,  L2_loadrigp>;
  2522:   def: Loada_pat<load,            v4i8,  addrgp,  L2_loadrigp>;
  2523:   def: Loada_pat<load,            i64,   addrgp,  L2_loadrdgp>;
  2524:   def: Loada_pat<load,            v2i32, addrgp,  L2_loadrdgp>;
  2525:   def: Loada_pat<load,            v4i16, addrgp,  L2_loadrdgp>;
  2526:   def: Loada_pat<load,            v8i8,  addrgp,  L2_loadrdgp>;
  2527:   def: Loada_pat<load,            f32,   addrgp,  L2_loadrigp>;
  2528:   def: Loada_pat<load,            f64,   addrgp,  L2_loadrdgp>;
  2529: 
  2530:   def: Loada_pat<atomic_load_azext_8,   i32, addrgp,  L2_loadrubgp>;
  2531:   def: Loada_pat<atomic_load_azext_16,  i32, addrgp,  L2_loadruhgp>;
  2532:   def: Loada_pat<atomic_load_nonext_32,  i32, addrgp,  L2_loadrigp>;
  2533:   def: Loada_pat<atomic_load_nonext_64,  i64, addrgp,  L2_loadrdgp>;
  2534: }
  2535: 
  2536: let AddedComplexity  = 70 in {
  2537:   def: Loadam_pat<sextloadi1,     i32, addrgp,  L1toI32,  L2_loadrubgp>;
  2538:   def: Loadam_pat<extloadi1,      i64, addrgp,  ToAext64, L2_loadrubgp>;
  2539:   def: Loadam_pat<sextloadi1,     i64, addrgp,  L1toI64,  L2_loadrubgp>;
  2540:   def: Loadam_pat<zextloadi1,     i64, addrgp,  ToZext64, L2_loadrubgp>;
  2541: 
  2542:   def: Loadam_pat<extloadi8,      i64, addrgp,  ToAext64, L2_loadrubgp>;
  2543:   def: Loadam_pat<sextloadi8,     i64, addrgp,  ToSext64, L2_loadrbgp>;
  2544:   def: Loadam_pat<zextloadi8,     i64, addrgp,  ToZext64, L2_loadrubgp>;
  2545:   def: Loadam_pat<extloadi16,     i64, addrgp,  ToAext64, L2_loadruhgp>;
  2546:   def: Loadam_pat<sextloadi16,    i64, addrgp,  ToSext64, L2_loadrhgp>;
  2547:   def: Loadam_pat<zextloadi16,    i64, addrgp,  ToZext64, L2_loadruhgp>;
  2548:   def: Loadam_pat<extloadi32,     i64, addrgp,  ToAext64, L2_loadrigp>;
  2549:   def: Loadam_pat<sextloadi32,    i64, addrgp,  ToSext64, L2_loadrigp>;
  2550:   def: Loadam_pat<zextloadi32,    i64, addrgp,  ToZext64, L2_loadrigp>;
  2551: 
  2552:   def: Loadam_pat<load,           i1,  addrgp,  I32toI1,  L2_loadrubgp>;
  2553: }
  2554: 
  2555: // Patterns for loads of i1:
  2556: def: Pat<(i1 (load AddrFI:$fi)),
  2557:          (C2_tfrrp (L2_loadrub_io AddrFI:$fi, 0))>;
  2558: def: Pat<(i1 (load (add I32:$Rs, anyimm0:$Off))),
  2559:          (C2_tfrrp (L2_loadrub_io IntRegs:$Rs, imm:$Off))>;
  2560: def: Pat<(i1 (load I32:$Rs)),
  2561:          (C2_tfrrp (L2_loadrub_io IntRegs:$Rs, 0))>;
  2562: 
  2563: 
  2564: // --(13) Store ----------------------------------------------------------
  2565: //
  2566: 
  2567: class Storepi_pat<PatFrag Store, PatFrag Value, PatFrag Offset, InstHexagon MI>
  2568:   : Pat<(Store Value:$Rt, I32:$Rx, Offset:$s4),
  2569:         (MI I32:$Rx, imm:$s4, Value:$Rt)>;
  2570: 
  2571: def: Storepi_pat<post_truncsti8,  I32, s4_0ImmPred, S2_storerb_pi>;
  2572: def: Storepi_pat<post_truncsti16, I32, s4_1ImmPred, S2_storerh_pi>;
  2573: def: Storepi_pat<post_store,      I32, s4_2ImmPred, S2_storeri_pi>;
  2574: def: Storepi_pat<post_store,      I64, s4_3ImmPred, S2_storerd_pi>;
  2575: 
  2576: // Patterns for generating stores, where the address takes different forms:
  2577: // - frameindex,
  2578: // - frameindex + offset,
  2579: // - base + offset,
  2580: // - simple (base address without offset).
  2581: // These would usually be used together (via Storexi_pat defined below), but
  2582: // in some cases one may want to apply different properties (such as
  2583: // AddedComplexity) to the individual patterns.
  2584: class Storexi_fi_pat<PatFrag Store, PatFrag Value, InstHexagon MI>
  2585:   : Pat<(Store Value:$Rs, AddrFI:$fi), (MI AddrFI:$fi, 0, Value:$Rs)>;
  2586: 
  2587: multiclass Storexi_fi_add_pat<PatFrag Store, PatFrag Value, PatFrag ImmPred,
  2588:                               InstHexagon MI> {
  2589:   def: Pat<(Store Value:$Rs, (add (i32 AddrFI:$fi), ImmPred:$Off)),
  2590:            (MI AddrFI:$fi, imm:$Off, Value:$Rs)>;
  2591:   def: Pat<(Store Value:$Rs, (IsOrAdd (i32 AddrFI:$fi), ImmPred:$Off)),
  2592:            (MI AddrFI:$fi, imm:$Off, Value:$Rs)>;
  2593: }
  2594: 
  2595: multiclass Storexi_add_pat<PatFrag Store, PatFrag Value, PatFrag ImmPred,
  2596:                            InstHexagon MI> {
  2597:   def: Pat<(Store Value:$Rt, (add I32:$Rs, ImmPred:$Off)),
  2598:            (MI IntRegs:$Rs, imm:$Off, Value:$Rt)>;
  2599:   def: Pat<(Store Value:$Rt, (IsOrAdd I32:$Rs, ImmPred:$Off)),
  2600:            (MI IntRegs:$Rs, imm:$Off, Value:$Rt)>;
```
- EN: It declares types such as Storepi_pat, Storexi_fi_pat, which carry the state or API of this component. It defines declarative TableGen records like Storepi_pat, Storexi_fi_pat, Storexi_fi_add_pat, Storexi_add_pat; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 Storepi_pat, Storexi_fi_pat 等类型，用来承载该组件的状态或接口。 这里定义了 Storepi_pat, Storexi_fi_pat, Storexi_fi_add_pat, Storexi_add_pat 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 2601-2800 / 第 2601-2800 行

```tablegen
  2601: }
  2602: 
  2603: class Storexi_base_pat<PatFrag Store, PatFrag Value, InstHexagon MI>
  2604:   : Pat<(Store Value:$Rt, I32:$Rs),
  2605:         (MI IntRegs:$Rs, 0, Value:$Rt)>;
  2606: 
  2607: // Patterns for generating stores, where the address takes different forms,
  2608: // and where the value being stored is transformed through the value modifier
  2609: // ValueMod.  The address forms are same as above.
  2610: class Storexim_fi_pat<PatFrag Store, PatFrag Value, PatFrag ValueMod,
  2611:                       InstHexagon MI>
  2612:   : Pat<(Store Value:$Rs, AddrFI:$fi),
  2613:         (MI AddrFI:$fi, 0, (ValueMod Value:$Rs))>;
  2614: 
  2615: multiclass Storexim_fi_add_pat<PatFrag Store, PatFrag Value, PatFrag ImmPred,
  2616:                                PatFrag ValueMod, InstHexagon MI> {
  2617:   def: Pat<(Store Value:$Rs, (add (i32 AddrFI:$fi), ImmPred:$Off)),
  2618:            (MI AddrFI:$fi, imm:$Off, (ValueMod Value:$Rs))>;
  2619:   def: Pat<(Store Value:$Rs, (IsOrAdd (i32 AddrFI:$fi), ImmPred:$Off)),
  2620:            (MI AddrFI:$fi, imm:$Off, (ValueMod Value:$Rs))>;
  2621: }
  2622: 
  2623: multiclass Storexim_add_pat<PatFrag Store, PatFrag Value, PatFrag ImmPred,
  2624:                             PatFrag ValueMod, InstHexagon MI> {
  2625:   def: Pat<(Store Value:$Rt, (add I32:$Rs, ImmPred:$Off)),
  2626:            (MI IntRegs:$Rs, imm:$Off, (ValueMod Value:$Rt))>;
  2627:   def: Pat<(Store Value:$Rt, (IsOrAdd I32:$Rs, ImmPred:$Off)),
  2628:            (MI IntRegs:$Rs, imm:$Off, (ValueMod Value:$Rt))>;
  2629: }
  2630: 
  2631: class Storexim_base_pat<PatFrag Store, PatFrag Value, PatFrag ValueMod,
  2632:                         InstHexagon MI>
  2633:   : Pat<(Store Value:$Rt, I32:$Rs),
  2634:         (MI IntRegs:$Rs, 0, (ValueMod Value:$Rt))>;
  2635: 
  2636: multiclass Storexi_pat<PatFrag Store, PatFrag Value, PatLeaf ImmPred,
  2637:                        InstHexagon MI> {
  2638:   defm: Storexi_fi_add_pat <Store, Value, ImmPred, MI>;
  2639:   def:  Storexi_fi_pat     <Store, Value,          MI>;
  2640:   defm: Storexi_add_pat    <Store, Value, ImmPred, MI>;
  2641: }
  2642: 
  2643: multiclass Storexim_pat<PatFrag Store, PatFrag Value, PatLeaf ImmPred,
  2644:                         PatFrag ValueMod, InstHexagon MI> {
  2645:   defm: Storexim_fi_add_pat <Store, Value, ImmPred, ValueMod, MI>;
  2646:   def:  Storexim_fi_pat     <Store, Value,          ValueMod, MI>;
  2647:   defm: Storexim_add_pat    <Store, Value, ImmPred, ValueMod, MI>;
  2648: }
  2649: 
  2650: // Reg<<S + Imm
  2651: class Storexu_shl_pat<PatFrag Store, PatFrag Value, PatFrag ImmPred, InstHexagon MI>
  2652:   : Pat<(Store Value:$Rt, (add (shl I32:$Ru, u2_0ImmPred:$u2), ImmPred:$A)),
  2653:         (MI IntRegs:$Ru, imm:$u2, ImmPred:$A, Value:$Rt)>;
  2654: 
  2655: // Reg<<S + Reg
  2656: class Storexr_shl_pat<PatFrag Store, PatFrag Value, InstHexagon MI>
  2657:   : Pat<(Store Value:$Ru, (add I32:$Rs, (shl I32:$Rt, u2_0ImmPred:$u2))),
  2658:         (MI IntRegs:$Rs, IntRegs:$Rt, imm:$u2, Value:$Ru)>;
  2659: 
  2660: // Reg + Reg
  2661: class Storexr_add_pat<PatFrag Store, PatFrag Value, InstHexagon MI>
  2662:   : Pat<(Store Value:$Ru, (add I32:$Rs, I32:$Rt)),
  2663:         (MI IntRegs:$Rs, IntRegs:$Rt, 0, Value:$Ru)>;
  2664: 
  2665: class Storea_pat<PatFrag Store, PatFrag Value, PatFrag Addr, InstHexagon MI>
  2666:   : Pat<(Store Value:$val, Addr:$addr), (MI Addr:$addr, Value:$val)>;
  2667: 
  2668: class Stoream_pat<PatFrag Store, PatFrag Value, PatFrag Addr, PatFrag ValueMod,
  2669:                   InstHexagon MI>
  2670:   : Pat<(Store Value:$val, Addr:$addr),
  2671:         (MI Addr:$addr, (ValueMod Value:$val))>;
  2672: 
  2673: def IMM_BYTE : SDNodeXForm<imm, [{
  2674:   // -1 can be represented as 255, etc.
  2675:   // assigning to a byte restores our desired signed value.
  2676:   int8_t imm = N->getSExtValue();
  2677:   return CurDAG->getSignedTargetConstant(imm, SDLoc(N), MVT::i32);
  2678: }]>;
  2679: 
  2680: def IMM_HALF : SDNodeXForm<imm, [{
  2681:   // -1 can be represented as 65535, etc.
  2682:   // assigning to a short restores our desired signed value.
  2683:   int16_t imm = N->getSExtValue();
  2684:   return CurDAG->getSignedTargetConstant(imm, SDLoc(N), MVT::i32);
  2685: }]>;
  2686: 
  2687: def IMM_WORD : SDNodeXForm<imm, [{
  2688:   // -1 can be represented as 4294967295, etc.
  2689:   // Currently, it's not doing this. But some optimization
  2690:   // might convert -1 to a large +ve number.
  2691:   // assigning to a word restores our desired signed value.
  2692:   int32_t imm = N->getSExtValue();
  2693:   return CurDAG->getSignedTargetConstant(imm, SDLoc(N), MVT::i32);
  2694: }]>;
  2695: 
  2696: def ToImmByte : OutPatFrag<(ops node:$R), (IMM_BYTE $R)>;
  2697: def ToImmHalf : OutPatFrag<(ops node:$R), (IMM_HALF $R)>;
  2698: def ToImmWord : OutPatFrag<(ops node:$R), (IMM_WORD $R)>;
  2699: 
  2700: // Even though the offset is not extendable in the store-immediate, we
  2701: // can still generate the fi# in the base address. If the final offset
  2702: // is not valid for the instruction, we will replace it with a scratch
  2703: // register.
  2704: class SmallStackStore<PatFrag Store>
  2705:   : PatFrag<(ops node:$Val, node:$Addr), (Store node:$Val, node:$Addr), [{
  2706:   return isSmallStackStore(cast<StoreSDNode>(N));
  2707: }]>;
  2708: 
  2709: // This is the complement of SmallStackStore.
  2710: class LargeStackStore<PatFrag Store>
  2711:   : PatFrag<(ops node:$Val, node:$Addr), (Store node:$Val, node:$Addr), [{
  2712:   return !isSmallStackStore(cast<StoreSDNode>(N));
  2713: }]>;
  2714: 
  2715: // Preferred addressing modes for various combinations of stored value
  2716: // and address computation.
  2717: // For stores where the address and value are both immediates, prefer
  2718: // store-immediate. The reason is that the constant-extender optimization
  2719: // can replace store-immediate with a store-register, but there is nothing
  2720: // to generate a store-immediate out of a store-register.
  2721: //
  2722: //         C     R     F    F+C   R+C   R+R   R<<S+C   R<<S+R
  2723: // --+-------+-----+-----+------+-----+-----+--------+--------
  2724: // C |   imm | imm | imm |  imm | imm |  rr |     ur |     rr
  2725: // R |  abs* |  io |  io |   io |  io |  rr |     ur |     rr
  2726: //
  2727: // (*) Absolute or GP-relative.
  2728: //
  2729: // Note that any expression can be matched by Reg. In particular, an immediate
  2730: // can always be placed in a register, so patterns checking for Imm should
  2731: // have a higher priority than the ones involving Reg that could also match.
  2732: // For example, *(p+4) could become r1=#4; memw(r0+r1<<#0) instead of the
  2733: // preferred memw(r0+#4). Similarly Reg+Imm or Reg+Reg should be tried before
  2734: // Reg alone.
  2735: //
  2736: // The order in which the different combinations are tried:
  2737: //
  2738: //         C     F     R    F+C   R+C   R+R   R<<S+C   R<<S+R
  2739: // --+-------+-----+-----+------+-----+-----+--------+--------
  2740: // C |     1 |   6 |   - |    5 |   9 |   - |      - |      -
  2741: // R |     2 |   8 |  12 |    7 |  10 |  11 |      3 |      4
  2742: 
  2743: 
  2744: // First, match the unusual case of doubleword store into Reg+Imm4, i.e.
  2745: // a store where the offset Imm4 is a multiple of 4, but not of 8. This
  2746: // implies that Reg is also a proper multiple of 4. To still generate a
  2747: // doubleword store, add 4 to Reg, and subtract 4 from the offset.
  2748: 
  2749: def s30_2ProperPred  : PatLeaf<(i32 imm), [{
  2750:   int64_t v = (int64_t)N->getSExtValue();
  2751:   return isShiftedInt<30,2>(v) && !isShiftedInt<29,3>(v);
  2752: }]>;
  2753: def RoundTo8 : SDNodeXForm<imm, [{
  2754:   int32_t Imm = N->getSExtValue();
  2755:   return CurDAG->getSignedTargetConstant(Imm & -8, SDLoc(N), MVT::i32);
  2756: }]>;
  2757: 
  2758: let AddedComplexity = 150 in
  2759: def: Pat<(store I64:$Ru, (add I32:$Rs, s30_2ProperPred:$Off)),
  2760:          (S2_storerd_io (A2_addi I32:$Rs, 4), (RoundTo8 $Off), I64:$Ru)>;
  2761: 
  2762: class Storexi_abs_pat<PatFrag Store, PatFrag Value, InstHexagon MI>
  2763:   : Pat<(Store Value:$val, anyimm:$addr),
  2764:         (MI (ToI32 $addr), 0, Value:$val)>;
  2765: class Storexim_abs_pat<PatFrag Store, PatFrag Value, PatFrag ValueMod,
  2766:                        InstHexagon MI>
  2767:   : Pat<(Store Value:$val, anyimm:$addr),
  2768:         (MI (ToI32 $addr), 0, (ValueMod Value:$val))>;
  2769: 
  2770: let AddedComplexity = 140 in {
  2771:   def: Storexim_abs_pat<truncstorei8,  anyint, ToImmByte, S4_storeirb_io>;
  2772:   def: Storexim_abs_pat<truncstorei16, anyint, ToImmHalf, S4_storeirh_io>;
  2773:   def: Storexim_abs_pat<store,         anyint, ToImmWord, S4_storeiri_io>;
  2774: 
  2775:   def: Storexi_abs_pat<truncstorei8,  anyimm, S4_storeirb_io>;
  2776:   def: Storexi_abs_pat<truncstorei16, anyimm, S4_storeirh_io>;
  2777:   def: Storexi_abs_pat<store,         anyimm, S4_storeiri_io>;
  2778: }
  2779: 
  2780: // GP-relative address
  2781: let AddedComplexity = 120 in {
  2782:   def: Storea_pat<truncstorei8,               I32, addrgp, S2_storerbgp>;
  2783:   def: Storea_pat<truncstorei16,              I32, addrgp, S2_storerhgp>;
  2784:   def: Storea_pat<store,                      I32, addrgp, S2_storerigp>;
  2785:   def: Storea_pat<store,                     V4I8, addrgp, S2_storerigp>;
  2786:   def: Storea_pat<store,                    V2I16, addrgp, S2_storerigp>;
  2787:   def: Storea_pat<store,                      I64, addrgp, S2_storerdgp>;
  2788:   def: Storea_pat<store,                     V8I8, addrgp, S2_storerdgp>;
  2789:   def: Storea_pat<store,                    V4I16, addrgp, S2_storerdgp>;
  2790:   def: Storea_pat<store,                    V2I32, addrgp, S2_storerdgp>;
  2791:   def: Storea_pat<store,                      F32, addrgp, S2_storerigp>;
  2792:   def: Storea_pat<store,                      F64, addrgp, S2_storerdgp>;
  2793:   def: Storea_pat<atomic_store_8,             I32, addrgp, S2_storerbgp>;
  2794:   def: Storea_pat<atomic_store_16,            I32, addrgp, S2_storerhgp>;
  2795:   def: Storea_pat<atomic_store_32,            I32, addrgp, S2_storerigp>;
  2796:   def: Storea_pat<atomic_store_32,           V4I8, addrgp, S2_storerigp>;
  2797:   def: Storea_pat<atomic_store_32,          V2I16, addrgp, S2_storerigp>;
  2798:   def: Storea_pat<atomic_store_64,            I64, addrgp, S2_storerdgp>;
  2799:   def: Storea_pat<atomic_store_64,           V8I8, addrgp, S2_storerdgp>;
  2800:   def: Storea_pat<atomic_store_64,          V4I16, addrgp, S2_storerdgp>;
```
- EN: It declares types such as Storexi_base_pat, Storexim_fi_pat, Storexim_base_pat, Storexu_shl_pat, ... (12 total), which carry the state or API of this component. It defines declarative TableGen records like Storexi_base_pat, Storexim_fi_pat, Storexim_fi_add_pat, Storexim_add_pat, Storexim_base_pat, ... (24 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getSExtValue, getSignedTargetConstant, isSmallStackStore, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 Storexi_base_pat, Storexim_fi_pat, Storexim_base_pat, Storexu_shl_pat, ... (12 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Storexi_base_pat, Storexim_fi_pat, Storexim_fi_add_pat, Storexim_add_pat, Storexim_base_pat, ... (24 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getSExtValue, getSignedTargetConstant, isSmallStackStore 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 2801-3000 / 第 2801-3000 行

```tablegen
  2801:   def: Storea_pat<atomic_store_64,          V2I32, addrgp, S2_storerdgp>;
  2802: 
  2803:   def: Stoream_pat<truncstorei8,  I64, addrgp, LoReg,    S2_storerbgp>;
  2804:   def: Stoream_pat<truncstorei16, I64, addrgp, LoReg,    S2_storerhgp>;
  2805:   def: Stoream_pat<truncstorei32, I64, addrgp, LoReg,    S2_storerigp>;
  2806:   def: Stoream_pat<store,         I1,  addrgp, I1toI32,  S2_storerbgp>;
  2807: }
  2808: 
  2809: // Absolute address
  2810: let AddedComplexity = 110 in {
  2811:   def: Storea_pat<truncstorei8,               I32, anyimm0, PS_storerbabs>;
  2812:   def: Storea_pat<truncstorei16,              I32, anyimm1, PS_storerhabs>;
  2813:   def: Storea_pat<store,                      I32, anyimm2, PS_storeriabs>;
  2814:   def: Storea_pat<store,                     V4I8, anyimm2, PS_storeriabs>;
  2815:   def: Storea_pat<store,                    V2I16, anyimm2, PS_storeriabs>;
  2816:   def: Storea_pat<store,                      I64, anyimm3, PS_storerdabs>;
  2817:   def: Storea_pat<store,                     V8I8, anyimm3, PS_storerdabs>;
  2818:   def: Storea_pat<store,                    V4I16, anyimm3, PS_storerdabs>;
  2819:   def: Storea_pat<store,                    V2I32, anyimm3, PS_storerdabs>;
  2820:   def: Storea_pat<store,                      F32, anyimm2, PS_storeriabs>;
  2821:   def: Storea_pat<store,                      F64, anyimm3, PS_storerdabs>;
  2822:   def: Storea_pat<atomic_store_8,             I32, anyimm0, PS_storerbabs>;
  2823:   def: Storea_pat<atomic_store_16,            I32, anyimm1, PS_storerhabs>;
  2824:   def: Storea_pat<atomic_store_32,            I32, anyimm2, PS_storeriabs>;
  2825:   def: Storea_pat<atomic_store_32,           V4I8, anyimm2, PS_storeriabs>;
  2826:   def: Storea_pat<atomic_store_32,          V2I16, anyimm2, PS_storeriabs>;
  2827:   def: Storea_pat<atomic_store_64,            I64, anyimm3, PS_storerdabs>;
  2828:   def: Storea_pat<atomic_store_64,           V8I8, anyimm3, PS_storerdabs>;
  2829:   def: Storea_pat<atomic_store_64,          V4I16, anyimm3, PS_storerdabs>;
  2830:   def: Storea_pat<atomic_store_64,          V2I32, anyimm3, PS_storerdabs>;
  2831: 
  2832:   def: Stoream_pat<truncstorei8,  I64, anyimm0, LoReg,    PS_storerbabs>;
  2833:   def: Stoream_pat<truncstorei16, I64, anyimm1, LoReg,    PS_storerhabs>;
  2834:   def: Stoream_pat<truncstorei32, I64, anyimm2, LoReg,    PS_storeriabs>;
  2835:   def: Stoream_pat<store,         I1,  anyimm0, I1toI32,  PS_storerbabs>;
  2836: }
  2837: 
  2838: // Reg<<S + Imm
  2839: let AddedComplexity = 100 in {
  2840:   def: Storexu_shl_pat<truncstorei8,    I32, anyimm0, S4_storerb_ur>;
  2841:   def: Storexu_shl_pat<truncstorei16,   I32, anyimm1, S4_storerh_ur>;
  2842:   def: Storexu_shl_pat<store,           I32, anyimm2, S4_storeri_ur>;
  2843:   def: Storexu_shl_pat<store,          V4I8, anyimm2, S4_storeri_ur>;
  2844:   def: Storexu_shl_pat<store,         V2I16, anyimm2, S4_storeri_ur>;
  2845:   def: Storexu_shl_pat<store,           I64, anyimm3, S4_storerd_ur>;
  2846:   def: Storexu_shl_pat<store,          V8I8, anyimm3, S4_storerd_ur>;
  2847:   def: Storexu_shl_pat<store,         V4I16, anyimm3, S4_storerd_ur>;
  2848:   def: Storexu_shl_pat<store,         V2I32, anyimm3, S4_storerd_ur>;
  2849:   def: Storexu_shl_pat<store,           F32, anyimm2, S4_storeri_ur>;
  2850:   def: Storexu_shl_pat<store,           F64, anyimm3, S4_storerd_ur>;
  2851: 
  2852:   def: Pat<(store I1:$Pu, (add (shl I32:$Rs, u2_0ImmPred:$u2), anyimm:$A)),
  2853:            (S4_storerb_ur IntRegs:$Rs, imm:$u2, imm:$A, (I1toI32 I1:$Pu))>;
  2854: }
  2855: 
  2856: // Reg<<S + Reg
  2857: let AddedComplexity = 90 in {
  2858:   def: Storexr_shl_pat<truncstorei8,    I32, S4_storerb_rr>;
  2859:   def: Storexr_shl_pat<truncstorei16,   I32, S4_storerh_rr>;
  2860:   def: Storexr_shl_pat<store,           I32, S4_storeri_rr>;
  2861:   def: Storexr_shl_pat<store,          V4I8, S4_storeri_rr>;
  2862:   def: Storexr_shl_pat<store,         V2I16, S4_storeri_rr>;
  2863:   def: Storexr_shl_pat<store,           I64, S4_storerd_rr>;
  2864:   def: Storexr_shl_pat<store,          V8I8, S4_storerd_rr>;
  2865:   def: Storexr_shl_pat<store,         V4I16, S4_storerd_rr>;
  2866:   def: Storexr_shl_pat<store,         V2I32, S4_storerd_rr>;
  2867:   def: Storexr_shl_pat<store,           F32, S4_storeri_rr>;
  2868:   def: Storexr_shl_pat<store,           F64, S4_storerd_rr>;
  2869: 
  2870:   def: Pat<(store I1:$Pu, (add (shl I32:$Rs, u2_0ImmPred:$u2), I32:$Rt)),
  2871:            (S4_storerb_rr IntRegs:$Rt, IntRegs:$Rs, imm:$u2, (I1toI32 I1:$Pu))>;
  2872: }
  2873: 
  2874: class SS_<PatFrag F> : SmallStackStore<F>;
  2875: class LS_<PatFrag F> : LargeStackStore<F>;
  2876: 
  2877: multiclass IMFA_<PatFrag S, PatFrag V, PatFrag O, PatFrag M, InstHexagon I> {
  2878:   defm: Storexim_fi_add_pat<S, V, O, M, I>;
  2879: }
  2880: multiclass IFA_<PatFrag S, PatFrag V, PatFrag O, InstHexagon I> {
  2881:   defm: Storexi_fi_add_pat<S, V, O, I>;
  2882: }
  2883: 
  2884: // Fi+Imm, store-immediate
  2885: let AddedComplexity = 80 in {
  2886:   defm: IMFA_<SS_<truncstorei8>,  anyint, u6_0ImmPred, ToImmByte, S4_storeirb_io>;
  2887:   defm: IMFA_<SS_<truncstorei16>, anyint, u6_1ImmPred, ToImmHalf, S4_storeirh_io>;
  2888:   defm: IMFA_<SS_<store>,         anyint, u6_2ImmPred, ToImmWord, S4_storeiri_io>;
  2889: 
  2890:   defm: IFA_<SS_<truncstorei8>,   anyimm, u6_0ImmPred, S4_storeirb_io>;
  2891:   defm: IFA_<SS_<truncstorei16>,  anyimm, u6_1ImmPred, S4_storeirh_io>;
  2892:   defm: IFA_<SS_<store>,          anyimm, u6_2ImmPred, S4_storeiri_io>;
  2893: 
  2894:   // For large-stack stores, generate store-register (prefer explicit Fi
  2895:   // in the address).
  2896:   defm: IMFA_<LS_<truncstorei8>,   anyimm, u6_0ImmPred, ToI32, S2_storerb_io>;
  2897:   defm: IMFA_<LS_<truncstorei16>,  anyimm, u6_1ImmPred, ToI32, S2_storerh_io>;
  2898:   defm: IMFA_<LS_<store>,          anyimm, u6_2ImmPred, ToI32, S2_storeri_io>;
  2899: }
  2900: 
  2901: // Fi, store-immediate
  2902: let AddedComplexity = 70 in {
  2903:   def: Storexim_fi_pat<SS_<truncstorei8>,  anyint, ToImmByte, S4_storeirb_io>;
  2904:   def: Storexim_fi_pat<SS_<truncstorei16>, anyint, ToImmHalf, S4_storeirh_io>;
  2905:   def: Storexim_fi_pat<SS_<store>,         anyint, ToImmWord, S4_storeiri_io>;
  2906: 
  2907:   def: Storexi_fi_pat<SS_<truncstorei8>,   anyimm, S4_storeirb_io>;
  2908:   def: Storexi_fi_pat<SS_<truncstorei16>,  anyimm, S4_storeirh_io>;
  2909:   def: Storexi_fi_pat<SS_<store>,          anyimm, S4_storeiri_io>;
  2910: 
  2911:   // For large-stack stores, generate store-register (prefer explicit Fi
  2912:   // in the address).
  2913:   def: Storexim_fi_pat<LS_<truncstorei8>,  anyimm, ToI32, S2_storerb_io>;
  2914:   def: Storexim_fi_pat<LS_<truncstorei16>, anyimm, ToI32, S2_storerh_io>;
  2915:   def: Storexim_fi_pat<LS_<store>,         anyimm, ToI32, S2_storeri_io>;
  2916: }
  2917: 
  2918: // Fi+Imm, Fi, store-register
  2919: let AddedComplexity = 60 in {
  2920:   defm: Storexi_fi_add_pat<truncstorei8,    I32, anyimm, S2_storerb_io>;
  2921:   defm: Storexi_fi_add_pat<truncstorei16,   I32, anyimm, S2_storerh_io>;
  2922:   defm: Storexi_fi_add_pat<store,           I32, anyimm, S2_storeri_io>;
  2923:   defm: Storexi_fi_add_pat<store,          V4I8, anyimm, S2_storeri_io>;
  2924:   defm: Storexi_fi_add_pat<store,         V2I16, anyimm, S2_storeri_io>;
  2925:   defm: Storexi_fi_add_pat<store,           I64, anyimm, S2_storerd_io>;
  2926:   defm: Storexi_fi_add_pat<store,          V8I8, anyimm, S2_storerd_io>;
  2927:   defm: Storexi_fi_add_pat<store,         V4I16, anyimm, S2_storerd_io>;
  2928:   defm: Storexi_fi_add_pat<store,         V2I32, anyimm, S2_storerd_io>;
  2929:   defm: Storexi_fi_add_pat<store,           F32, anyimm, S2_storeri_io>;
  2930:   defm: Storexi_fi_add_pat<store,           F64, anyimm, S2_storerd_io>;
  2931:   defm: Storexim_fi_add_pat<store, I1, anyimm, I1toI32, S2_storerb_io>;
  2932: 
  2933:   def: Storexi_fi_pat<truncstorei8,     I32, S2_storerb_io>;
  2934:   def: Storexi_fi_pat<truncstorei16,    I32, S2_storerh_io>;
  2935:   def: Storexi_fi_pat<store,            I32, S2_storeri_io>;
  2936:   def: Storexi_fi_pat<store,           V4I8, S2_storeri_io>;
  2937:   def: Storexi_fi_pat<store,          V2I16, S2_storeri_io>;
  2938:   def: Storexi_fi_pat<store,            I64, S2_storerd_io>;
  2939:   def: Storexi_fi_pat<store,           V8I8, S2_storerd_io>;
  2940:   def: Storexi_fi_pat<store,          V4I16, S2_storerd_io>;
  2941:   def: Storexi_fi_pat<store,          V2I32, S2_storerd_io>;
  2942:   def: Storexi_fi_pat<store,            F32, S2_storeri_io>;
  2943:   def: Storexi_fi_pat<store,            F64, S2_storerd_io>;
  2944:   def: Storexim_fi_pat<store, I1, I1toI32, S2_storerb_io>;
  2945: }
  2946: 
  2947: 
  2948: multiclass IMRA_<PatFrag S, PatFrag V, PatFrag O, PatFrag M, InstHexagon I> {
  2949:   defm: Storexim_add_pat<S, V, O, M, I>;
  2950: }
  2951: multiclass IRA_<PatFrag S, PatFrag V, PatFrag O, InstHexagon I> {
  2952:   defm: Storexi_add_pat<S, V, O, I>;
  2953: }
  2954: 
  2955: // Reg+Imm, store-immediate
  2956: let AddedComplexity = 50 in {
  2957:   defm: IMRA_<truncstorei8,   anyint, u6_0ImmPred, ToImmByte, S4_storeirb_io>;
  2958:   defm: IMRA_<truncstorei16,  anyint, u6_1ImmPred, ToImmHalf, S4_storeirh_io>;
  2959:   defm: IMRA_<store,          anyint, u6_2ImmPred, ToImmWord, S4_storeiri_io>;
  2960: 
  2961:   defm: IRA_<truncstorei8,    anyimm, u6_0ImmPred, S4_storeirb_io>;
  2962:   defm: IRA_<truncstorei16,   anyimm, u6_1ImmPred, S4_storeirh_io>;
  2963:   defm: IRA_<store,           anyimm, u6_2ImmPred, S4_storeiri_io>;
  2964: }
  2965: 
  2966: // Reg+Imm, store-register
  2967: let AddedComplexity = 40 in {
  2968:   defm: Storexi_pat<truncstorei8,     I32, anyimm0, S2_storerb_io>;
  2969:   defm: Storexi_pat<truncstorei16,    I32, anyimm1, S2_storerh_io>;
  2970:   defm: Storexi_pat<store,            I32, anyimm2, S2_storeri_io>;
  2971:   defm: Storexi_pat<store,           V4I8, anyimm2, S2_storeri_io>;
  2972:   defm: Storexi_pat<store,          V2I16, anyimm2, S2_storeri_io>;
  2973:   defm: Storexi_pat<store,            I64, anyimm3, S2_storerd_io>;
  2974:   defm: Storexi_pat<store,           V8I8, anyimm3, S2_storerd_io>;
  2975:   defm: Storexi_pat<store,          V4I16, anyimm3, S2_storerd_io>;
  2976:   defm: Storexi_pat<store,          V2I32, anyimm3, S2_storerd_io>;
  2977:   defm: Storexi_pat<store,            F32, anyimm2, S2_storeri_io>;
  2978:   defm: Storexi_pat<store,            F64, anyimm3, S2_storerd_io>;
  2979: 
  2980:   defm: Storexim_pat<truncstorei8,  I64, anyimm0, LoReg,   S2_storerb_io>;
  2981:   defm: Storexim_pat<truncstorei16, I64, anyimm1, LoReg,   S2_storerh_io>;
  2982:   defm: Storexim_pat<truncstorei32, I64, anyimm2, LoReg,   S2_storeri_io>;
  2983:   defm: Storexim_pat<store,         I1,  anyimm0, I1toI32, S2_storerb_io>;
  2984: 
  2985:   defm: Storexi_pat<atomic_store_8,     I32, anyimm0, S2_storerb_io>;
  2986:   defm: Storexi_pat<atomic_store_16,    I32, anyimm1, S2_storerh_io>;
  2987:   defm: Storexi_pat<atomic_store_32,    I32, anyimm2, S2_storeri_io>;
  2988:   defm: Storexi_pat<atomic_store_32,   V4I8, anyimm2, S2_storeri_io>;
  2989:   defm: Storexi_pat<atomic_store_32,  V2I16, anyimm2, S2_storeri_io>;
  2990:   defm: Storexi_pat<atomic_store_64,    I64, anyimm3, S2_storerd_io>;
  2991:   defm: Storexi_pat<atomic_store_64,   V8I8, anyimm3, S2_storerd_io>;
  2992:   defm: Storexi_pat<atomic_store_64,  V4I16, anyimm3, S2_storerd_io>;
  2993:   defm: Storexi_pat<atomic_store_64,  V2I32, anyimm3, S2_storerd_io>;
  2994: }
  2995: 
  2996: // Reg+Reg
  2997: let AddedComplexity = 30 in {
  2998:   def: Storexr_add_pat<truncstorei8,    I32, S4_storerb_rr>;
  2999:   def: Storexr_add_pat<truncstorei16,   I32, S4_storerh_rr>;
  3000:   def: Storexr_add_pat<store,           I32, S4_storeri_rr>;
```
- EN: It declares types such as SS_, LS_, which carry the state or API of this component. It defines declarative TableGen records like SS_, LS_, IMFA_, IFA_, IMRA_, ... (6 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 SS_, LS_ 等类型，用来承载该组件的状态或接口。 这里定义了 SS_, LS_, IMFA_, IFA_, IMRA_, ... (6 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 3001-3200 / 第 3001-3200 行

```tablegen
  3001:   def: Storexr_add_pat<store,          V4I8, S4_storeri_rr>;
  3002:   def: Storexr_add_pat<store,         V2I16, S4_storeri_rr>;
  3003:   def: Storexr_add_pat<store,           I64, S4_storerd_rr>;
  3004:   def: Storexr_add_pat<store,          V8I8, S4_storerd_rr>;
  3005:   def: Storexr_add_pat<store,         V4I16, S4_storerd_rr>;
  3006:   def: Storexr_add_pat<store,         V2I32, S4_storerd_rr>;
  3007:   def: Storexr_add_pat<store,           F32, S4_storeri_rr>;
  3008:   def: Storexr_add_pat<store,           F64, S4_storerd_rr>;
  3009: 
  3010:   def: Pat<(store I1:$Pu, (add I32:$Rs, I32:$Rt)),
  3011:            (S4_storerb_rr IntRegs:$Rs, IntRegs:$Rt, 0, (I1toI32 I1:$Pu))>;
  3012: }
  3013: 
  3014: // Reg, store-immediate
  3015: let AddedComplexity = 20 in {
  3016:   def: Storexim_base_pat<truncstorei8,  anyint, ToImmByte, S4_storeirb_io>;
  3017:   def: Storexim_base_pat<truncstorei16, anyint, ToImmHalf, S4_storeirh_io>;
  3018:   def: Storexim_base_pat<store,         anyint, ToImmWord, S4_storeiri_io>;
  3019: 
  3020:   def: Storexi_base_pat<truncstorei8,   anyimm, S4_storeirb_io>;
  3021:   def: Storexi_base_pat<truncstorei16,  anyimm, S4_storeirh_io>;
  3022:   def: Storexi_base_pat<store,          anyimm, S4_storeiri_io>;
  3023: }
  3024: 
  3025: // Reg, store-register
  3026: let AddedComplexity = 10 in {
  3027:   def: Storexi_base_pat<truncstorei8,     I32, S2_storerb_io>;
  3028:   def: Storexi_base_pat<truncstorei16,    I32, S2_storerh_io>;
  3029:   def: Storexi_base_pat<store,            I32, S2_storeri_io>;
  3030:   def: Storexi_base_pat<store,           V4I8, S2_storeri_io>;
  3031:   def: Storexi_base_pat<store,          V2I16, S2_storeri_io>;
  3032:   def: Storexi_base_pat<store,            I64, S2_storerd_io>;
  3033:   def: Storexi_base_pat<store,           V8I8, S2_storerd_io>;
  3034:   def: Storexi_base_pat<store,          V4I16, S2_storerd_io>;
  3035:   def: Storexi_base_pat<store,          V2I32, S2_storerd_io>;
  3036:   def: Storexi_base_pat<store,            F32, S2_storeri_io>;
  3037:   def: Storexi_base_pat<store,            F64, S2_storerd_io>;
  3038: 
  3039:   def: Storexim_base_pat<truncstorei8,  I64, LoReg,   S2_storerb_io>;
  3040:   def: Storexim_base_pat<truncstorei16, I64, LoReg,   S2_storerh_io>;
  3041:   def: Storexim_base_pat<truncstorei32, I64, LoReg,   S2_storeri_io>;
  3042:   def: Storexim_base_pat<store,         I1,  I1toI32, S2_storerb_io>;
  3043: 
  3044:   def: Storexi_base_pat<atomic_store_8,     I32, S2_storerb_io>;
  3045:   def: Storexi_base_pat<atomic_store_16,    I32, S2_storerh_io>;
  3046:   def: Storexi_base_pat<atomic_store_32,    I32, S2_storeri_io>;
  3047:   def: Storexi_base_pat<atomic_store_32,   V4I8, S2_storeri_io>;
  3048:   def: Storexi_base_pat<atomic_store_32,  V2I16, S2_storeri_io>;
  3049:   def: Storexi_base_pat<atomic_store_64,    I64, S2_storerd_io>;
  3050:   def: Storexi_base_pat<atomic_store_64,   V8I8, S2_storerd_io>;
  3051:   def: Storexi_base_pat<atomic_store_64,  V4I16, S2_storerd_io>;
  3052:   def: Storexi_base_pat<atomic_store_64,  V2I32, S2_storerd_io>;
  3053: }
  3054: 
  3055: 
  3056: // --(14) Memop ----------------------------------------------------------
  3057: //
  3058: 
  3059: def m5_0Imm8Pred : PatLeaf<(i32 imm), [{
  3060:   int8_t V = N->getSExtValue();
  3061:   return -32 < V && V <= -1;
  3062: }]>;
  3063: 
  3064: def m5_0Imm16Pred : PatLeaf<(i32 imm), [{
  3065:   int16_t V = N->getSExtValue();
  3066:   return -32 < V && V <= -1;
  3067: }]>;
  3068: 
  3069: def m5_0ImmPred  : PatLeaf<(i32 imm), [{
  3070:   int64_t V = N->getSExtValue();
  3071:   return -31 <= V && V <= -1;
  3072: }]>;
  3073: 
  3074: def IsNPow2_8 : PatLeaf<(i32 imm), [{
  3075:   uint8_t NV = ~N->getZExtValue();
  3076:   return isPowerOf2_32(NV);
  3077: }]>;
  3078: 
  3079: def IsNPow2_16 : PatLeaf<(i32 imm), [{
  3080:   uint16_t NV = ~N->getZExtValue();
  3081:   return isPowerOf2_32(NV);
  3082: }]>;
  3083: 
  3084: def Log2_8 : SDNodeXForm<imm, [{
  3085:   uint8_t V = N->getZExtValue();
  3086:   return CurDAG->getTargetConstant(Log2_32(V), SDLoc(N), MVT::i32);
  3087: }]>;
  3088: 
  3089: def Log2_16 : SDNodeXForm<imm, [{
  3090:   uint16_t V = N->getZExtValue();
  3091:   return CurDAG->getTargetConstant(Log2_32(V), SDLoc(N), MVT::i32);
  3092: }]>;
  3093: 
  3094: def LogN2_8 : SDNodeXForm<imm, [{
  3095:   uint8_t NV = ~N->getZExtValue();
  3096:   return CurDAG->getTargetConstant(Log2_32(NV), SDLoc(N), MVT::i32);
  3097: }]>;
  3098: 
  3099: def LogN2_16 : SDNodeXForm<imm, [{
  3100:   uint16_t NV = ~N->getZExtValue();
  3101:   return CurDAG->getTargetConstant(Log2_32(NV), SDLoc(N), MVT::i32);
  3102: }]>;
  3103: 
  3104: def IdImm : SDNodeXForm<imm, [{ return SDValue(N, 0); }]>;
  3105: 
  3106: multiclass Memopxr_base_pat<PatFrag Load, PatFrag Store, SDNode Oper,
  3107:                             InstHexagon MI> {
  3108:   // Addr: i32
  3109:   def: Pat<(Store (Oper (Load I32:$Rs), I32:$A), I32:$Rs),
  3110:            (MI I32:$Rs, 0, I32:$A)>;
  3111:   // Addr: fi
  3112:   def: Pat<(Store (Oper (Load AddrFI:$Rs), I32:$A), AddrFI:$Rs),
  3113:            (MI AddrFI:$Rs, 0, I32:$A)>;
  3114: }
  3115: 
  3116: multiclass Memopxr_add_pat<PatFrag Load, PatFrag Store, PatFrag ImmPred,
  3117:                            SDNode Oper, InstHexagon MI> {
  3118:   // Addr: i32
  3119:   def: Pat<(Store (Oper (Load (add I32:$Rs, ImmPred:$Off)), I32:$A),
  3120:                   (add I32:$Rs, ImmPred:$Off)),
  3121:            (MI I32:$Rs, imm:$Off, I32:$A)>;
  3122:   def: Pat<(Store (Oper (Load (IsOrAdd I32:$Rs, ImmPred:$Off)), I32:$A),
  3123:                   (IsOrAdd I32:$Rs, ImmPred:$Off)),
  3124:            (MI I32:$Rs, imm:$Off, I32:$A)>;
  3125:   // Addr: fi
  3126:   def: Pat<(Store (Oper (Load (add AddrFI:$Rs, ImmPred:$Off)), I32:$A),
  3127:                   (add AddrFI:$Rs, ImmPred:$Off)),
  3128:            (MI AddrFI:$Rs, imm:$Off, I32:$A)>;
  3129:   def: Pat<(Store (Oper (Load (IsOrAdd AddrFI:$Rs, ImmPred:$Off)), I32:$A),
  3130:                   (IsOrAdd AddrFI:$Rs, ImmPred:$Off)),
  3131:            (MI AddrFI:$Rs, imm:$Off, I32:$A)>;
  3132: }
  3133: 
  3134: multiclass Memopxr_pat<PatFrag Load, PatFrag Store, PatFrag ImmPred,
  3135:                        SDNode Oper, InstHexagon MI> {
  3136:   let Predicates = [UseMEMOPS] in {
  3137:     defm: Memopxr_base_pat <Load, Store,          Oper, MI>;
  3138:     defm: Memopxr_add_pat  <Load, Store, ImmPred, Oper, MI>;
  3139:   }
  3140: }
  3141: 
  3142: let AddedComplexity = 200 in {
  3143:   // add reg
  3144:   defm: Memopxr_pat<extloadi8, truncstorei8, u6_0ImmPred, add,
  3145:         /*anyext*/  L4_add_memopb_io>;
  3146:   defm: Memopxr_pat<sextloadi8, truncstorei8, u6_0ImmPred, add,
  3147:         /*sext*/    L4_add_memopb_io>;
  3148:   defm: Memopxr_pat<zextloadi8, truncstorei8, u6_0ImmPred, add,
  3149:         /*zext*/    L4_add_memopb_io>;
  3150:   defm: Memopxr_pat<extloadi16, truncstorei16, u6_1ImmPred, add,
  3151:         /*anyext*/  L4_add_memoph_io>;
  3152:   defm: Memopxr_pat<sextloadi16, truncstorei16, u6_1ImmPred, add,
  3153:         /*sext*/    L4_add_memoph_io>;
  3154:   defm: Memopxr_pat<zextloadi16, truncstorei16, u6_1ImmPred, add,
  3155:         /*zext*/    L4_add_memoph_io>;
  3156:   defm: Memopxr_pat<load, store, u6_2ImmPred, add, L4_add_memopw_io>;
  3157: 
  3158:   // sub reg
  3159:   defm: Memopxr_pat<extloadi8, truncstorei8, u6_0ImmPred, sub,
  3160:         /*anyext*/  L4_sub_memopb_io>;
  3161:   defm: Memopxr_pat<sextloadi8, truncstorei8, u6_0ImmPred, sub,
  3162:         /*sext*/    L4_sub_memopb_io>;
  3163:   defm: Memopxr_pat<zextloadi8, truncstorei8, u6_0ImmPred, sub,
  3164:         /*zext*/    L4_sub_memopb_io>;
  3165:   defm: Memopxr_pat<extloadi16, truncstorei16, u6_1ImmPred, sub,
  3166:         /*anyext*/  L4_sub_memoph_io>;
  3167:   defm: Memopxr_pat<sextloadi16, truncstorei16, u6_1ImmPred, sub,
  3168:         /*sext*/    L4_sub_memoph_io>;
  3169:   defm: Memopxr_pat<zextloadi16, truncstorei16, u6_1ImmPred, sub,
  3170:         /*zext*/    L4_sub_memoph_io>;
  3171:   defm: Memopxr_pat<load, store, u6_2ImmPred, sub, L4_sub_memopw_io>;
  3172: 
  3173:   // and reg
  3174:   defm: Memopxr_pat<extloadi8, truncstorei8, u6_0ImmPred, and,
  3175:         /*anyext*/  L4_and_memopb_io>;
  3176:   defm: Memopxr_pat<sextloadi8, truncstorei8, u6_0ImmPred, and,
  3177:         /*sext*/    L4_and_memopb_io>;
  3178:   defm: Memopxr_pat<zextloadi8, truncstorei8, u6_0ImmPred, and,
  3179:         /*zext*/    L4_and_memopb_io>;
  3180:   defm: Memopxr_pat<extloadi16, truncstorei16, u6_1ImmPred, and,
  3181:         /*anyext*/  L4_and_memoph_io>;
  3182:   defm: Memopxr_pat<sextloadi16, truncstorei16, u6_1ImmPred, and,
  3183:         /*sext*/    L4_and_memoph_io>;
  3184:   defm: Memopxr_pat<zextloadi16, truncstorei16, u6_1ImmPred, and,
  3185:         /*zext*/    L4_and_memoph_io>;
  3186:   defm: Memopxr_pat<load, store, u6_2ImmPred, and, L4_and_memopw_io>;
  3187: 
  3188:   // or reg
  3189:   defm: Memopxr_pat<extloadi8, truncstorei8, u6_0ImmPred, or,
  3190:         /*anyext*/  L4_or_memopb_io>;
  3191:   defm: Memopxr_pat<sextloadi8, truncstorei8, u6_0ImmPred, or,
  3192:         /*sext*/    L4_or_memopb_io>;
  3193:   defm: Memopxr_pat<zextloadi8, truncstorei8, u6_0ImmPred, or,
  3194:         /*zext*/    L4_or_memopb_io>;
  3195:   defm: Memopxr_pat<extloadi16, truncstorei16, u6_1ImmPred, or,
  3196:         /*anyext*/  L4_or_memoph_io>;
  3197:   defm: Memopxr_pat<sextloadi16, truncstorei16, u6_1ImmPred, or,
  3198:         /*sext*/    L4_or_memoph_io>;
  3199:   defm: Memopxr_pat<zextloadi16, truncstorei16, u6_1ImmPred, or,
  3200:         /*zext*/    L4_or_memoph_io>;
```
- EN: It defines declarative TableGen records like m5_0Imm8Pred, m5_0Imm16Pred, m5_0ImmPred, IsNPow2_8, IsNPow2_16, ... (13 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getSExtValue, getZExtValue, isPowerOf2_32, getTargetConstant, ... (5 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里定义了 m5_0Imm8Pred, m5_0Imm16Pred, m5_0ImmPred, IsNPow2_8, IsNPow2_16, ... (13 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getSExtValue, getZExtValue, isPowerOf2_32, getTargetConstant, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 3201-3400 / 第 3201-3400 行

```tablegen
  3201:   defm: Memopxr_pat<load, store, u6_2ImmPred, or, L4_or_memopw_io>;
  3202: }
  3203: 
  3204: 
  3205: multiclass Memopxi_base_pat<PatFrag Load, PatFrag Store, SDNode Oper,
  3206:                             PatFrag Arg, SDNodeXForm ArgMod, InstHexagon MI> {
  3207:   // Addr: i32
  3208:   def: Pat<(Store (Oper (Load I32:$Rs), Arg:$A), I32:$Rs),
  3209:            (MI I32:$Rs, 0, (ArgMod Arg:$A))>;
  3210:   // Addr: fi
  3211:   def: Pat<(Store (Oper (Load AddrFI:$Rs), Arg:$A), AddrFI:$Rs),
  3212:            (MI AddrFI:$Rs, 0, (ArgMod Arg:$A))>;
  3213: }
  3214: 
  3215: multiclass Memopxi_add_pat<PatFrag Load, PatFrag Store, PatFrag ImmPred,
  3216:                            SDNode Oper, PatFrag Arg, SDNodeXForm ArgMod,
  3217:                            InstHexagon MI> {
  3218:   // Addr: i32
  3219:   def: Pat<(Store (Oper (Load (add I32:$Rs, ImmPred:$Off)), Arg:$A),
  3220:                   (add I32:$Rs, ImmPred:$Off)),
  3221:            (MI I32:$Rs, imm:$Off, (ArgMod Arg:$A))>;
  3222:   def: Pat<(Store (Oper (Load (IsOrAdd I32:$Rs, ImmPred:$Off)), Arg:$A),
  3223:                   (IsOrAdd I32:$Rs, ImmPred:$Off)),
  3224:            (MI I32:$Rs, imm:$Off, (ArgMod Arg:$A))>;
  3225:   // Addr: fi
  3226:   def: Pat<(Store (Oper (Load (add AddrFI:$Rs, ImmPred:$Off)), Arg:$A),
  3227:                   (add AddrFI:$Rs, ImmPred:$Off)),
  3228:            (MI AddrFI:$Rs, imm:$Off, (ArgMod Arg:$A))>;
  3229:   def: Pat<(Store (Oper (Load (IsOrAdd AddrFI:$Rs, ImmPred:$Off)), Arg:$A),
  3230:                   (IsOrAdd AddrFI:$Rs, ImmPred:$Off)),
  3231:            (MI AddrFI:$Rs, imm:$Off, (ArgMod Arg:$A))>;
  3232: }
  3233: 
  3234: multiclass Memopxi_pat<PatFrag Load, PatFrag Store, PatFrag ImmPred,
  3235:                        SDNode Oper, PatFrag Arg, SDNodeXForm ArgMod,
  3236:                        InstHexagon MI> {
  3237:   let Predicates = [UseMEMOPS] in {
  3238:     defm: Memopxi_base_pat <Load, Store,          Oper, Arg, ArgMod, MI>;
  3239:     defm: Memopxi_add_pat  <Load, Store, ImmPred, Oper, Arg, ArgMod, MI>;
  3240:   }
  3241: }
  3242: 
  3243: let AddedComplexity = 220 in {
  3244:   // add imm
  3245:   defm: Memopxi_pat<extloadi8, truncstorei8, u6_0ImmPred, add, u5_0ImmPred,
  3246:         /*anyext*/  IdImm, L4_iadd_memopb_io>;
  3247:   defm: Memopxi_pat<sextloadi8, truncstorei8, u6_0ImmPred, add, u5_0ImmPred,
  3248:         /*sext*/    IdImm, L4_iadd_memopb_io>;
  3249:   defm: Memopxi_pat<zextloadi8, truncstorei8, u6_0ImmPred, add, u5_0ImmPred,
  3250:         /*zext*/    IdImm, L4_iadd_memopb_io>;
  3251:   defm: Memopxi_pat<extloadi16, truncstorei16, u6_1ImmPred, add, u5_0ImmPred,
  3252:         /*anyext*/  IdImm, L4_iadd_memoph_io>;
  3253:   defm: Memopxi_pat<extloadi16, truncstorei16, u6_1ImmPred, add, u5_0ImmPred,
  3254:         /*sext*/    IdImm, L4_iadd_memoph_io>;
  3255:   defm: Memopxi_pat<extloadi16, truncstorei16, u6_1ImmPred, add, u5_0ImmPred,
  3256:         /*zext*/    IdImm, L4_iadd_memoph_io>;
  3257:   defm: Memopxi_pat<load, store, u6_2ImmPred, add, u5_0ImmPred, IdImm,
  3258:                     L4_iadd_memopw_io>;
  3259:   defm: Memopxi_pat<extloadi8, truncstorei8, u6_0ImmPred, sub, m5_0Imm8Pred,
  3260:         /*anyext*/  NegImm8, L4_iadd_memopb_io>;
  3261:   defm: Memopxi_pat<sextloadi8, truncstorei8, u6_0ImmPred, sub, m5_0Imm8Pred,
  3262:         /*sext*/    NegImm8, L4_iadd_memopb_io>;
  3263:   defm: Memopxi_pat<zextloadi8, truncstorei8, u6_0ImmPred, sub, m5_0Imm8Pred,
  3264:         /*zext*/    NegImm8, L4_iadd_memopb_io>;
  3265:   defm: Memopxi_pat<extloadi16, truncstorei16, u6_1ImmPred, sub, m5_0Imm16Pred,
  3266:         /*anyext*/  NegImm16, L4_iadd_memoph_io>;
  3267:   defm: Memopxi_pat<sextloadi16, truncstorei16, u6_1ImmPred, sub, m5_0Imm16Pred,
  3268:         /*sext*/    NegImm16, L4_iadd_memoph_io>;
  3269:   defm: Memopxi_pat<zextloadi16, truncstorei16, u6_1ImmPred, sub, m5_0Imm16Pred,
  3270:         /*zext*/    NegImm16, L4_iadd_memoph_io>;
  3271:   defm: Memopxi_pat<load, store, u6_2ImmPred, sub, m5_0ImmPred, NegImm32,
  3272:                     L4_iadd_memopw_io>;
  3273: 
  3274:   // sub imm
  3275:   defm: Memopxi_pat<extloadi8, truncstorei8, u6_0ImmPred, sub, u5_0ImmPred,
  3276:         /*anyext*/  IdImm, L4_isub_memopb_io>;
  3277:   defm: Memopxi_pat<sextloadi8, truncstorei8, u6_0ImmPred, sub, u5_0ImmPred,
  3278:         /*sext*/    IdImm, L4_isub_memopb_io>;
  3279:   defm: Memopxi_pat<zextloadi8, truncstorei8, u6_0ImmPred, sub, u5_0ImmPred,
  3280:         /*zext*/    IdImm, L4_isub_memopb_io>;
  3281:   defm: Memopxi_pat<extloadi16, truncstorei16, u6_1ImmPred, sub, u5_0ImmPred,
  3282:         /*anyext*/  IdImm, L4_isub_memoph_io>;
  3283:   defm: Memopxi_pat<sextloadi16, truncstorei16, u6_1ImmPred, sub, u5_0ImmPred,
  3284:         /*sext*/    IdImm, L4_isub_memoph_io>;
  3285:   defm: Memopxi_pat<zextloadi16, truncstorei16, u6_1ImmPred, sub, u5_0ImmPred,
  3286:         /*zext*/    IdImm, L4_isub_memoph_io>;
  3287:   defm: Memopxi_pat<load, store, u6_2ImmPred, sub, u5_0ImmPred, IdImm,
  3288:                     L4_isub_memopw_io>;
  3289:   defm: Memopxi_pat<extloadi8, truncstorei8, u6_0ImmPred, add, m5_0Imm8Pred,
  3290:         /*anyext*/  NegImm8, L4_isub_memopb_io>;
  3291:   defm: Memopxi_pat<sextloadi8, truncstorei8, u6_0ImmPred, add, m5_0Imm8Pred,
  3292:         /*sext*/    NegImm8, L4_isub_memopb_io>;
  3293:   defm: Memopxi_pat<zextloadi8, truncstorei8, u6_0ImmPred, add, m5_0Imm8Pred,
  3294:         /*zext*/    NegImm8, L4_isub_memopb_io>;
  3295:   defm: Memopxi_pat<extloadi16, truncstorei16, u6_1ImmPred, add, m5_0Imm16Pred,
  3296:         /*anyext*/  NegImm16, L4_isub_memoph_io>;
  3297:   defm: Memopxi_pat<sextloadi16, truncstorei16, u6_1ImmPred, add, m5_0Imm16Pred,
  3298:         /*sext*/    NegImm16, L4_isub_memoph_io>;
  3299:   defm: Memopxi_pat<zextloadi16, truncstorei16, u6_1ImmPred, add, m5_0Imm16Pred,
  3300:         /*zext*/    NegImm16, L4_isub_memoph_io>;
  3301:   defm: Memopxi_pat<load, store, u6_2ImmPred, add, m5_0ImmPred, NegImm32,
  3302:                     L4_isub_memopw_io>;
  3303: 
  3304:   // clrbit imm
  3305:   defm: Memopxi_pat<extloadi8, truncstorei8, u6_0ImmPred, and, IsNPow2_8,
  3306:         /*anyext*/  LogN2_8, L4_iand_memopb_io>;
  3307:   defm: Memopxi_pat<sextloadi8, truncstorei8, u6_0ImmPred, and, IsNPow2_8,
  3308:         /*sext*/    LogN2_8, L4_iand_memopb_io>;
  3309:   defm: Memopxi_pat<zextloadi8, truncstorei8, u6_0ImmPred, and, IsNPow2_8,
  3310:         /*zext*/    LogN2_8, L4_iand_memopb_io>;
  3311:   defm: Memopxi_pat<extloadi16, truncstorei16, u6_1ImmPred, and, IsNPow2_16,
  3312:         /*anyext*/  LogN2_16, L4_iand_memoph_io>;
  3313:   defm: Memopxi_pat<sextloadi16, truncstorei16, u6_1ImmPred, and, IsNPow2_16,
  3314:         /*sext*/    LogN2_16, L4_iand_memoph_io>;
  3315:   defm: Memopxi_pat<zextloadi16, truncstorei16, u6_1ImmPred, and, IsNPow2_16,
  3316:         /*zext*/    LogN2_16, L4_iand_memoph_io>;
  3317:   defm: Memopxi_pat<load, store, u6_2ImmPred, and, IsNPow2_32,
  3318: 		    LogN2_32, L4_iand_memopw_io>;
  3319: 
  3320:   // setbit imm
  3321:   defm: Memopxi_pat<extloadi8, truncstorei8, u6_0ImmPred, or, IsPow2_32,
  3322:         /*anyext*/  Log2_8, L4_ior_memopb_io>;
  3323:   defm: Memopxi_pat<sextloadi8, truncstorei8, u6_0ImmPred, or, IsPow2_32,
  3324:         /*sext*/    Log2_8, L4_ior_memopb_io>;
  3325:   defm: Memopxi_pat<zextloadi8, truncstorei8, u6_0ImmPred, or, IsPow2_32,
  3326:         /*zext*/    Log2_8, L4_ior_memopb_io>;
  3327:   defm: Memopxi_pat<extloadi16, truncstorei16, u6_1ImmPred, or, IsPow2_32,
  3328:         /*anyext*/  Log2_16, L4_ior_memoph_io>;
  3329:   defm: Memopxi_pat<sextloadi16, truncstorei16, u6_1ImmPred, or, IsPow2_32,
  3330:         /*sext*/    Log2_16, L4_ior_memoph_io>;
  3331:   defm: Memopxi_pat<zextloadi16, truncstorei16, u6_1ImmPred, or, IsPow2_32,
  3332:         /*zext*/    Log2_16, L4_ior_memoph_io>;
  3333:   defm: Memopxi_pat<load, store, u6_2ImmPred, or, IsPow2_32,
  3334: 		    Log2_32, L4_ior_memopw_io>;
  3335: }
  3336: 
  3337: 
  3338: // --(15) Call -----------------------------------------------------------
  3339: //
  3340: 
  3341: // Pseudo instructions.
  3342: def SDT_SPCallSeqStart
  3343:   : SDCallSeqStart<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
  3344: def SDT_SPCallSeqEnd
  3345:   : SDCallSeqEnd<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
  3346: 
  3347: def callseq_start: SDNode<"ISD::CALLSEQ_START", SDT_SPCallSeqStart,
  3348:                           [SDNPHasChain, SDNPOutGlue]>;
  3349: def callseq_end:   SDNode<"ISD::CALLSEQ_END",   SDT_SPCallSeqEnd,
  3350:                           [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
  3351: 
  3352: def SDT_SPCall: SDTypeProfile<0, 1, [SDTCisVT<0, i32>]>;
  3353: 
  3354: def HexagonTCRet: SDNode<"HexagonISD::TC_RETURN", SDT_SPCall,
  3355:                          [SDNPHasChain,  SDNPOptInGlue, SDNPVariadic]>;
  3356: 
  3357: // Function call.
  3358: def callv3: SDNode<"HexagonISD::CALL", SDT_SPCall,
  3359:                    [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue, SDNPVariadic]>;
  3360: 
  3361: // Function call that does not return.
  3362: def callv3nr: SDNode<"HexagonISD::CALLnr", SDT_SPCall,
  3363:                      [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue, SDNPVariadic]>;
  3364: 
  3365: def: Pat<(callseq_start timm:$amt, timm:$amt2),
  3366:          (ADJCALLSTACKDOWN imm:$amt, imm:$amt2)>;
  3367: def: Pat<(callseq_end timm:$amt1, timm:$amt2),
  3368:          (ADJCALLSTACKUP imm:$amt1, imm:$amt2)>;
  3369: 
  3370: def: Pat<(HexagonTCRet tglobaladdr:$dst),   (PS_tailcall_i tglobaladdr:$dst)>;
  3371: def: Pat<(HexagonTCRet texternalsym:$dst),  (PS_tailcall_i texternalsym:$dst)>;
  3372: def: Pat<(HexagonTCRet I32:$dst),           (PS_tailcall_r I32:$dst)>;
  3373: 
  3374: def: Pat<(callv3 I32:$dst),                 (J2_callr I32:$dst)>;
  3375: def: Pat<(callv3 tglobaladdr:$dst),         (J2_call tglobaladdr:$dst)>;
  3376: def: Pat<(callv3 texternalsym:$dst),        (J2_call texternalsym:$dst)>;
  3377: def: Pat<(callv3 tglobaltlsaddr:$dst),      (J2_call tglobaltlsaddr:$dst)>;
  3378: 
  3379: def: Pat<(callv3nr I32:$dst),               (PS_callr_nr I32:$dst)>;
  3380: def: Pat<(callv3nr tglobaladdr:$dst),       (PS_call_nr tglobaladdr:$dst)>;
  3381: def: Pat<(callv3nr texternalsym:$dst),      (PS_call_nr texternalsym:$dst)>;
  3382: 
  3383: // Return with a glue operand.
  3384: def retglue : SDNode<"HexagonISD::RET_GLUE", SDTNone,
  3385:                      [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
  3386: 
  3387: def eh_return: SDNode<"HexagonISD::EH_RETURN", SDTNone, [SDNPHasChain]>;
  3388: 
  3389: def: Pat<(retglue),   (PS_jmpret (i32 R31))>;
  3390: def: Pat<(eh_return), (EH_RETURN_JMPR (i32 R31))>;
  3391: 
  3392: 
  3393: // --(16) Branch ---------------------------------------------------------
  3394: //
  3395: 
  3396: def: Pat<(br      bb:$dst),         (J2_jump  b30_2Imm:$dst)>;
  3397: def: Pat<(brind   I32:$dst),        (J2_jumpr I32:$dst)>;
  3398: 
  3399: def: Pat<(brcond I1:$Pu, bb:$dst),
  3400:          (J2_jumpt I1:$Pu, bb:$dst)>;
```
- EN: It defines declarative TableGen records like Memopxi_base_pat, Memopxi_add_pat, Memopxi_pat, SDT_SPCallSeqStart, SDT_SPCallSeqEnd, ... (13 total); these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonTCRet, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里定义了 Memopxi_base_pat, Memopxi_add_pat, Memopxi_pat, SDT_SPCallSeqStart, SDT_SPCallSeqEnd, ... (13 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonTCRet, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 3401-3517 / 第 3401-3517 行

```tablegen
  3401: def: Pat<(brcond (not I1:$Pu), bb:$dst),
  3402:          (J2_jumpf I1:$Pu, bb:$dst)>;
  3403: def: Pat<(brcond (i1 (setne I1:$Pu, -1)), bb:$dst),
  3404:          (J2_jumpf I1:$Pu, bb:$dst)>;
  3405: def: Pat<(brcond (i1 (seteq I1:$Pu, 0)), bb:$dst),
  3406:          (J2_jumpf I1:$Pu, bb:$dst)>;
  3407: def: Pat<(brcond (i1 (setne I1:$Pu, 0)), bb:$dst),
  3408:          (J2_jumpt I1:$Pu, bb:$dst)>;
  3409: 
  3410: 
  3411: // --(17) Misc -----------------------------------------------------------
  3412: 
  3413: 
  3414: // Generate code of the form 'C2_muxii(cmpbgtui(Rdd, C-1),0,1)'
  3415: // for C code of the form r = (c>='0' && c<='9') ? 1 : 0.
  3416: // The isdigit transformation relies on two 'clever' aspects:
  3417: // 1) The data type is unsigned which allows us to eliminate a zero test after
  3418: //    biasing the expression by 48. We are depending on the representation of
  3419: //    the unsigned types, and semantics.
  3420: // 2) The front end has converted <= 9 into < 10 on entry to LLVM.
  3421: //
  3422: // For the C code:
  3423: //   retval = (c >= '0' && c <= '9') ? 1 : 0;
  3424: // The code is transformed upstream of llvm into
  3425: //   retval = (c-48) < 10 ? 1 : 0;
  3426: 
  3427: def u7_0PosImmPred : ImmLeaf<i32, [{
  3428:   // True if the immediate fits in an 7-bit unsigned field and is positive.
  3429:   return Imm > 0 && isUInt<7>(Imm);
  3430: }]>;
  3431: 
  3432: let AddedComplexity = 139 in
  3433: def: Pat<(i32 (zext (i1 (setult (and I32:$Rs, 255), u7_0PosImmPred:$u7)))),
  3434:          (C2_muxii (A4_cmpbgtui IntRegs:$Rs, (UDEC1 imm:$u7)), 0, 1)>;
  3435: 
  3436: let AddedComplexity = 100 in
  3437: def: Pat<(or (or (shl (HexagonINSERT (i32 (zextloadi8 (add I32:$b, 2))),
  3438:                                      (i32 (extloadi8  (add I32:$b, 3))),
  3439:                                      24, 8),
  3440:                       (i32 16)),
  3441:                  (shl (i32 (zextloadi8 (add I32:$b, 1))), (i32 8))),
  3442:              (zextloadi8 I32:$b)),
  3443:          (A2_swiz (L2_loadri_io I32:$b, 0))>;
  3444: 
  3445: 
  3446: // We need custom lowering of ISD::PREFETCH into HexagonISD::DCFETCH
  3447: // because the SDNode ISD::PREFETCH has properties MayLoad and MayStore.
  3448: // We don't really want either one here.
  3449: def SDTHexagonDCFETCH: SDTypeProfile<0, 2, [SDTCisPtrTy<0>,SDTCisInt<1>]>;
  3450: def HexagonDCFETCH: SDNode<"HexagonISD::DCFETCH", SDTHexagonDCFETCH,
  3451:                            [SDNPHasChain]>;
  3452: 
  3453: def: Pat<(HexagonDCFETCH IntRegs:$Rs, u11_3ImmPred:$u11_3),
  3454:          (Y2_dcfetchbo IntRegs:$Rs, imm:$u11_3)>;
  3455: def: Pat<(HexagonDCFETCH (i32 (add IntRegs:$Rs, u11_3ImmPred:$u11_3)), (i32 0)),
  3456:          (Y2_dcfetchbo IntRegs:$Rs, imm:$u11_3)>;
  3457: 
  3458: def SDTHexagonALLOCA
  3459:   : SDTypeProfile<1, 2, [SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
  3460: def HexagonALLOCA
  3461:   : SDNode<"HexagonISD::ALLOCA", SDTHexagonALLOCA, [SDNPHasChain]>;
  3462: 
  3463: def: Pat<(HexagonALLOCA I32:$Rs, (i32 imm:$A)),
  3464:          (PS_alloca IntRegs:$Rs, imm:$A)>;
  3465: 
  3466: // Memory barrier.
  3467: def HexagonBARRIER: SDNode<"HexagonISD::BARRIER", SDTNone, [SDNPHasChain]>;
  3468: 
  3469: def: Pat<(HexagonBARRIER), (Y2_barrier)>;
  3470: 
  3471: def: Pat<(trap), (PS_crash)>;
  3472: def: Pat<(debugtrap), (J2_trap0 0xDB)>;
  3473: 
  3474: // Read cycle counter.
  3475: def : Pat<(i64 (readcyclecounter)), (PS_readcr64 UPCYCLE)>;
  3476: 
  3477: // Read packet counter
  3478: def : Pat<(i64 (readsteadycounter)), (PS_readcr64 UTIMER)>;
  3479: 
  3480: def SDTInt32Leaf : SDTypeProfile<1, 0, [SDTCisVT<0, i32>]>;
  3481: def HexagonTHREADPOINTER : SDNode<"HexagonISD::THREAD_POINTER", SDTPtrLeaf>;
  3482: 
  3483: def : Pat<(HexagonTHREADPOINTER), (i32(COPY UGP))>;
  3484: 
  3485: // The declared return value of the store-locked intrinsics is i32, but
  3486: // the instructions actually define i1. To avoid register copies from
  3487: // IntRegs to PredRegs and back, fold the entire pattern checking the
  3488: // result against true/false.
  3489: let AddedComplexity = 100 in {
  3490:   def: Pat<(i1 (setne (int_hexagon_S2_storew_locked I32:$Rs, I32:$Rt), 0)),
  3491:            (S2_storew_locked I32:$Rs, I32:$Rt)>;
  3492:   def: Pat<(i1 (seteq (int_hexagon_S2_storew_locked I32:$Rs, I32:$Rt), 0)),
  3493:            (C2_not (S2_storew_locked I32:$Rs, I32:$Rt))>;
  3494:   def: Pat<(i1 (setne (int_hexagon_S4_stored_locked I32:$Rs, I64:$Rt), 0)),
  3495:            (S4_stored_locked I32:$Rs, I64:$Rt)>;
  3496:   def: Pat<(i1 (seteq (int_hexagon_S4_stored_locked I32:$Rs, I64:$Rt), 0)),
  3497:            (C2_not (S4_stored_locked I32:$Rs, I64:$Rt))>;
  3498: }
  3499: 
  3500: multiclass FloatClass<SDPatternOperator IntOp, InstHexagon MI,
  3501:                       PatFrag RegPred> {
  3502:   let AddedComplexity = 100 in {
  3503:     def: Pat<(i1 (seteq (IntOp RegPred:$Rs, u5_0ImmPred_timm:$u5), 0)),
  3504:              (C2_not (MI RegPred:$Rs, u5_0ImmPred_timm:$u5))>;
  3505:     def: Pat<(i1 (setne (IntOp RegPred:$Rs, u5_0ImmPred_timm:$u5), 0)),
  3506:              (MI RegPred:$Rs, u5_0ImmPred_timm:$u5)>;
  3507:   }
  3508: }
  3509: 
  3510: defm : FloatClass<int_hexagon_F2_sfclass, F2_sfclass, F32>;
  3511: defm : FloatClass<int_hexagon_F2_dfclass, F2_dfclass, F64>;
  3512: 
  3513: def: Pat<(int_hexagon_instrprof_custom (HexagonAtPcrel tglobaladdr:$addr), u32_0ImmPred:$I),
  3514:          (PS_call_instrprof_custom tglobaladdr:$addr, imm:$I)>;
  3515: 
  3516: def: Pat<(int_hexagon_instrprof_custom (HexagonCONST32 tglobaladdr:$addr), u32_0ImmPred:$I),
  3517:          (PS_call_instrprof_custom tglobaladdr:$addr, imm:$I)>;
```
- EN: It defines declarative TableGen records like u7_0PosImmPred, SDTHexagonDCFETCH, HexagonDCFETCH, SDTHexagonALLOCA, HexagonALLOCA, ... (9 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as isUInt<7>, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonINSERT, HexagonISD, HexagonDCFETCH, HexagonALLOCA, ... (8 total), showing how the code connects to sibling backend components.
- CN: 这里定义了 u7_0PosImmPred, SDTHexagonDCFETCH, HexagonDCFETCH, SDTHexagonALLOCA, HexagonALLOCA, ... (9 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 isUInt<7> 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonINSERT, HexagonISD, HexagonDCFETCH, HexagonALLOCA, ... (8 total)，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- pattern-driven instruction selection / 基于模式的指令选择
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonPatterns, HexagonCONST32, HexagonCONST32_GP, HexagonADDC, HexagonISD, HexagonSUBC, HexagonSMUL_LOHI, HexagonUMUL_LOHI, HexagonUSMUL_LOHI, HexagonPTRUE, ... (34 total)`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
