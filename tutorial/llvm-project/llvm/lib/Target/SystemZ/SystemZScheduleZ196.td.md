# SystemZScheduleZ196.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZScheduleZ196.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //=- SystemZScheduleZ196.td - SystemZ Scheduling Definitions ---*- tblgen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the machine model for Z196 to support instruction
  10: // scheduling and other instruction cost heuristics.
  11: //
  12: // Pseudos expanded right after isel do not need to be modelled here.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: def Z196Model : SchedMachineModel {
  17: 
  18:     let UnsupportedFeatures = Arch9UnsupportedFeatures.List;
  19:     
  20:     let IssueWidth = 3;
  21:     let MicroOpBufferSize = 40;     // Issue queues
  22:     let LoadLatency = 1;            // Optimistic load latency.
  23: 
  24:     let PostRAScheduler = 1;
  25: 
  26:     // Extra cycles for a mispredicted branch.
  27:     let MispredictPenalty = 16;
  28: }
  29: 
  30: let SchedModel = Z196Model in  {
  31: // These definitions need the SchedModel value. They could be put in a
  32: // subtarget common include file, but it seems the include system in Tablegen
  33: // currently (2016) rejects multiple includes of same file.
  34: 
  35: // Decoder grouping rules
  36: let NumMicroOps = 1 in {
  37:   def : WriteRes<NormalGr, []>;
  38:   def : WriteRes<BeginGroup, []> { let BeginGroup  = 1; }
  39:   def : WriteRes<EndGroup, []>   { let EndGroup    = 1; }
  40: }
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `Z196Model`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `Z196Model` 等 TableGen 记录。

### Lines 41-80 / 第 41-80 行
```tablegen
  41: def : WriteRes<GroupAlone, []> {
  42:   let NumMicroOps = 3;
  43:   let BeginGroup  = 1;
  44:   let EndGroup    = 1;
  45: }
  46: def : WriteRes<GroupAlone2, []> {
  47:   let NumMicroOps = 6;
  48:   let BeginGroup  = 1;
  49:   let EndGroup    = 1;
  50: }
  51: def : WriteRes<GroupAlone3, []> {
  52:   let NumMicroOps = 9;
  53:   let BeginGroup  = 1;
  54:   let EndGroup    = 1;
  55: }
  56: 
  57: // Incoming latency removed from the register operand which is used together
  58: // with a memory operand by the instruction.
  59: def : ReadAdvance<RegReadAdv, 4>;
  60: 
  61: // LoadLatency (above) is not used for instructions in this file. This is
  62: // instead the role of LSULatency, which is the latency value added to the
  63: // result of loads and instructions with folded memory operands.
  64: def : WriteRes<LSULatency, []> { let Latency = 4; let NumMicroOps = 0; }
  65: 
  66: let NumMicroOps = 0 in {
  67:   foreach L = 1-30 in {
  68:     def : WriteRes<!cast<SchedWrite>("WLat"#L), []> { let Latency = L; }
  69:   }
  70: }
  71: 
  72: // Execution units.
  73: def Z196_FXUnit : ProcResource<2>;
  74: def Z196_LSUnit : ProcResource<2>;
  75: def Z196_FPUnit : ProcResource<1>;
  76: def Z196_DFUnit : ProcResource<1>;
  77: def Z196_MCD    : ProcResource<1>;
  78: 
  79: // Subtarget specific definitions of scheduling resources.
  80: let NumMicroOps = 0 in {
```
- **EN**: This block declares or refines TableGen records such as `Z196_FXUnit`, `Z196_LSUnit`, `Z196_FPUnit`, `Z196_DFUnit`, `Z196_MCD`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `Z196_FXUnit`, `Z196_LSUnit`, `Z196_FPUnit`, `Z196_DFUnit`, `Z196_MCD` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 81-120 / 第 81-120 行
```tablegen
  81:   def : WriteRes<FXU, [Z196_FXUnit]>;
  82:   def : WriteRes<LSU, [Z196_LSUnit]>;
  83:   def : WriteRes<FPU, [Z196_FPUnit]>;
  84:   def : WriteRes<DFU, [Z196_DFUnit]>;
  85:   foreach Num = 2-6 in { let ReleaseAtCycles = [Num] in {
  86:     def : WriteRes<!cast<SchedWrite>("FXU"#Num), [Z196_FXUnit]>;
  87:     def : WriteRes<!cast<SchedWrite>("LSU"#Num), [Z196_LSUnit]>;
  88:     def : WriteRes<!cast<SchedWrite>("FPU"#Num), [Z196_FPUnit]>;
  89:     def : WriteRes<!cast<SchedWrite>("DFU"#Num), [Z196_DFUnit]>;
  90:   }}
  91: }
  92: 
  93: def : WriteRes<MCD, [Z196_MCD]> { let NumMicroOps = 3;
  94:                                   let BeginGroup  = 1;
  95:                                   let EndGroup    = 1; }
  96: 
  97: // -------------------------- INSTRUCTIONS ---------------------------------- //
  98: 
  99: // InstRW constructs have been used in order to preserve the
 100: // readability of the InstrInfo files.
 101: 
 102: // For each instruction, as matched by a regexp, provide a list of
 103: // resources that it needs. These will be combined into a SchedClass.
 104: 
 105: //===----------------------------------------------------------------------===//
 106: // Stack allocation
 107: //===----------------------------------------------------------------------===//
 108: 
 109: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ADJDYNALLOC$")>; // Pseudo -> LA / LAY
 110: 
 111: //===----------------------------------------------------------------------===//
 112: // Branch instructions
 113: //===----------------------------------------------------------------------===//
 114: 
 115: // Branch
 116: def : InstRW<[WLat1, LSU, EndGroup], (instregex "(Call)?BRC(L)?(Asm.*)?$")>;
 117: def : InstRW<[WLat1, LSU, EndGroup], (instregex "(Call)?J(G)?(Asm.*)?$")>;
 118: def : InstRW<[WLat1, LSU, EndGroup], (instregex "(Call)?BC(R)?(Asm.*)?$")>;
 119: def : InstRW<[WLat1, LSU, EndGroup], (instregex "(Call)?B(R)?(Asm.*)?$")>;
 120: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "BRCT(G|H)?$")>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 121-160 / 第 121-160 行
```tablegen
 121: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "BCT(G)?(R)?$")>;
 122: def : InstRW<[WLat1, FXU3, LSU, GroupAlone2],
 123:              (instregex "B(R)?X(H|L).*$")>;
 124: 
 125: // Compare and branch
 126: def : InstRW<[WLat1, FXU, LSU, GroupAlone],
 127:              (instregex "C(L)?(G)?(I|R)J(Asm.*)?$")>;
 128: def : InstRW<[WLat1, FXU, LSU, GroupAlone],
 129:              (instregex "C(L)?(G)?(I|R)B(Call|Return|Asm.*)?$")>;
 130: 
 131: //===----------------------------------------------------------------------===//
 132: // Trap instructions
 133: //===----------------------------------------------------------------------===//
 134: 
 135: // Trap
 136: def : InstRW<[WLat1, LSU, EndGroup], (instregex "(Cond)?Trap$")>;
 137: 
 138: // Compare and trap
 139: def : InstRW<[WLat1, FXU, NormalGr], (instregex "C(G)?(I|R)T(Asm.*)?$")>;
 140: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CL(G)?RT(Asm.*)?$")>;
 141: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CL(F|G)IT(Asm.*)?$")>;
 142: 
 143: //===----------------------------------------------------------------------===//
 144: // Call and return instructions
 145: //===----------------------------------------------------------------------===//
 146: 
 147: // Call
 148: def : InstRW<[WLat1, LSU, FXU2, GroupAlone], (instregex "(Call)?BRAS$")>;
 149: def : InstRW<[WLat1, LSU, FXU2, GroupAlone], (instregex "(Call)?BRASL(_XPLINK64)?$")>;
 150: def : InstRW<[WLat1, LSU, FXU2, GroupAlone], (instregex "(Call)?BAS(R)?(_XPLINK64|_STACKEXT)?$")>;
 151: def : InstRW<[WLat1, LSU, FXU2, GroupAlone], (instregex "TLS_(G|L)DCALL$")>;
 152: 
 153: // Return
 154: def : InstRW<[WLat1, LSU, EndGroup], (instregex "Return(_XPLINK)?$")>;
 155: def : InstRW<[WLat1, LSU, EndGroup], (instregex "CondReturn(_XPLINK)?$")>;
 156: 
 157: //===----------------------------------------------------------------------===//
 158: // Move instructions
 159: //===----------------------------------------------------------------------===//
 160: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 161-200 / 第 161-200 行
```tablegen
 161: // Moves
 162: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "MV(G|H)?HI$")>;
 163: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "MVI(Y)?$")>;
 164: 
 165: // Move character
 166: def : InstRW<[WLat1, FXU, LSU3, GroupAlone], (instregex "MVC$")>;
 167: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "MVCL(E|U)?$")>;
 168: 
 169: // Pseudo -> reg move
 170: def : InstRW<[WLat1, FXU, NormalGr], (instregex "COPY(_TO_REGCLASS)?$")>;
 171: def : InstRW<[WLat1, FXU, NormalGr], (instregex "EXTRACT_SUBREG$")>;
 172: def : InstRW<[WLat1, FXU, NormalGr], (instregex "INSERT_SUBREG$")>;
 173: def : InstRW<[WLat1, FXU, NormalGr], (instregex "REG_SEQUENCE$")>;
 174: 
 175: // Loads
 176: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L(Y|FH|RL|Mux)?$")>;
 177: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LG(RL)?$")>;
 178: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L128$")>;
 179: 
 180: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLIH(F|H|L)$")>;
 181: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLIL(F|H|L)$")>;
 182: 
 183: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LG(F|H)I$")>;
 184: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LHI(Mux)?$")>;
 185: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LR$")>;
 186: 
 187: // Load and test
 188: def : InstRW<[WLat1LSU, WLat1LSU, LSU, FXU, NormalGr], (instregex "LT(G)?$")>;
 189: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LT(G)?R$")>;
 190: 
 191: // Stores
 192: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STG(RL)?$")>;
 193: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "ST128$")>;
 194: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "ST(Y|FH|RL|Mux)?$")>;
 195: 
 196: // String moves.
 197: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "MVST$")>;
 198: 
 199: //===----------------------------------------------------------------------===//
 200: // Conditional move instructions
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 201-240 / 第 201-240 行
```tablegen
 201: //===----------------------------------------------------------------------===//
 202: 
 203: def : InstRW<[WLat2, FXU, EndGroup], (instregex "LOC(G)?R(Asm.*)?$")>;
 204: def : InstRW<[WLat2LSU, RegReadAdv, FXU, LSU, EndGroup],
 205:              (instregex "LOC(G)?(Asm.*)?$")>;
 206: def : InstRW<[WLat1, FXU, LSU, EndGroup], (instregex "STOC(G)?(Asm.*)?$")>;
 207: 
 208: //===----------------------------------------------------------------------===//
 209: // Sign extensions
 210: //===----------------------------------------------------------------------===//
 211: 
 212: def : InstRW<[WLat1, FXU, NormalGr], (instregex "L(B|H|G)R$")>;
 213: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LG(B|H|F)R$")>;
 214: 
 215: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LTGF$")>;
 216: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LTGFR$")>;
 217: 
 218: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LB(H|Mux)?$")>;
 219: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LH(Y)?$")>;
 220: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LH(H|Mux|RL)$")>;
 221: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LG(B|H|F)$")>;
 222: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LG(H|F)RL$")>;
 223: 
 224: //===----------------------------------------------------------------------===//
 225: // Zero extensions
 226: //===----------------------------------------------------------------------===//
 227: 
 228: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLCR(Mux)?$")>;
 229: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLHR(Mux)?$")>;
 230: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLG(C|H|F|T)R$")>;
 231: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLC(Mux)?$")>;
 232: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLH(Mux)?$")>;
 233: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LL(C|H)H$")>;
 234: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLHRL$")>;
 235: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLG(C|H|F|T|HRL|FRL)$")>;
 236: 
 237: //===----------------------------------------------------------------------===//
 238: // Truncations
 239: //===----------------------------------------------------------------------===//
 240: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 241-280 / 第 241-280 行
```tablegen
 241: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STC(H|Y|Mux)?$")>;
 242: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STH(H|Y|RL|Mux)?$")>;
 243: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STCM(H|Y)?$")>;
 244: 
 245: //===----------------------------------------------------------------------===//
 246: // Multi-register moves
 247: //===----------------------------------------------------------------------===//
 248: 
 249: // Load multiple (estimated average of 5 ops)
 250: def : InstRW<[WLat10, WLat10, LSU5, GroupAlone], (instregex "LM(H|Y|G)?$")>;
 251: 
 252: // Load multiple disjoint
 253: def : InstRW<[WLat30, WLat30, MCD], (instregex "LMD$")>;
 254: 
 255: // Store multiple (estimated average of 3 ops)
 256: def : InstRW<[WLat1, LSU2, FXU5, GroupAlone], (instregex "STM(H|Y|G)?$")>;
 257: 
 258: //===----------------------------------------------------------------------===//
 259: // Byte swaps
 260: //===----------------------------------------------------------------------===//
 261: 
 262: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LRV(G)?R$")>;
 263: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LRV(G|H)?$")>;
 264: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STRV(G|H)?$")>;
 265: def : InstRW<[WLat30, MCD], (instregex "MVCIN$")>;
 266: 
 267: //===----------------------------------------------------------------------===//
 268: // Load address instructions
 269: //===----------------------------------------------------------------------===//
 270: 
 271: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LA(Y|RL)?$")>;
 272: 
 273: // Load the Global Offset Table address
 274: def : InstRW<[WLat1, FXU, NormalGr], (instregex "GOT$")>;
 275: 
 276: //===----------------------------------------------------------------------===//
 277: // Absolute and Negation
 278: //===----------------------------------------------------------------------===//
 279: 
 280: def : InstRW<[WLat2, WLat2, FXU, NormalGr], (instregex "LP(G)?R$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 281-320 / 第 281-320 行
```tablegen
 281: def : InstRW<[WLat3, WLat3, FXU2, GroupAlone], (instregex "L(N|P)GFR$")>;
 282: def : InstRW<[WLat2, WLat2, FXU, NormalGr], (instregex "LN(R|GR)$")>;
 283: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LC(R|GR)$")>;
 284: def : InstRW<[WLat2, WLat2, FXU2, GroupAlone], (instregex "LCGFR$")>;
 285: 
 286: //===----------------------------------------------------------------------===//
 287: // Insertion
 288: //===----------------------------------------------------------------------===//
 289: 
 290: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "IC(Y)?$")>;
 291: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 292:              (instregex "IC32(Y)?$")>;
 293: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 294:              (instregex "ICM(H|Y)?$")>;
 295: def : InstRW<[WLat1, FXU, NormalGr], (instregex "II(F|H|L)Mux$")>;
 296: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IIHF(64)?$")>;
 297: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IIHH(64)?$")>;
 298: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IIHL(64)?$")>;
 299: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IILF(64)?$")>;
 300: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IILH(64)?$")>;
 301: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IILL(64)?$")>;
 302: 
 303: //===----------------------------------------------------------------------===//
 304: // Addition
 305: //===----------------------------------------------------------------------===//
 306: 
 307: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 308:              (instregex "A(L)?(Y)?$")>;
 309: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "A(L)?SI$")>;
 310: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU2, LSU, GroupAlone],
 311:              (instregex "AH(Y)?$")>;
 312: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AIH$")>;
 313: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AFI(Mux)?$")>;
 314: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AGFI$")>;
 315: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AGHI(K)?$")>;
 316: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AGR(K)?$")>;
 317: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AHI(K)?$")>;
 318: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AHIMux(K)?$")>;
 319: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AL(FI|HSIK)$")>;
 320: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 321-360 / 第 321-360 行
```tablegen
 321:              (instregex "ALGF$")>;
 322: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALGHSIK$")>;
 323: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALGF(I|R)$")>;
 324: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALGR(K)?$")>;
 325: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALR(K)?$")>;
 326: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AR(K)?$")>;
 327: def : InstRW<[WLat1, FXU, NormalGr], (instregex "A(L)?HHHR$")>;
 328: def : InstRW<[WLat2, WLat2, FXU2, GroupAlone], (instregex "A(L)?HHLR$")>;
 329: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALSIH(N)?$")>;
 330: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 331:              (instregex "A(L)?G$")>;
 332: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "A(L)?GSI$")>;
 333: 
 334: // Logical addition with carry
 335: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU, LSU, GroupAlone],
 336:              (instregex "ALC(G)?$")>;
 337: def : InstRW<[WLat2, WLat2, FXU, GroupAlone], (instregex "ALC(G)?R$")>;
 338: 
 339: // Add with sign extension (32 -> 64)
 340: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU2, LSU, GroupAlone],
 341:              (instregex "AGF$")>;
 342: def : InstRW<[WLat2, WLat2, FXU2, GroupAlone], (instregex "AGFR$")>;
 343: 
 344: //===----------------------------------------------------------------------===//
 345: // Subtraction
 346: //===----------------------------------------------------------------------===//
 347: 
 348: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 349:              (instregex "S(G|Y)?$")>;
 350: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU2, LSU, GroupAlone],
 351:              (instregex "SH(Y)?$")>;
 352: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SGR(K)?$")>;
 353: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLFI$")>;
 354: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 355:              (instregex "SL(G|GF|Y)?$")>;
 356: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLGF(I|R)$")>;
 357: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLGR(K)?$")>;
 358: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLR(K)?$")>;
 359: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SR(K)?$")>;
 360: def : InstRW<[WLat1, FXU, NormalGr], (instregex "S(L)?HHHR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 361-400 / 第 361-400 行
```tablegen
 361: def : InstRW<[WLat2, WLat2, FXU2, GroupAlone], (instregex "S(L)?HHLR$")>;
 362: 
 363: // Subtraction with borrow
 364: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU, LSU, GroupAlone],
 365:              (instregex "SLB(G)?$")>;
 366: def : InstRW<[WLat2, WLat2, FXU, GroupAlone], (instregex "SLB(G)?R$")>;
 367: 
 368: // Subtraction with sign extension (32 -> 64)
 369: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU2, LSU, GroupAlone],
 370:              (instregex "SGF$")>;
 371: def : InstRW<[WLat2, WLat2, FXU2, GroupAlone], (instregex "SGFR$")>;
 372: 
 373: //===----------------------------------------------------------------------===//
 374: // AND
 375: //===----------------------------------------------------------------------===//
 376: 
 377: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 378:              (instregex "N(G|Y)?$")>;
 379: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NGR(K)?$")>;
 380: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NI(FMux|HMux|LMux)$")>;
 381: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "NI(Y)?$")>;
 382: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NIHF(64)?$")>;
 383: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NIHH(64)?$")>;
 384: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NIHL(64)?$")>;
 385: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NILF(64)?$")>;
 386: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NILH(64)?$")>;
 387: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NILL(64)?$")>;
 388: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NR(K)?$")>;
 389: def : InstRW<[WLat5LSU, LSU2, FXU, GroupAlone], (instregex "NC$")>;
 390: 
 391: //===----------------------------------------------------------------------===//
 392: // OR
 393: //===----------------------------------------------------------------------===//
 394: 
 395: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 396:              (instregex "O(G|Y)?$")>;
 397: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OGR(K)?$")>;
 398: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "OI(Y)?$")>;
 399: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OI(FMux|HMux|LMux)$")>;
 400: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OIHF(64)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 401-440 / 第 401-440 行
```tablegen
 401: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OIHH(64)?$")>;
 402: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OIHL(64)?$")>;
 403: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OILF(64)?$")>;
 404: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OILH(64)?$")>;
 405: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OILL(64)?$")>;
 406: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OR(K)?$")>;
 407: def : InstRW<[WLat5LSU, LSU2, FXU, GroupAlone], (instregex "OC$")>;
 408: 
 409: //===----------------------------------------------------------------------===//
 410: // XOR
 411: //===----------------------------------------------------------------------===//
 412: 
 413: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 414:              (instregex "X(G|Y)?$")>;
 415: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "XI(Y)?$")>;
 416: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XIFMux$")>;
 417: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XGR(K)?$")>;
 418: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XIHF(64)?$")>;
 419: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XILF(64)?$")>;
 420: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XR(K)?$")>;
 421: def : InstRW<[WLat5LSU, LSU2, FXU, GroupAlone], (instregex "XC$")>;
 422: 
 423: //===----------------------------------------------------------------------===//
 424: // Multiplication
 425: //===----------------------------------------------------------------------===//
 426: 
 427: def : InstRW<[WLat6LSU, RegReadAdv, FXU, LSU, NormalGr],
 428:              (instregex "MS(GF|Y)?$")>;
 429: def : InstRW<[WLat6, FXU, NormalGr], (instregex "MS(R|FI)$")>;
 430: def : InstRW<[WLat8LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "MSG$")>;
 431: def : InstRW<[WLat8, FXU, NormalGr], (instregex "MSGR$")>;
 432: def : InstRW<[WLat6, FXU, NormalGr], (instregex "MSGF(I|R)$")>;
 433: def : InstRW<[WLat11LSU, RegReadAdv, FXU2, LSU, GroupAlone],
 434:              (instregex "MLG$")>;
 435: def : InstRW<[WLat9, FXU2, GroupAlone], (instregex "MLGR$")>;
 436: def : InstRW<[WLat5, FXU, NormalGr], (instregex "MGHI$")>;
 437: def : InstRW<[WLat5, FXU, NormalGr], (instregex "MHI$")>;
 438: def : InstRW<[WLat5LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "MH(Y)?$")>;
 439: def : InstRW<[WLat7, FXU2, GroupAlone], (instregex "M(L)?R$")>;
 440: def : InstRW<[WLat7LSU, RegReadAdv, FXU2, LSU, GroupAlone],
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 441-480 / 第 441-480 行
```tablegen
 441:              (instregex "M(FY|L)?$")>;
 442: 
 443: //===----------------------------------------------------------------------===//
 444: // Division and remainder
 445: //===----------------------------------------------------------------------===//
 446: 
 447: def : InstRW<[WLat30, FPU4, FXU5, GroupAlone3], (instregex "DR$")>;
 448: def : InstRW<[WLat30, RegReadAdv, FPU4, LSU, FXU4, GroupAlone3],
 449:              (instregex "D$")>;
 450: def : InstRW<[WLat30, FPU4, FXU4, GroupAlone3], (instregex "DSG(F)?R$")>;
 451: def : InstRW<[WLat30, RegReadAdv, FPU4, LSU, FXU3, GroupAlone3],
 452:              (instregex "DSG(F)?$")>;
 453: def : InstRW<[WLat30, FPU4, FXU5, GroupAlone3], (instregex "DL(G)?R$")>;
 454: def : InstRW<[WLat30, RegReadAdv, FPU4, LSU, FXU4, GroupAlone3],
 455:              (instregex "DL(G)?$")>;
 456: 
 457: //===----------------------------------------------------------------------===//
 458: // Shifts
 459: //===----------------------------------------------------------------------===//
 460: 
 461: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLL(G|K)?$")>;
 462: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SRL(G|K)?$")>;
 463: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SRA(G|K)?$")>;
 464: def : InstRW<[WLat2, WLat2, FXU, NormalGr], (instregex "SLA(G|K)?$")>;
 465: def : InstRW<[WLat5LSU, WLat5LSU, FXU4, LSU, GroupAlone2],
 466:              (instregex "S(L|R)D(A|L)$")>;
 467: 
 468: // Rotate
 469: def : InstRW<[WLat2LSU, FXU, LSU, NormalGr], (instregex "RLL(G)?$")>;
 470: 
 471: // Rotate and insert
 472: def : InstRW<[WLat1, FXU, NormalGr], (instregex "RISBH(G|H|L)(Opt)?$")>;
 473: def : InstRW<[WLat1, FXU, NormalGr], (instregex "RISBL(G|H|L)(Opt)?$")>;
 474: def : InstRW<[WLat1, FXU, NormalGr], (instregex "RISBG(32)?(Z)?(Opt)?$")>;
 475: def : InstRW<[WLat1, FXU, NormalGr], (instregex "RISBMux$")>;
 476: 
 477: // Rotate and Select
 478: def : InstRW<[WLat3, WLat3, FXU2, GroupAlone], (instregex "R(N|O|X)SBG(Opt)?$")>;
 479: 
 480: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: // Comparison
 482: //===----------------------------------------------------------------------===//
 483: 
 484: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "C(G|Y|Mux|RL)?$")>;
 485: def : InstRW<[WLat1, FXU, NormalGr], (instregex "C(F|H)I(Mux)?$")>;
 486: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CG(F|H)I$")>;
 487: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CG(HSI|RL)$")>;
 488: def : InstRW<[WLat1, FXU, NormalGr], (instregex "C(G)?R$")>;
 489: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CIH$")>;
 490: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CHF$")>;
 491: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CHSI$")>;
 492: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 493:              (instregex "CL(Y|Mux)?$")>;
 494: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLFHSI$")>;
 495: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLFI(Mux)?$")>;
 496: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CLG$")>;
 497: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLG(HRL|HSI)$")>;
 498: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CLGF$")>;
 499: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLGFRL$")>;
 500: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLGF(I|R)$")>;
 501: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLGR$")>;
 502: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLGRL$")>;
 503: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CLHF$")>;
 504: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLH(RL|HSI)$")>;
 505: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLIH$")>;
 506: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLI(Y)?$")>;
 507: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLR$")>;
 508: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLRL$")>;
 509: def : InstRW<[WLat1, FXU, NormalGr], (instregex "C(L)?HHR$")>;
 510: def : InstRW<[WLat2, FXU2, GroupAlone], (instregex "C(L)?HLR$")>;
 511: 
 512: // Compare halfword
 513: def : InstRW<[WLat2LSU, RegReadAdv, FXU2, LSU, GroupAlone],
 514:              (instregex "CH(Y)?$")>;
 515: def : InstRW<[WLat2LSU, FXU2, LSU, GroupAlone], (instregex "CHRL$")>;
 516: def : InstRW<[WLat2LSU, RegReadAdv, FXU2, LSU, GroupAlone], (instregex "CGH$")>;
 517: def : InstRW<[WLat2LSU, FXU2, LSU, GroupAlone], (instregex "CGHRL$")>;
 518: def : InstRW<[WLat2LSU, FXU2, LSU, GroupAlone], (instregex "CHHSI$")>;
 519: 
 520: // Compare with sign extension (32 -> 64)
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 521-560 / 第 521-560 行
```tablegen
 521: def : InstRW<[WLat2LSU, RegReadAdv, FXU2, LSU, GroupAlone], (instregex "CGF$")>;
 522: def : InstRW<[WLat2LSU, FXU2, LSU, GroupAlone], (instregex "CGFRL$")>;
 523: def : InstRW<[WLat2, FXU2, GroupAlone], (instregex "CGFR$")>;
 524: 
 525: // Compare logical character
 526: def : InstRW<[WLat9, FXU, LSU2, GroupAlone], (instregex "CLC$")>;
 527: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLCL(E|U)?$")>;
 528: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLST$")>;
 529: 
 530: // Test under mask
 531: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "TM(Y)?$")>;
 532: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TM(H|L)Mux$")>;
 533: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TMHH(64)?$")>;
 534: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TMHL(64)?$")>;
 535: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TMLH(64)?$")>;
 536: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TMLL(64)?$")>;
 537: 
 538: // Compare logical characters under mask
 539: def : InstRW<[WLat2LSU, RegReadAdv, FXU2, LSU, GroupAlone],
 540:              (instregex "CLM(H|Y)?$")>;
 541: 
 542: //===----------------------------------------------------------------------===//
 543: // Prefetch
 544: //===----------------------------------------------------------------------===//
 545: 
 546: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "PFD(RL)?$")>;
 547: 
 548: //===----------------------------------------------------------------------===//
 549: // Atomic operations
 550: //===----------------------------------------------------------------------===//
 551: 
 552: def : InstRW<[WLat1, LSU, EndGroup], (instregex "Serialize$")>;
 553: 
 554: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAA(G)?$")>;
 555: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAAL(G)?$")>;
 556: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAN(G)?$")>;
 557: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAO(G)?$")>;
 558: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAX(G)?$")>;
 559: 
 560: // Test and set
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: def : InstRW<[WLat1LSU, FXU, LSU, EndGroup], (instregex "TS$")>;
 562: 
 563: // Compare and swap
 564: def : InstRW<[WLat2LSU, WLat2LSU, FXU2, LSU, GroupAlone],
 565:              (instregex "CS(G|Y)?$")>;
 566: 
 567: // Compare double and swap
 568: def : InstRW<[WLat5LSU, WLat5LSU, FXU5, LSU, GroupAlone2],
 569:              (instregex "CDS(Y)?$")>;
 570: def : InstRW<[WLat12, WLat12, FXU6, LSU2, GroupAlone],
 571:              (instregex "CDSG$")>;
 572: 
 573: // Compare and swap and store
 574: def : InstRW<[WLat30, MCD], (instregex "CSST$")>;
 575: 
 576: // Perform locked operation
 577: def : InstRW<[WLat30, MCD], (instregex "PLO$")>;
 578: 
 579: // Load/store pair from/to quadword
 580: def : InstRW<[WLat4LSU, LSU2, GroupAlone], (instregex "LPQ$")>;
 581: def : InstRW<[WLat1, FXU2, LSU2, GroupAlone], (instregex "STPQ$")>;
 582: 
 583: // Load pair disjoint
 584: def : InstRW<[WLat2LSU, WLat2LSU, LSU2, GroupAlone], (instregex "LPD(G)?$")>;
 585: 
 586: //===----------------------------------------------------------------------===//
 587: // Translate and convert
 588: //===----------------------------------------------------------------------===//
 589: 
 590: def : InstRW<[WLat30, MCD], (instregex "TR$")>;
 591: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRT$")>;
 592: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRTR$")>;
 593: def : InstRW<[WLat30, WLat30, MCD], (instregex "TRE$")>;
 594: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRT(R)?E(Opt)?$")>;
 595: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TR(T|O)(T|O)(Opt)?$")>;
 596: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 597:              (instregex "CU(12|14|21|24|41|42)(Opt)?$")>;
 598: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "(CUUTF|CUTFU)(Opt)?$")>;
 599: 
 600: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 601-640 / 第 601-640 行
```tablegen
 601: // Message-security assist
 602: //===----------------------------------------------------------------------===//
 603: 
 604: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD],
 605:              (instregex "KM(C|F|O|CTR)?$")>;
 606: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "(KIMD|KLMD|KMAC|PCC)$")>;
 607: 
 608: //===----------------------------------------------------------------------===//
 609: // Decimal arithmetic
 610: //===----------------------------------------------------------------------===//
 611: 
 612: def : InstRW<[WLat30, RegReadAdv, FXU, DFU2, LSU2, GroupAlone2],
 613:              (instregex "CVBG$")>;
 614: def : InstRW<[WLat20, RegReadAdv, FXU, DFU, LSU, GroupAlone2],
 615:              (instregex "CVB(Y)?$")>;
 616: def : InstRW<[WLat1, FXU3, DFU4, LSU, GroupAlone3], (instregex "CVDG$")>;
 617: def : InstRW<[WLat1, FXU2, DFU, LSU, GroupAlone3], (instregex "CVD(Y)?$")>;
 618: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "MV(N|O|Z)$")>;
 619: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "(PACK|PKA|PKU)$")>;
 620: def : InstRW<[WLat10, LSU5, GroupAlone], (instregex "UNPK(A|U)$")>;
 621: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "UNPK$")>;
 622: 
 623: def : InstRW<[WLat11LSU, FXU, DFU4, LSU2, GroupAlone],
 624:              (instregex "(A|S|ZA)P$")>;
 625: def : InstRW<[WLat1, FXU, DFU4, LSU2, GroupAlone], (instregex "(M|D)P$")>;
 626: def : InstRW<[WLat15, FXU2, DFU4, LSU3, GroupAlone], (instregex "SRP$")>;
 627: def : InstRW<[WLat11, DFU4, LSU2, GroupAlone], (instregex "CP$")>;
 628: def : InstRW<[WLat5LSU, DFU2, LSU2, GroupAlone], (instregex "TP$")>;
 629: def : InstRW<[WLat30, MCD], (instregex "ED(MK)?$")>;
 630: 
 631: //===----------------------------------------------------------------------===//
 632: // Access registers
 633: //===----------------------------------------------------------------------===//
 634: 
 635: // Extract/set/copy access register
 636: def : InstRW<[WLat3, LSU, NormalGr], (instregex "(EAR|SAR|CPYA)$")>;
 637: 
 638: // Load address extended
 639: def : InstRW<[WLat5, LSU, FXU, GroupAlone], (instregex "LAE(Y)?$")>;
 640: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 641-680 / 第 641-680 行
```tablegen
 641: // Load/store access multiple (not modeled precisely)
 642: def : InstRW<[WLat10, WLat10, LSU5, GroupAlone], (instregex "LAM(Y)?$")>;
 643: def : InstRW<[WLat1, FXU5, LSU5, GroupAlone], (instregex "STAM(Y)?$")>;
 644: 
 645: //===----------------------------------------------------------------------===//
 646: // Program mask and addressing mode
 647: //===----------------------------------------------------------------------===//
 648: 
 649: // Insert Program Mask
 650: def : InstRW<[WLat3, FXU, EndGroup], (instregex "IPM$")>;
 651: 
 652: // Set Program Mask
 653: def : InstRW<[WLat3, LSU, EndGroup], (instregex "SPM$")>;
 654: 
 655: // Branch and link
 656: def : InstRW<[WLat1, FXU2, LSU, GroupAlone], (instregex "BAL(R)?$")>;
 657: 
 658: // Test addressing mode
 659: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TAM$")>;
 660: 
 661: // Set addressing mode
 662: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SAM(24|31|64)$")>;
 663: 
 664: // Branch (and save) and set mode.
 665: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "BSM$")>;
 666: def : InstRW<[WLat1, FXU2, LSU, GroupAlone], (instregex "BASSM$")>;
 667: 
 668: //===----------------------------------------------------------------------===//
 669: // Miscellaneous Instructions.
 670: //===----------------------------------------------------------------------===//
 671: 
 672: // Find leftmost one
 673: def : InstRW<[WLat7, WLat7, FXU2, GroupAlone], (instregex "FLOGR$")>;
 674: 
 675: // Population count
 676: def : InstRW<[WLat3, WLat3, FXU, NormalGr], (instregex "POPCNT$")>;
 677: 
 678: // String instructions
 679: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "SRST(U)?$")>;
 680: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CUSE$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 681-720 / 第 681-720 行
```tablegen
 681: 
 682: // Various complex instructions
 683: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CFC$")>;
 684: def : InstRW<[WLat30, WLat30, WLat30, WLat30, WLat30, WLat30, MCD],
 685:              (instregex "UPT$")>;
 686: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CKSM$")>;
 687: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CMPSC$")>;
 688: 
 689: // Execute
 690: def : InstRW<[LSU, GroupAlone], (instregex "EX(RL)?$")>;
 691: 
 692: //===----------------------------------------------------------------------===//
 693: // .insn directive instructions
 694: //===----------------------------------------------------------------------===//
 695: 
 696: // An "empty" sched-class will be assigned instead of the "invalid sched-class".
 697: // getNumDecoderSlots() will then return 1 instead of 0.
 698: def : InstRW<[], (instregex "Insn.*")>;
 699: 
 700: 
 701: // ----------------------------- Floating point ----------------------------- //
 702: 
 703: //===----------------------------------------------------------------------===//
 704: // FP: Move instructions
 705: //===----------------------------------------------------------------------===//
 706: 
 707: // Load zero
 708: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LZ(DR|ER|ER_16)$")>;
 709: def : InstRW<[WLat2, FXU2, GroupAlone2], (instregex "LZXR$")>;
 710: 
 711: // Load
 712: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LER(16)?$")>;
 713: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LD(R|R16|R32|GR)$")>;
 714: def : InstRW<[WLat3, FXU, NormalGr], (instregex "LGDR$")>;
 715: def : InstRW<[WLat2, FXU2, GroupAlone2], (instregex "LXR$")>;
 716: 
 717: // Load and Test
 718: def : InstRW<[WLat9, WLat9, FPU, NormalGr], (instregex "LT(E|D)BR$")>;
 719: def : InstRW<[WLat10, WLat10, FPU4, GroupAlone], (instregex "LTXBR$")>;
 720: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: // Copy sign
 722: def : InstRW<[WLat5, FXU2, GroupAlone], (instregex "CPSDR(d|s|h)(d|s|h)$")>;
 723: 
 724: //===----------------------------------------------------------------------===//
 725: // FP: Load instructions
 726: //===----------------------------------------------------------------------===//
 727: 
 728: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L(E16|E|D)(Y|E32)?$")>;
 729: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LX$")>;
 730: 
 731: //===----------------------------------------------------------------------===//
 732: // FP: Store instructions
 733: //===----------------------------------------------------------------------===//
 734: 
 735: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "ST(E16|E|D)(Y)?$")>;
 736: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STX$")>;
 737: 
 738: //===----------------------------------------------------------------------===//
 739: // FP: Conversion instructions
 740: //===----------------------------------------------------------------------===//
 741: 
 742: // Load rounded
 743: def : InstRW<[WLat7, FPU, NormalGr], (instregex "LEDBR(A)?$")>;
 744: def : InstRW<[WLat9, FPU2, NormalGr], (instregex "L(E|D)XBR(A)?$")>;
 745: 
 746: // Load lengthened
 747: def : InstRW<[WLat7LSU, FPU, LSU, NormalGr], (instregex "LDEB$")>;
 748: def : InstRW<[WLat7, FPU, NormalGr], (instregex "LDEBR$")>;
 749: def : InstRW<[WLat11LSU, FPU4, LSU, GroupAlone], (instregex "LX(E|D)B$")>;
 750: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "LX(E|D)BR$")>;
 751: 
 752: // Convert from fixed / logical
 753: def : InstRW<[WLat8, FXU, FPU, GroupAlone], (instregex "C(E|D)(F|G)BR(A)?$")>;
 754: def : InstRW<[WLat11, FXU, FPU4, GroupAlone2], (instregex "CX(F|G)BR(A?)$")>;
 755: def : InstRW<[WLat8, FXU, FPU, GroupAlone], (instregex "CEL(F|G)BR$")>;
 756: def : InstRW<[WLat8, FXU, FPU, GroupAlone], (instregex "CDL(F|G)BR$")>;
 757: def : InstRW<[WLat11, FXU, FPU4, GroupAlone2], (instregex "CXL(F|G)BR$")>;
 758: 
 759: // Convert to fixed / logical
 760: def : InstRW<[WLat12, WLat12, FXU, FPU, GroupAlone],
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 761-800 / 第 761-800 行
```tablegen
 761:              (instregex "C(F|G)(E|D)BR(A?)$")>;
 762: def : InstRW<[WLat12, WLat12, FXU, FPU2, GroupAlone],
 763:              (instregex "C(F|G)XBR(A?)$")>;
 764: def : InstRW<[WLat12, WLat12, FXU, FPU, GroupAlone],
 765:              (instregex "CL(F|G)(E|D)BR$")>;
 766: def : InstRW<[WLat12, WLat12, FXU, FPU2, GroupAlone], (instregex "CL(F|G)XBR$")>;
 767: 
 768: //===----------------------------------------------------------------------===//
 769: // FP: Unary arithmetic
 770: //===----------------------------------------------------------------------===//
 771: 
 772: // Load Complement / Negative / Positive
 773: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "L(C|N|P)(E|D)BR$")>;
 774: def : InstRW<[WLat1, FXU, NormalGr], (instregex "L(C|N|P)DFR(_32|_16)?$")>;
 775: def : InstRW<[WLat10, WLat10, FPU4, GroupAlone], (instregex "L(C|N|P)XBR$")>;
 776: 
 777: // Square root
 778: def : InstRW<[WLat30, FPU, LSU, NormalGr], (instregex "SQ(E|D)B$")>;
 779: def : InstRW<[WLat30, FPU, NormalGr], (instregex "SQ(E|D)BR$")>;
 780: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "SQXBR$")>;
 781: 
 782: // Load FP integer
 783: def : InstRW<[WLat7, FPU, NormalGr], (instregex "FI(E|D)BR(A)?$")>;
 784: def : InstRW<[WLat15, FPU4, GroupAlone], (instregex "FIXBR(A)?$")>;
 785: 
 786: //===----------------------------------------------------------------------===//
 787: // FP: Binary arithmetic
 788: //===----------------------------------------------------------------------===//
 789: 
 790: // Addition
 791: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 792:              (instregex "A(E|D)B$")>;
 793: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "A(E|D)BR$")>;
 794: def : InstRW<[WLat20, WLat20, FPU4, GroupAlone], (instregex "AXBR$")>;
 795: 
 796: // Subtraction
 797: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 798:              (instregex "S(E|D)B$")>;
 799: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "S(E|D)BR$")>;
 800: def : InstRW<[WLat20, WLat20, FPU4, GroupAlone], (instregex "SXBR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: 
 802: // Multiply
 803: def : InstRW<[WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 804:              (instregex "M(D|DE|EE)B$")>;
 805: def : InstRW<[WLat7, FPU, NormalGr], (instregex "M(D|DE|EE)BR$")>;
 806: def : InstRW<[WLat11LSU, RegReadAdv, FPU4, LSU, GroupAlone],
 807:              (instregex "MXDB$")>;
 808: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "MXDBR$")>;
 809: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "MXBR$")>;
 810: 
 811: // Multiply and add / subtract
 812: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, FPU2, LSU, GroupAlone],
 813:              (instregex "M(A|S)EB$")>;
 814: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "M(A|S)EBR$")>;
 815: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, FPU2, LSU, GroupAlone],
 816:              (instregex "M(A|S)DB$")>;
 817: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "M(A|S)DBR$")>;
 818: 
 819: // Division
 820: def : InstRW<[WLat30, RegReadAdv, FPU, LSU, NormalGr], (instregex "D(E|D)B$")>;
 821: def : InstRW<[WLat30, FPU, NormalGr], (instregex "D(E|D)BR$")>;
 822: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "DXBR$")>;
 823: 
 824: // Divide to integer
 825: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "DI(E|D)BR$")>;
 826: 
 827: //===----------------------------------------------------------------------===//
 828: // FP: Comparisons
 829: //===----------------------------------------------------------------------===//
 830: 
 831: // Compare
 832: def : InstRW<[WLat11LSU, RegReadAdv, FPU, LSU, NormalGr],
 833:              (instregex "(K|C)(E|D)B$")>;
 834: def : InstRW<[WLat9, FPU, NormalGr], (instregex "(K|C)(E|D)BR$")>;
 835: def : InstRW<[WLat30, FPU2, NormalGr], (instregex "(K|C)XBR$")>;
 836: 
 837: // Test Data Class
 838: def : InstRW<[WLat15, FPU, LSU, NormalGr], (instregex "TC(E|D)B$")>;
 839: def : InstRW<[WLat15, FPU4, LSU, GroupAlone], (instregex "TCXB$")>;
 840: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: //===----------------------------------------------------------------------===//
 842: // FP: Floating-point control register instructions
 843: //===----------------------------------------------------------------------===//
 844: 
 845: def : InstRW<[WLat4, FXU, LSU, GroupAlone], (instregex "EFPC$")>;
 846: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "STFPC$")>;
 847: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "SFPC$")>;
 848: def : InstRW<[WLat1, LSU2, GroupAlone], (instregex "LFPC$")>;
 849: def : InstRW<[WLat30, MCD], (instregex "SFASR$")>;
 850: def : InstRW<[WLat30, MCD], (instregex "LFAS$")>;
 851: def : InstRW<[WLat2, FXU, GroupAlone], (instregex "SRNM(B|T)?$")>;
 852: 
 853: 
 854: // --------------------- Hexadecimal floating point ------------------------- //
 855: 
 856: //===----------------------------------------------------------------------===//
 857: // HFP: Move instructions
 858: //===----------------------------------------------------------------------===//
 859: 
 860: // Load and Test
 861: def : InstRW<[WLat9, WLat9, FPU, NormalGr], (instregex "LT(E|D)R$")>;
 862: def : InstRW<[WLat9, WLat9, FPU4, GroupAlone], (instregex "LTXR$")>;
 863: 
 864: //===----------------------------------------------------------------------===//
 865: // HFP: Conversion instructions
 866: //===----------------------------------------------------------------------===//
 867: 
 868: // Load rounded
 869: def : InstRW<[WLat7, FPU, NormalGr], (instregex "(LEDR|LRER)$")>;
 870: def : InstRW<[WLat7, FPU, NormalGr], (instregex "LEXR$")>;
 871: def : InstRW<[WLat9, FPU, NormalGr], (instregex "(LDXR|LRDR)$")>;
 872: 
 873: // Load lengthened
 874: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LDE$")>;
 875: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LDER$")>;
 876: def : InstRW<[WLat11LSU, FPU4, LSU, GroupAlone], (instregex "LX(E|D)$")>;
 877: def : InstRW<[WLat9, FPU4, GroupAlone], (instregex "LX(E|D)R$")>;
 878: 
 879: // Convert from fixed
 880: def : InstRW<[WLat8, FXU, FPU, GroupAlone], (instregex "C(E|D)(F|G)R$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 881-920 / 第 881-920 行
```tablegen
 881: def : InstRW<[WLat10, FXU, FPU4, GroupAlone2], (instregex "CX(F|G)R$")>;
 882: 
 883: // Convert to fixed
 884: def : InstRW<[WLat12, WLat12, FXU, FPU, GroupAlone],
 885:              (instregex "C(F|G)(E|D)R$")>;
 886: def : InstRW<[WLat30, WLat30, FXU, FPU2, GroupAlone], (instregex "C(F|G)XR$")>;
 887: 
 888: // Convert BFP to HFP / HFP to BFP.
 889: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "THD(E)?R$")>;
 890: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "TB(E)?DR$")>;
 891: 
 892: //===----------------------------------------------------------------------===//
 893: // HFP: Unary arithmetic
 894: //===----------------------------------------------------------------------===//
 895: 
 896: // Load Complement / Negative / Positive
 897: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "L(C|N|P)(E|D)R$")>;
 898: def : InstRW<[WLat9, WLat9, FPU4, GroupAlone], (instregex "L(C|N|P)XR$")>;
 899: 
 900: // Halve
 901: def : InstRW<[WLat7, FPU, NormalGr], (instregex "H(E|D)R$")>;
 902: 
 903: // Square root
 904: def : InstRW<[WLat30, FPU, LSU, NormalGr], (instregex "SQ(E|D)$")>;
 905: def : InstRW<[WLat30, FPU, NormalGr], (instregex "SQ(E|D)R$")>;
 906: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "SQXR$")>;
 907: 
 908: // Load FP integer
 909: def : InstRW<[WLat7, FPU, NormalGr], (instregex "FI(E|D)R$")>;
 910: def : InstRW<[WLat15, FPU4, GroupAlone], (instregex "FIXR$")>;
 911: 
 912: //===----------------------------------------------------------------------===//
 913: // HFP: Binary arithmetic
 914: //===----------------------------------------------------------------------===//
 915: 
 916: // Addition
 917: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 918:              (instregex "A(E|D|U|W)$")>;
 919: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "A(E|D|U|W)R$")>;
 920: def : InstRW<[WLat15, WLat15, FPU4, GroupAlone], (instregex "AXR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: 
 922: // Subtraction
 923: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 924:              (instregex "S(E|D|U|W)$")>;
 925: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "S(E|D|U|W)R$")>;
 926: def : InstRW<[WLat15, WLat15, FPU4, GroupAlone], (instregex "SXR$")>;
 927: 
 928: // Multiply
 929: def : InstRW<[WLat7LSU, RegReadAdv, FPU, LSU, NormalGr], (instregex "M(D|EE)$")>;
 930: def : InstRW<[WLat8LSU, RegReadAdv, FPU, LSU, NormalGr], (instregex "M(DE|E)$")>;
 931: def : InstRW<[WLat7, FPU, NormalGr], (instregex "M(D|EE)R$")>;
 932: def : InstRW<[WLat8, FPU, NormalGr], (instregex "M(DE|E)R$")>;
 933: def : InstRW<[WLat11LSU, RegReadAdv, FPU4, LSU, GroupAlone], (instregex "MXD$")>;
 934: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "MXDR$")>;
 935: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "MXR$")>;
 936: def : InstRW<[WLat11LSU, RegReadAdv, FPU4, LSU, GroupAlone], (instregex "MY$")>;
 937: def : InstRW<[WLat7LSU, RegReadAdv, FPU2, LSU, GroupAlone],
 938:              (instregex "MY(H|L)$")>;
 939: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "MYR$")>;
 940: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "MY(H|L)R$")>;
 941: 
 942: // Multiply and add / subtract
 943: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, FPU2, LSU, GroupAlone],
 944:              (instregex "M(A|S)(E|D)$")>;
 945: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "M(A|S)(E|D)R$")>;
 946: def : InstRW<[WLat11LSU, RegReadAdv, RegReadAdv, FPU4, LSU, GroupAlone],
 947:              (instregex "MAY$")>;
 948: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, FPU2, LSU, GroupAlone],
 949:              (instregex "MAY(H|L)$")>;
 950: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "MAYR$")>;
 951: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "MAY(H|L)R$")>;
 952: 
 953: // Division
 954: def : InstRW<[WLat30, RegReadAdv, FPU, LSU, NormalGr], (instregex "D(E|D)$")>;
 955: def : InstRW<[WLat30, FPU, NormalGr], (instregex "D(E|D)R$")>;
 956: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "DXR$")>;
 957: 
 958: //===----------------------------------------------------------------------===//
 959: // HFP: Comparisons
 960: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: 
 962: // Compare
 963: def : InstRW<[WLat11LSU, RegReadAdv, FPU, LSU, NormalGr], (instregex "C(E|D)$")>;
 964: def : InstRW<[WLat9, FPU, NormalGr], (instregex "C(E|D)R$")>;
 965: def : InstRW<[WLat15, FPU2, NormalGr], (instregex "CXR$")>;
 966: 
 967: 
 968: // ------------------------ Decimal floating point -------------------------- //
 969: 
 970: //===----------------------------------------------------------------------===//
 971: // DFP: Move instructions
 972: //===----------------------------------------------------------------------===//
 973: 
 974: // Load and Test
 975: def : InstRW<[WLat4, WLat4, DFU, NormalGr], (instregex "LTDTR$")>;
 976: def : InstRW<[WLat6, WLat6, DFU4, GroupAlone], (instregex "LTXTR$")>;
 977: 
 978: //===----------------------------------------------------------------------===//
 979: // DFP: Conversion instructions
 980: //===----------------------------------------------------------------------===//
 981: 
 982: // Load rounded
 983: def : InstRW<[WLat30, DFU, NormalGr], (instregex "LEDTR$")>;
 984: def : InstRW<[WLat30, DFU2, NormalGr], (instregex "LDXTR$")>;
 985: 
 986: // Load lengthened
 987: def : InstRW<[WLat7, DFU, NormalGr], (instregex "LDETR$")>;
 988: def : InstRW<[WLat6, DFU4, GroupAlone], (instregex "LXDTR$")>;
 989: 
 990: // Convert from fixed / logical
 991: def : InstRW<[WLat9, FXU, DFU, GroupAlone], (instregex "CDFTR$")>;
 992: def : InstRW<[WLat30, FXU, DFU, GroupAlone], (instregex "CDGTR(A)?$")>;
 993: def : InstRW<[WLat5, FXU, DFU4, GroupAlone2], (instregex "CXFTR(A)?$")>;
 994: def : InstRW<[WLat30, FXU, DFU4, GroupAlone2], (instregex "CXGTR(A)?$")>;
 995: def : InstRW<[WLat9, FXU, DFU, GroupAlone], (instregex "CDL(F|G)TR$")>;
 996: def : InstRW<[WLat9, FXU, DFU4, GroupAlone2], (instregex "CXLFTR$")>;
 997: def : InstRW<[WLat5, FXU, DFU4, GroupAlone2], (instregex "CXLGTR$")>;
 998: 
 999: // Convert to fixed / logical
1000: def : InstRW<[WLat11, WLat11, FXU, DFU, GroupAlone], (instregex "CFDTR(A)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: def : InstRW<[WLat30, WLat30, FXU, DFU, GroupAlone], (instregex "CGDTR(A)?$")>;
1002: def : InstRW<[WLat7, WLat7, FXU, DFU2, GroupAlone], (instregex "CFXTR$")>;
1003: def : InstRW<[WLat30, WLat30, FXU, DFU2, GroupAlone], (instregex "CGXTR(A)?$")>;
1004: def : InstRW<[WLat11, WLat11, FXU, DFU, GroupAlone], (instregex "CL(F|G)DTR$")>;
1005: def : InstRW<[WLat7, WLat7, FXU, DFU2, GroupAlone], (instregex "CL(F|G)XTR$")>;
1006: 
1007: // Convert from / to signed / unsigned packed
1008: def : InstRW<[WLat5, FXU, DFU, GroupAlone], (instregex "CD(S|U)TR$")>;
1009: def : InstRW<[WLat8, FXU2, DFU4, GroupAlone2], (instregex "CX(S|U)TR$")>;
1010: def : InstRW<[WLat7, FXU, DFU, GroupAlone], (instregex "C(S|U)DTR$")>;
1011: def : InstRW<[WLat12, FXU2, DFU4, GroupAlone2], (instregex "C(S|U)XTR$")>;
1012: 
1013: // Perform floating-point operation
1014: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "PFPO$")>;
1015: 
1016: //===----------------------------------------------------------------------===//
1017: // DFP: Unary arithmetic
1018: //===----------------------------------------------------------------------===//
1019: 
1020: // Load FP integer
1021: def : InstRW<[WLat8, DFU, NormalGr], (instregex "FIDTR$")>;
1022: def : InstRW<[WLat10, DFU4, GroupAlone], (instregex "FIXTR$")>;
1023: 
1024: // Extract biased exponent
1025: def : InstRW<[WLat7, FXU, DFU, GroupAlone], (instregex "EEDTR$")>;
1026: def : InstRW<[WLat8, FXU, DFU2, GroupAlone], (instregex "EEXTR$")>;
1027: 
1028: // Extract significance
1029: def : InstRW<[WLat7, FXU, DFU, GroupAlone], (instregex "ESDTR$")>;
1030: def : InstRW<[WLat8, FXU, DFU2, GroupAlone], (instregex "ESXTR$")>;
1031: 
1032: //===----------------------------------------------------------------------===//
1033: // DFP: Binary arithmetic
1034: //===----------------------------------------------------------------------===//
1035: 
1036: // Addition
1037: def : InstRW<[WLat9, WLat9, DFU, NormalGr], (instregex "ADTR(A)?$")>;
1038: def : InstRW<[WLat30, WLat30, DFU4, GroupAlone], (instregex "AXTR(A)?$")>;
1039: 
1040: // Subtraction
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: def : InstRW<[WLat9, WLat9, DFU, NormalGr], (instregex "SDTR(A)?$")>;
1042: def : InstRW<[WLat30, WLat30, DFU4, GroupAlone], (instregex "SXTR(A)?$")>;
1043: 
1044: // Multiply
1045: def : InstRW<[WLat30, DFU, NormalGr], (instregex "MDTR(A)?$")>;
1046: def : InstRW<[WLat30, DFU4, GroupAlone], (instregex "MXTR(A)?$")>;
1047: 
1048: // Division
1049: def : InstRW<[WLat30, DFU, NormalGr], (instregex "DDTR(A)?$")>;
1050: def : InstRW<[WLat30, DFU4, GroupAlone], (instregex "DXTR(A)?$")>;
1051: 
1052: // Quantize
1053: def : InstRW<[WLat8, WLat8, DFU, NormalGr], (instregex "QADTR$")>;
1054: def : InstRW<[WLat10, WLat10, DFU4, GroupAlone], (instregex "QAXTR$")>;
1055: 
1056: // Reround
1057: def : InstRW<[WLat11, WLat11, FXU, DFU, GroupAlone], (instregex "RRDTR$")>;
1058: def : InstRW<[WLat30, WLat30, FXU, DFU4, GroupAlone2], (instregex "RRXTR$")>;
1059: 
1060: // Shift significand left/right
1061: def : InstRW<[WLat7LSU, LSU, DFU, GroupAlone], (instregex "S(L|R)DT$")>;
1062: def : InstRW<[WLat11LSU, LSU, DFU4, GroupAlone], (instregex "S(L|R)XT$")>;
1063: 
1064: // Insert biased exponent
1065: def : InstRW<[WLat5, FXU, DFU, GroupAlone], (instregex "IEDTR$")>;
1066: def : InstRW<[WLat7, FXU, DFU4, GroupAlone2], (instregex "IEXTR$")>;
1067: 
1068: //===----------------------------------------------------------------------===//
1069: // DFP: Comparisons
1070: //===----------------------------------------------------------------------===//
1071: 
1072: // Compare
1073: def : InstRW<[WLat9, DFU, NormalGr], (instregex "(K|C)DTR$")>;
1074: def : InstRW<[WLat10, DFU2, NormalGr], (instregex "(K|C)XTR$")>;
1075: 
1076: // Compare biased exponent
1077: def : InstRW<[WLat4, DFU, NormalGr], (instregex "CEDTR$")>;
1078: def : InstRW<[WLat5, DFU2, NormalGr], (instregex "CEXTR$")>;
1079: 
1080: // Test Data Class/Group
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: def : InstRW<[WLat9, LSU, DFU, NormalGr], (instregex "TD(C|G)DT$")>;
1082: def : InstRW<[WLat10, LSU, DFU, NormalGr], (instregex "TD(C|G)ET$")>;
1083: def : InstRW<[WLat10, LSU, DFU2, NormalGr], (instregex "TD(C|G)XT$")>;
1084: 
1085: 
1086: // -------------------------------- System ---------------------------------- //
1087: 
1088: //===----------------------------------------------------------------------===//
1089: // System: Program-Status Word Instructions
1090: //===----------------------------------------------------------------------===//
1091: 
1092: def : InstRW<[WLat30, WLat30, MCD], (instregex "EPSW$")>;
1093: def : InstRW<[WLat30, MCD], (instregex "LPSW(E)?$")>;
1094: def : InstRW<[WLat3, FXU, GroupAlone], (instregex "IPK$")>;
1095: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SPKA$")>;
1096: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SSM$")>;
1097: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "ST(N|O)SM$")>;
1098: def : InstRW<[WLat3, FXU, NormalGr], (instregex "IAC$")>;
1099: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SAC(F)?$")>;
1100: 
1101: //===----------------------------------------------------------------------===//
1102: // System: Control Register Instructions
1103: //===----------------------------------------------------------------------===//
1104: 
1105: def : InstRW<[WLat10, WLat10, LSU2, GroupAlone], (instregex "LCTL(G)?$")>;
1106: def : InstRW<[WLat1, FXU5, LSU5, GroupAlone], (instregex "STCT(L|G)$")>;
1107: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "E(P|S)A(I)?R$")>;
1108: def : InstRW<[WLat30, MCD], (instregex "SSA(I)?R$")>;
1109: def : InstRW<[WLat30, MCD], (instregex "ESEA$")>;
1110: 
1111: //===----------------------------------------------------------------------===//
1112: // System: Prefix-Register Instructions
1113: //===----------------------------------------------------------------------===//
1114: 
1115: def : InstRW<[WLat30, MCD], (instregex "S(T)?PX$")>;
1116: 
1117: //===----------------------------------------------------------------------===//
1118: // System: Storage-Key and Real Memory Instructions
1119: //===----------------------------------------------------------------------===//
1120: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: def : InstRW<[WLat30, MCD], (instregex "ISKE$")>;
1122: def : InstRW<[WLat30, MCD], (instregex "IVSK$")>;
1123: def : InstRW<[WLat30, MCD], (instregex "SSKE(Opt)?$")>;
1124: def : InstRW<[WLat30, MCD], (instregex "RRB(E|M)$")>;
1125: def : InstRW<[WLat30, MCD], (instregex "PFMF$")>;
1126: def : InstRW<[WLat30, WLat30, MCD], (instregex "TB$")>;
1127: def : InstRW<[WLat30, MCD], (instregex "PGIN$")>;
1128: def : InstRW<[WLat30, MCD], (instregex "PGOUT$")>;
1129: 
1130: //===----------------------------------------------------------------------===//
1131: // System: Dynamic-Address-Translation Instructions
1132: //===----------------------------------------------------------------------===//
1133: 
1134: def : InstRW<[WLat30, MCD], (instregex "IPTE(Opt)?(Opt)?$")>;
1135: def : InstRW<[WLat30, MCD], (instregex "IDTE(Opt)?$")>;
1136: def : InstRW<[WLat30, MCD], (instregex "PTLB$")>;
1137: def : InstRW<[WLat30, WLat30, MCD], (instregex "CSP(G)?$")>;
1138: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "LPTEA$")>;
1139: def : InstRW<[WLat30, WLat30, MCD], (instregex "LRA(Y|G)?$")>;
1140: def : InstRW<[WLat30, MCD], (instregex "STRAG$")>;
1141: def : InstRW<[WLat30, MCD], (instregex "LURA(G)?$")>;
1142: def : InstRW<[WLat30, MCD], (instregex "STUR(A|G)$")>;
1143: def : InstRW<[WLat30, MCD], (instregex "TPROT$")>;
1144: 
1145: //===----------------------------------------------------------------------===//
1146: // System: Memory-move Instructions
1147: //===----------------------------------------------------------------------===//
1148: 
1149: def : InstRW<[WLat30, MCD], (instregex "MVC(K|P|S)$")>;
1150: def : InstRW<[WLat30, MCD], (instregex "MVC(S|D)K$")>;
1151: def : InstRW<[WLat30, MCD], (instregex "MVCOS$")>;
1152: def : InstRW<[WLat30, MCD], (instregex "MVPG$")>;
1153: 
1154: //===----------------------------------------------------------------------===//
1155: // System: Address-Space Instructions
1156: //===----------------------------------------------------------------------===//
1157: 
1158: def : InstRW<[WLat30, MCD], (instregex "LASP$")>;
1159: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "PALB$")>;
1160: def : InstRW<[WLat30, MCD], (instregex "PC$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161: def : InstRW<[WLat30, MCD], (instregex "PR$")>;
1162: def : InstRW<[WLat30, MCD], (instregex "PT(I)?$")>;
1163: def : InstRW<[WLat30, MCD], (instregex "RP$")>;
1164: def : InstRW<[WLat30, MCD], (instregex "BS(G|A)$")>;
1165: def : InstRW<[WLat30, MCD], (instregex "TAR$")>;
1166: 
1167: //===----------------------------------------------------------------------===//
1168: // System: Linkage-Stack Instructions
1169: //===----------------------------------------------------------------------===//
1170: 
1171: def : InstRW<[WLat30, MCD], (instregex "BAKR$")>;
1172: def : InstRW<[WLat30, MCD], (instregex "EREG(G)?$")>;
1173: def : InstRW<[WLat30, WLat30, MCD], (instregex "(E|M)STA$")>;
1174: 
1175: //===----------------------------------------------------------------------===//
1176: // System: Time-Related Instructions
1177: //===----------------------------------------------------------------------===//
1178: 
1179: def : InstRW<[WLat30, MCD], (instregex "PTFF$")>;
1180: def : InstRW<[WLat30, MCD], (instregex "SCK$")>;
1181: def : InstRW<[WLat30, MCD], (instregex "SCKPF$")>;
1182: def : InstRW<[WLat30, MCD], (instregex "SCKC$")>;
1183: def : InstRW<[WLat30, MCD], (instregex "SPT$")>;
1184: def : InstRW<[WLat30, MCD], (instregex "STCK(F)?$")>;
1185: def : InstRW<[WLat30, MCD], (instregex "STCKE$")>;
1186: def : InstRW<[WLat30, MCD], (instregex "STCKC$")>;
1187: def : InstRW<[WLat30, MCD], (instregex "STPT$")>;
1188: 
1189: //===----------------------------------------------------------------------===//
1190: // System: CPU-Related Instructions
1191: //===----------------------------------------------------------------------===//
1192: 
1193: def : InstRW<[WLat30, MCD], (instregex "STAP$")>;
1194: def : InstRW<[WLat30, MCD], (instregex "STIDP$")>;
1195: def : InstRW<[WLat30, WLat30, MCD], (instregex "STSI$")>;
1196: def : InstRW<[WLat30, WLat30, MCD], (instregex "STFL(E)?$")>;
1197: def : InstRW<[WLat30, MCD], (instregex "ECAG$")>;
1198: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECTG$")>;
1199: def : InstRW<[WLat30, MCD], (instregex "PTF$")>;
1200: def : InstRW<[WLat30, MCD], (instregex "PCKMO$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201: 
1202: //===----------------------------------------------------------------------===//
1203: // System: Miscellaneous Instructions
1204: //===----------------------------------------------------------------------===//
1205: 
1206: def : InstRW<[WLat30, MCD], (instregex "SVC$")>;
1207: def : InstRW<[WLat1, FXU, GroupAlone], (instregex "MC$")>;
1208: def : InstRW<[WLat30, MCD], (instregex "DIAG$")>;
1209: def : InstRW<[WLat30, MCD], (instregex "TRAC(E|G)$")>;
1210: def : InstRW<[WLat30, MCD], (instregex "TRAP(2|4)$")>;
1211: def : InstRW<[WLat30, MCD], (instregex "SIG(P|A)$")>;
1212: def : InstRW<[WLat30, MCD], (instregex "SIE$")>;
1213: 
1214: //===----------------------------------------------------------------------===//
1215: // System: CPU-Measurement Facility Instructions
1216: //===----------------------------------------------------------------------===//
1217: 
1218: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LPP$")>;
1219: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECPGA$")>;
1220: def : InstRW<[WLat30, WLat30, MCD], (instregex "E(C|P)CTR$")>;
1221: def : InstRW<[WLat30, MCD], (instregex "LCCTL$")>;
1222: def : InstRW<[WLat30, MCD], (instregex "L(P|S)CTL$")>;
1223: def : InstRW<[WLat30, MCD], (instregex "Q(S|CTR)I$")>;
1224: def : InstRW<[WLat30, MCD], (instregex "S(C|P)CTR$")>;
1225: 
1226: //===----------------------------------------------------------------------===//
1227: // System: I/O Instructions
1228: //===----------------------------------------------------------------------===//
1229: 
1230: def : InstRW<[WLat30, MCD], (instregex "(C|H|R|X)SCH$")>;
1231: def : InstRW<[WLat30, MCD], (instregex "(M|S|ST|T)SCH$")>;
1232: def : InstRW<[WLat30, MCD], (instregex "RCHP$")>;
1233: def : InstRW<[WLat30, MCD], (instregex "SCHM$")>;
1234: def : InstRW<[WLat30, MCD], (instregex "STC(PS|RW)$")>;
1235: def : InstRW<[WLat30, MCD], (instregex "TPI$")>;
1236: def : InstRW<[WLat30, MCD], (instregex "SAL$")>;
1237: 
1238: //===----------------------------------------------------------------------===//
1239: // NOPs
1240: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1241-1245 / 第 1241-1245 行
```tablegen
1241: 
1242: def : InstRW<[WLat1, LSU, EndGroup], (instregex "NOP(R)?(Opt)?$")>;
1243: def : InstRW<[WLat1, LSU, EndGroup], (instregex "J(G)?NOP$")>;
1244: }
1245: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
