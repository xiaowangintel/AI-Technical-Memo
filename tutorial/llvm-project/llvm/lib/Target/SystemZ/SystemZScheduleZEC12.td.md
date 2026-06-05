# SystemZScheduleZEC12.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZScheduleZEC12.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //=- SystemZScheduleZEC12.td - SystemZ Scheduling Definitions --*- tblgen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the machine model for ZEC12 to support instruction
  10: // scheduling and other instruction cost heuristics.
  11: //
  12: // Pseudos expanded right after isel do not need to be modelled here.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: def ZEC12Model : SchedMachineModel {
  17: 
  18:     let UnsupportedFeatures = Arch10UnsupportedFeatures.List;
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
  30: let SchedModel = ZEC12Model in {
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
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `ZEC12Model`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `ZEC12Model` 等 TableGen 记录。

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
  73: def ZEC12_FXUnit : ProcResource<2>;
  74: def ZEC12_LSUnit : ProcResource<2>;
  75: def ZEC12_FPUnit : ProcResource<1>;
  76: def ZEC12_DFUnit : ProcResource<1>;
  77: def ZEC12_VBUnit : ProcResource<1>;
  78: def ZEC12_MCD    : ProcResource<1>;
  79: 
  80: // Subtarget specific definitions of scheduling resources.
```
- **EN**: This block declares or refines TableGen records such as `ZEC12_FXUnit`, `ZEC12_LSUnit`, `ZEC12_FPUnit`, `ZEC12_DFUnit`, `ZEC12_VBUnit`, `ZEC12_MCD`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `ZEC12_FXUnit`, `ZEC12_LSUnit`, `ZEC12_FPUnit`, `ZEC12_DFUnit`, `ZEC12_VBUnit`, `ZEC12_MCD` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 81-120 / 第 81-120 行
```tablegen
  81: let NumMicroOps = 0 in {
  82:   def : WriteRes<FXU, [ZEC12_FXUnit]>;
  83:   def : WriteRes<LSU, [ZEC12_LSUnit]>;
  84:   def : WriteRes<FPU, [ZEC12_FPUnit]>;
  85:   def : WriteRes<DFU, [ZEC12_DFUnit]>;
  86:   foreach Num = 2-6 in { let ReleaseAtCycles = [Num] in {
  87:     def : WriteRes<!cast<SchedWrite>("FXU"#Num), [ZEC12_FXUnit]>;
  88:     def : WriteRes<!cast<SchedWrite>("LSU"#Num), [ZEC12_LSUnit]>;
  89:     def : WriteRes<!cast<SchedWrite>("FPU"#Num), [ZEC12_FPUnit]>;
  90:     def : WriteRes<!cast<SchedWrite>("DFU"#Num), [ZEC12_DFUnit]>;
  91:   }}
  92: 
  93:   def : WriteRes<VBU,  [ZEC12_VBUnit]>; // Virtual Branching Unit
  94: }
  95: 
  96: def : WriteRes<MCD, [ZEC12_MCD]> { let NumMicroOps = 3;
  97:                                    let BeginGroup  = 1;
  98:                                    let EndGroup    = 1; }
  99: 
 100: // -------------------------- INSTRUCTIONS ---------------------------------- //
 101: 
 102: // InstRW constructs have been used in order to preserve the
 103: // readability of the InstrInfo files.
 104: 
 105: // For each instruction, as matched by a regexp, provide a list of
 106: // resources that it needs. These will be combined into a SchedClass.
 107: 
 108: //===----------------------------------------------------------------------===//
 109: // Stack allocation
 110: //===----------------------------------------------------------------------===//
 111: 
 112: // Pseudo -> LA / LAY
 113: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ADJDYNALLOC$")>;
 114: 
 115: //===----------------------------------------------------------------------===//
 116: // Branch instructions
 117: //===----------------------------------------------------------------------===//
 118: 
 119: // Branch
 120: def : InstRW<[WLat1, VBU, NormalGr], (instregex "(Call)?BRC(L)?(Asm.*)?$")>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 121-160 / 第 121-160 行
```tablegen
 121: def : InstRW<[WLat1, VBU, NormalGr], (instregex "(Call)?J(G)?(Asm.*)?$")>;
 122: def : InstRW<[WLat1, LSU, NormalGr], (instregex "(Call)?BC(R)?(Asm.*)?$")>;
 123: def : InstRW<[WLat1, LSU, NormalGr], (instregex "(Call)?B(R)?(Asm.*)?$")>;
 124: def : InstRW<[WLat1, FXU, EndGroup], (instregex "BRCT(G)?$")>;
 125: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "BRCTH$")>;
 126: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "BCT(G)?(R)?$")>;
 127: def : InstRW<[WLat1, FXU3, LSU, GroupAlone2],
 128:              (instregex "B(R)?X(H|L).*$")>;
 129: 
 130: // Compare and branch
 131: def : InstRW<[WLat1, FXU, NormalGr], (instregex "C(L)?(G)?(I|R)J(Asm.*)?$")>;
 132: def : InstRW<[WLat1, FXU, LSU, GroupAlone],
 133:              (instregex "C(L)?(G)?(I|R)B(Call|Return|Asm.*)?$")>;
 134: 
 135: //===----------------------------------------------------------------------===//
 136: // Trap instructions
 137: //===----------------------------------------------------------------------===//
 138: 
 139: // Trap
 140: def : InstRW<[WLat1, VBU, NormalGr], (instregex "(Cond)?Trap$")>;
 141: 
 142: // Compare and trap
 143: def : InstRW<[WLat1, FXU, NormalGr], (instregex "C(G)?(I|R)T(Asm.*)?$")>;
 144: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CL(G)?RT(Asm.*)?$")>;
 145: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CL(F|G)IT(Asm.*)?$")>;
 146: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "CL(G)?T(Asm.*)?$")>;
 147: 
 148: //===----------------------------------------------------------------------===//
 149: // Call and return instructions
 150: //===----------------------------------------------------------------------===//
 151: 
 152: // Call
 153: def : InstRW<[WLat1, FXU2, VBU, GroupAlone], (instregex "(Call)?BRAS$")>;
 154: def : InstRW<[WLat1, FXU2, LSU, GroupAlone], (instregex "(Call)?BRASL(_XPLINK64)?$")>;
 155: def : InstRW<[WLat1, FXU2, LSU, GroupAlone], (instregex "(Call)?BAS(R)?(_XPLINK64|_STACKEXT)?$")>;
 156: def : InstRW<[WLat1, FXU2, LSU, GroupAlone], (instregex "TLS_(G|L)DCALL$")>;
 157: 
 158: // Return
 159: def : InstRW<[WLat1, LSU, EndGroup], (instregex "Return(_XPLINK)?$")>;
 160: def : InstRW<[WLat1, LSU, NormalGr], (instregex "CondReturn(_XPLINK)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 161-200 / 第 161-200 行
```tablegen
 161: 
 162: //===----------------------------------------------------------------------===//
 163: // Move instructions
 164: //===----------------------------------------------------------------------===//
 165: 
 166: // Moves
 167: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "MV(G|H)?HI$")>;
 168: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "MVI(Y)?$")>;
 169: 
 170: // Move character
 171: def : InstRW<[WLat1, FXU, LSU3, GroupAlone], (instregex "MVC$")>;
 172: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "MVCL(E|U)?$")>;
 173: 
 174: // Pseudo -> reg move
 175: def : InstRW<[WLat1, FXU, NormalGr], (instregex "COPY(_TO_REGCLASS)?$")>;
 176: def : InstRW<[WLat1, FXU, NormalGr], (instregex "EXTRACT_SUBREG$")>;
 177: def : InstRW<[WLat1, FXU, NormalGr], (instregex "INSERT_SUBREG$")>;
 178: def : InstRW<[WLat1, FXU, NormalGr], (instregex "REG_SEQUENCE$")>;
 179: 
 180: // Loads
 181: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L(Y|FH|RL|Mux)?$")>;
 182: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LG(RL)?$")>;
 183: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L128$")>;
 184: 
 185: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLIH(F|H|L)$")>;
 186: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLIL(F|H|L)$")>;
 187: 
 188: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LG(F|H)I$")>;
 189: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LHI(Mux)?$")>;
 190: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LR$")>;
 191: 
 192: // Load and trap
 193: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "L(FH|G)?AT$")>;
 194: 
 195: // Load and test
 196: def : InstRW<[WLat1LSU, WLat1LSU, LSU, FXU, NormalGr], (instregex "LT(G)?$")>;
 197: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LT(G)?R$")>;
 198: 
 199: // Stores
 200: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STG(RL)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 201-240 / 第 201-240 行
```tablegen
 201: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "ST128$")>;
 202: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "ST(Y|FH|RL|Mux)?$")>;
 203: 
 204: // String moves.
 205: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "MVST$")>;
 206: 
 207: //===----------------------------------------------------------------------===//
 208: // Conditional move instructions
 209: //===----------------------------------------------------------------------===//
 210: 
 211: def : InstRW<[WLat2, FXU, NormalGr], (instregex "LOC(G)?R(Asm.*)?$")>;
 212: def : InstRW<[WLat2LSU, RegReadAdv, FXU, LSU, NormalGr],
 213:              (instregex "LOC(G)?(Asm.*)?$")>;
 214: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STOC(G)?(Asm.*)?$")>;
 215: 
 216: //===----------------------------------------------------------------------===//
 217: // Sign extensions
 218: //===----------------------------------------------------------------------===//
 219: 
 220: def : InstRW<[WLat1, FXU, NormalGr], (instregex "L(B|H|G)R$")>;
 221: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LG(B|H|F)R$")>;
 222: 
 223: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LTGF$")>;
 224: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LTGFR$")>;
 225: 
 226: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LB(H|Mux)?$")>;
 227: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LH(Y)?$")>;
 228: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LH(H|Mux|RL)$")>;
 229: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LG(B|H|F)$")>;
 230: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LG(H|F)RL$")>;
 231: 
 232: //===----------------------------------------------------------------------===//
 233: // Zero extensions
 234: //===----------------------------------------------------------------------===//
 235: 
 236: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLCR(Mux)?$")>;
 237: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLHR(Mux)?$")>;
 238: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LLG(C|H|F|T)R$")>;
 239: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLC(Mux)?$")>;
 240: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLH(Mux)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 241-280 / 第 241-280 行
```tablegen
 241: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LL(C|H)H$")>;
 242: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLHRL$")>;
 243: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLG(C|H|F|T|HRL|FRL)$")>;
 244: 
 245: // Load and trap
 246: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LLG(F|T)?AT$")>;
 247: 
 248: //===----------------------------------------------------------------------===//
 249: // Truncations
 250: //===----------------------------------------------------------------------===//
 251: 
 252: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STC(H|Y|Mux)?$")>;
 253: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STH(H|Y|RL|Mux)?$")>;
 254: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STCM(H|Y)?$")>;
 255: 
 256: //===----------------------------------------------------------------------===//
 257: // Multi-register moves
 258: //===----------------------------------------------------------------------===//
 259: 
 260: // Load multiple (estimated average of 5 ops)
 261: def : InstRW<[WLat10, WLat10, LSU5, GroupAlone], (instregex "LM(H|Y|G)?$")>;
 262: 
 263: // Load multiple disjoint
 264: def : InstRW<[WLat30, WLat30, MCD], (instregex "LMD$")>;
 265: 
 266: // Store multiple (estimated average of 3 ops)
 267: def : InstRW<[WLat1, LSU2, FXU5, GroupAlone], (instregex "STM(H|Y|G)?$")>;
 268: 
 269: //===----------------------------------------------------------------------===//
 270: // Byte swaps
 271: //===----------------------------------------------------------------------===//
 272: 
 273: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LRV(G)?R$")>;
 274: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "LRV(G|H)?$")>;
 275: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STRV(G|H)?$")>;
 276: def : InstRW<[WLat30, MCD], (instregex "MVCIN$")>;
 277: 
 278: //===----------------------------------------------------------------------===//
 279: // Load address instructions
 280: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 281-320 / 第 281-320 行
```tablegen
 281: 
 282: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LA(Y|RL)?$")>;
 283: 
 284: // Load the Global Offset Table address
 285: def : InstRW<[WLat1, FXU, NormalGr], (instregex "GOT$")>;
 286: 
 287: //===----------------------------------------------------------------------===//
 288: // Absolute and Negation
 289: //===----------------------------------------------------------------------===//
 290: 
 291: def : InstRW<[WLat2, WLat2, FXU, NormalGr], (instregex "LP(G)?R$")>;
 292: def : InstRW<[WLat3, WLat3, FXU2, GroupAlone], (instregex "L(N|P)GFR$")>;
 293: def : InstRW<[WLat2, WLat2, FXU, NormalGr], (instregex "LN(R|GR)$")>;
 294: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LC(R|GR)$")>;
 295: def : InstRW<[WLat2, WLat2, FXU2, GroupAlone], (instregex "LCGFR$")>;
 296: 
 297: //===----------------------------------------------------------------------===//
 298: // Insertion
 299: //===----------------------------------------------------------------------===//
 300: 
 301: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "IC(Y)?$")>;
 302: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 303:              (instregex "IC32(Y)?$")>;
 304: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 305:              (instregex "ICM(H|Y)?$")>;
 306: def : InstRW<[WLat1, FXU, NormalGr], (instregex "II(F|H|L)Mux$")>;
 307: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IIHF(64)?$")>;
 308: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IIHH(64)?$")>;
 309: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IIHL(64)?$")>;
 310: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IILF(64)?$")>;
 311: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IILH(64)?$")>;
 312: def : InstRW<[WLat1, FXU, NormalGr], (instregex "IILL(64)?$")>;
 313: 
 314: //===----------------------------------------------------------------------===//
 315: // Addition
 316: //===----------------------------------------------------------------------===//
 317: 
 318: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 319:              (instregex "A(L)?(Y)?$")>;
 320: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "A(L)?SI$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 321-360 / 第 321-360 行
```tablegen
 321: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU, LSU, NormalGr],
 322:              (instregex "AH(Y)?$")>;
 323: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AIH$")>;
 324: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AFI(Mux)?$")>;
 325: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AGFI$")>;
 326: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AGHI(K)?$")>;
 327: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AGR(K)?$")>;
 328: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AHI(K)?$")>;
 329: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AHIMux(K)?$")>;
 330: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AL(FI|HSIK)$")>;
 331: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 332:              (instregex "ALGF$")>;
 333: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALGHSIK$")>;
 334: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALGF(I|R)$")>;
 335: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALGR(K)?$")>;
 336: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALR(K)?$")>;
 337: def : InstRW<[WLat1, FXU, NormalGr], (instregex "AR(K)?$")>;
 338: def : InstRW<[WLat1, FXU, NormalGr], (instregex "A(L)?HHHR$")>;
 339: def : InstRW<[WLat2, WLat2, FXU, NormalGr], (instregex "A(L)?HHLR$")>;
 340: def : InstRW<[WLat1, FXU, NormalGr], (instregex "ALSIH(N)?$")>;
 341: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 342:              (instregex "A(L)?G$")>;
 343: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "A(L)?GSI$")>;
 344: 
 345: // Logical addition with carry
 346: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU, LSU, GroupAlone],
 347:              (instregex "ALC(G)?$")>;
 348: def : InstRW<[WLat2, WLat2, FXU, GroupAlone], (instregex "ALC(G)?R$")>;
 349: 
 350: // Add with sign extension (32 -> 64)
 351: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU, LSU, NormalGr],
 352:              (instregex "AGF$")>;
 353: def : InstRW<[WLat2, WLat2, FXU, NormalGr], (instregex "AGFR$")>;
 354: 
 355: //===----------------------------------------------------------------------===//
 356: // Subtraction
 357: //===----------------------------------------------------------------------===//
 358: 
 359: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 360:              (instregex "S(G|Y)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 361-400 / 第 361-400 行
```tablegen
 361: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU, LSU, NormalGr],
 362:              (instregex "SH(Y)?$")>;
 363: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SGR(K)?$")>;
 364: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLFI$")>;
 365: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 366:              (instregex "SL(G|GF|Y)?$")>;
 367: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLGF(I|R)$")>;
 368: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLGR(K)?$")>;
 369: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLR(K)?$")>;
 370: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SR(K)?$")>;
 371: def : InstRW<[WLat1, FXU, NormalGr], (instregex "S(L)?HHHR$")>;
 372: def : InstRW<[WLat2, WLat2, FXU, NormalGr], (instregex "S(L)?HHLR$")>;
 373: 
 374: // Subtraction with borrow
 375: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU, LSU, GroupAlone],
 376:              (instregex "SLB(G)?$")>;
 377: def : InstRW<[WLat2, WLat2, FXU, GroupAlone], (instregex "SLB(G)?R$")>;
 378: 
 379: // Subtraction with sign extension (32 -> 64)
 380: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXU, LSU, NormalGr],
 381:              (instregex "SGF$")>;
 382: def : InstRW<[WLat2, WLat2, FXU, NormalGr], (instregex "SGFR$")>;
 383: 
 384: //===----------------------------------------------------------------------===//
 385: // AND
 386: //===----------------------------------------------------------------------===//
 387: 
 388: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 389:              (instregex "N(G|Y)?$")>;
 390: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NGR(K)?$")>;
 391: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NI(FMux|HMux|LMux)$")>;
 392: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "NI(Y)?$")>;
 393: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NIHF(64)?$")>;
 394: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NIHH(64)?$")>;
 395: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NIHL(64)?$")>;
 396: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NILF(64)?$")>;
 397: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NILH(64)?$")>;
 398: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NILL(64)?$")>;
 399: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NR(K)?$")>;
 400: def : InstRW<[WLat5LSU, LSU2, FXU, GroupAlone], (instregex "NC$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 401-440 / 第 401-440 行
```tablegen
 401: 
 402: //===----------------------------------------------------------------------===//
 403: // OR
 404: //===----------------------------------------------------------------------===//
 405: 
 406: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 407:              (instregex "O(G|Y)?$")>;
 408: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OGR(K)?$")>;
 409: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "OI(Y)?$")>;
 410: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OI(FMux|HMux|LMux)$")>;
 411: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OIHF(64)?$")>;
 412: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OIHH(64)?$")>;
 413: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OIHL(64)?$")>;
 414: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OILF(64)?$")>;
 415: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OILH(64)?$")>;
 416: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OILL(64)?$")>;
 417: def : InstRW<[WLat1, FXU, NormalGr], (instregex "OR(K)?$")>;
 418: def : InstRW<[WLat5LSU, LSU2, FXU, GroupAlone], (instregex "OC$")>;
 419: 
 420: //===----------------------------------------------------------------------===//
 421: // XOR
 422: //===----------------------------------------------------------------------===//
 423: 
 424: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 425:              (instregex "X(G|Y)?$")>;
 426: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "XI(Y)?$")>;
 427: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XIFMux$")>;
 428: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XGR(K)?$")>;
 429: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XIHF(64)?$")>;
 430: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XILF(64)?$")>;
 431: def : InstRW<[WLat1, FXU, NormalGr], (instregex "XR(K)?$")>;
 432: def : InstRW<[WLat5LSU, LSU2, FXU, GroupAlone], (instregex "XC$")>;
 433: 
 434: //===----------------------------------------------------------------------===//
 435: // Multiplication
 436: //===----------------------------------------------------------------------===//
 437: 
 438: def : InstRW<[WLat6LSU, RegReadAdv, FXU, LSU, NormalGr],
 439:              (instregex "MS(GF|Y)?$")>;
 440: def : InstRW<[WLat6, FXU, NormalGr], (instregex "MS(R|FI)$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 441-480 / 第 441-480 行
```tablegen
 441: def : InstRW<[WLat8LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "MSG$")>;
 442: def : InstRW<[WLat8, FXU, NormalGr], (instregex "MSGR$")>;
 443: def : InstRW<[WLat6, FXU, NormalGr], (instregex "MSGF(I|R)$")>;
 444: def : InstRW<[WLat11LSU, RegReadAdv, FXU2, LSU, GroupAlone],
 445:              (instregex "MLG$")>;
 446: def : InstRW<[WLat9, FXU2, GroupAlone], (instregex "MLGR$")>;
 447: def : InstRW<[WLat5, FXU, NormalGr], (instregex "MGHI$")>;
 448: def : InstRW<[WLat5, FXU, NormalGr], (instregex "MHI$")>;
 449: def : InstRW<[WLat5LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "MH(Y)?$")>;
 450: def : InstRW<[WLat7, FXU2, GroupAlone], (instregex "M(L)?R$")>;
 451: def : InstRW<[WLat7LSU, RegReadAdv, FXU2, LSU, GroupAlone],
 452:              (instregex "M(FY|L)?$")>;
 453: 
 454: //===----------------------------------------------------------------------===//
 455: // Division and remainder
 456: //===----------------------------------------------------------------------===//
 457: 
 458: def : InstRW<[WLat30, FPU4, FXU5, GroupAlone3], (instregex "DR$")>;
 459: def : InstRW<[WLat30, RegReadAdv, FPU4, LSU, FXU4, GroupAlone3],
 460:              (instregex "D$")>;
 461: def : InstRW<[WLat30, FPU4, FXU4, GroupAlone3], (instregex "DSG(F)?R$")>;
 462: def : InstRW<[WLat30, RegReadAdv, FPU4, LSU, FXU3, GroupAlone3],
 463:              (instregex "DSG(F)?$")>;
 464: def : InstRW<[WLat30, FPU4, FXU5, GroupAlone3], (instregex "DL(G)?R$")>;
 465: def : InstRW<[WLat30, RegReadAdv, FPU4, LSU, FXU4, GroupAlone3],
 466:              (instregex "DL(G)?$")>;
 467: 
 468: //===----------------------------------------------------------------------===//
 469: // Shifts
 470: //===----------------------------------------------------------------------===//
 471: 
 472: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLL(G|K)?$")>;
 473: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SRL(G|K)?$")>;
 474: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SRA(G|K)?$")>;
 475: def : InstRW<[WLat1, FXU, NormalGr], (instregex "SLA(G|K)?$")>;
 476: def : InstRW<[WLat5LSU, WLat5LSU, FXU4, LSU, GroupAlone2],
 477:              (instregex "S(L|R)D(A|L)$")>;
 478: 
 479: // Rotate
 480: def : InstRW<[WLat2LSU, FXU, LSU, NormalGr], (instregex "RLL(G)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: 
 482: // Rotate and insert
 483: def : InstRW<[WLat1, FXU, NormalGr], (instregex "RISBH(G|H|L)(Opt)?$")>;
 484: def : InstRW<[WLat1, FXU, NormalGr], (instregex "RISBL(G|H|L)(Opt)?$")>;
 485: def : InstRW<[WLat1, FXU, NormalGr], (instregex "RISBG(N|32)?(Z)?(Opt)?$")>;
 486: def : InstRW<[WLat1, FXU, NormalGr], (instregex "RISBMux$")>;
 487: 
 488: // Rotate and Select
 489: def : InstRW<[WLat3, WLat3, FXU2, GroupAlone], (instregex "R(N|O|X)SBG(Opt)?$")>;
 490: 
 491: //===----------------------------------------------------------------------===//
 492: // Comparison
 493: //===----------------------------------------------------------------------===//
 494: 
 495: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "C(G|Y|Mux|RL)?$")>;
 496: def : InstRW<[WLat1, FXU, NormalGr], (instregex "C(F|H)I(Mux)?$")>;
 497: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CG(F|H)I$")>;
 498: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CG(HSI|RL)$")>;
 499: def : InstRW<[WLat1, FXU, NormalGr], (instregex "C(G)?R$")>;
 500: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CIH$")>;
 501: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CHF$")>;
 502: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CHSI$")>;
 503: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr],
 504:              (instregex "CL(Y|Mux)?$")>;
 505: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLFHSI$")>;
 506: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLFI(Mux)?$")>;
 507: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CLG$")>;
 508: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLG(HRL|HSI)$")>;
 509: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CLGF$")>;
 510: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLGFRL$")>;
 511: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLGF(I|R)$")>;
 512: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLGR$")>;
 513: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLGRL$")>;
 514: def : InstRW<[WLat1LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CLHF$")>;
 515: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLH(RL|HSI)$")>;
 516: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLIH$")>;
 517: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLI(Y)?$")>;
 518: def : InstRW<[WLat1, FXU, NormalGr], (instregex "CLR$")>;
 519: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "CLRL$")>;
 520: def : InstRW<[WLat1, FXU, NormalGr], (instregex "C(L)?HHR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 521-560 / 第 521-560 行
```tablegen
 521: def : InstRW<[WLat2, FXU, NormalGr], (instregex "C(L)?HLR$")>;
 522: 
 523: // Compare halfword
 524: def : InstRW<[WLat2LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CH(Y)?$")>;
 525: def : InstRW<[WLat2LSU, FXU, LSU, NormalGr], (instregex "CHRL$")>;
 526: def : InstRW<[WLat2LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CGH$")>;
 527: def : InstRW<[WLat2LSU, FXU, LSU, NormalGr], (instregex "CGHRL$")>;
 528: def : InstRW<[WLat2LSU, FXU2, LSU, GroupAlone], (instregex "CHHSI$")>;
 529: 
 530: // Compare with sign extension (32 -> 64)
 531: def : InstRW<[WLat2LSU, RegReadAdv, FXU, LSU, NormalGr], (instregex "CGF$")>;
 532: def : InstRW<[WLat2LSU, FXU, LSU, NormalGr], (instregex "CGFRL$")>;
 533: def : InstRW<[WLat2, FXU, NormalGr], (instregex "CGFR$")>;
 534: 
 535: // Compare logical character
 536: def : InstRW<[WLat9, FXU, LSU2, GroupAlone], (instregex "CLC$")>;
 537: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLCL(E|U)?$")>;
 538: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLST$")>;
 539: 
 540: // Test under mask
 541: def : InstRW<[WLat1LSU, FXU, LSU, NormalGr], (instregex "TM(Y)?$")>;
 542: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TM(H|L)Mux$")>;
 543: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TMHH(64)?$")>;
 544: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TMHL(64)?$")>;
 545: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TMLH(64)?$")>;
 546: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TMLL(64)?$")>;
 547: 
 548: // Compare logical characters under mask
 549: def : InstRW<[WLat2LSU, RegReadAdv, FXU, LSU, NormalGr],
 550:              (instregex "CLM(H|Y)?$")>;
 551: 
 552: //===----------------------------------------------------------------------===//
 553: // Prefetch and execution hint
 554: //===----------------------------------------------------------------------===//
 555: 
 556: def : InstRW<[WLat1, LSU, NormalGr], (instregex "PFD(RL)?$")>;
 557: def : InstRW<[WLat1, LSU, NormalGr], (instregex "BP(R)?P$")>;
 558: def : InstRW<[WLat1, FXU, NormalGr], (instregex "NIAI$")>;
 559: 
 560: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: // Atomic operations
 562: //===----------------------------------------------------------------------===//
 563: 
 564: def : InstRW<[WLat1, LSU, EndGroup], (instregex "Serialize$")>;
 565: 
 566: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAA(G)?$")>;
 567: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAAL(G)?$")>;
 568: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAN(G)?$")>;
 569: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAO(G)?$")>;
 570: def : InstRW<[WLat1LSU, WLat1LSU, FXU, LSU, NormalGr], (instregex "LAX(G)?$")>;
 571: 
 572: // Test and set
 573: def : InstRW<[WLat1LSU, FXU, LSU, EndGroup], (instregex "TS$")>;
 574: 
 575: // Compare and swap
 576: def : InstRW<[WLat2LSU, WLat2LSU, FXU2, LSU, GroupAlone],
 577:              (instregex "CS(G|Y)?$")>;
 578: 
 579: // Compare double and swap
 580: def : InstRW<[WLat5LSU, WLat5LSU, FXU5, LSU, GroupAlone2],
 581:              (instregex "CDS(Y)?$")>;
 582: def : InstRW<[WLat12, WLat12, FXU6, LSU2, GroupAlone],
 583:              (instregex "CDSG$")>;
 584: 
 585: // Compare and swap and store
 586: def : InstRW<[WLat30, MCD], (instregex "CSST$")>;
 587: 
 588: // Perform locked operation
 589: def : InstRW<[WLat30, MCD], (instregex "PLO$")>;
 590: 
 591: // Load/store pair from/to quadword
 592: def : InstRW<[WLat4LSU, LSU2, GroupAlone], (instregex "LPQ$")>;
 593: def : InstRW<[WLat1, FXU2, LSU2, GroupAlone], (instregex "STPQ$")>;
 594: 
 595: // Load pair disjoint
 596: def : InstRW<[WLat2LSU, WLat2LSU, LSU2, GroupAlone], (instregex "LPD(G)?$")>;
 597: 
 598: //===----------------------------------------------------------------------===//
 599: // Translate and convert
 600: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 601-640 / 第 601-640 行
```tablegen
 601: 
 602: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "TR$")>;
 603: def : InstRW<[WLat30, WLat30, WLat30, FXU3, LSU2, GroupAlone2],
 604:              (instregex "TRT$")>;
 605: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRTR$")>;
 606: def : InstRW<[WLat30, WLat30, MCD], (instregex "TRE$")>;
 607: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRT(R)?E(Opt)?$")>;
 608: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TR(T|O)(T|O)(Opt)?$")>;
 609: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 610:              (instregex "CU(12|14|21|24|41|42)(Opt)?$")>;
 611: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "(CUUTF|CUTFU)(Opt)?$")>;
 612: 
 613: //===----------------------------------------------------------------------===//
 614: // Message-security assist
 615: //===----------------------------------------------------------------------===//
 616: 
 617: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD],
 618:              (instregex "KM(C|F|O|CTR)?$")>;
 619: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "(KIMD|KLMD|KMAC|PCC)$")>;
 620: 
 621: //===----------------------------------------------------------------------===//
 622: // Decimal arithmetic
 623: //===----------------------------------------------------------------------===//
 624: 
 625: def : InstRW<[WLat30, RegReadAdv, FXU, DFU2, LSU2, GroupAlone2],
 626:              (instregex "CVBG$")>;
 627: def : InstRW<[WLat20, RegReadAdv, FXU, DFU, LSU, GroupAlone],
 628:              (instregex "CVB(Y)?$")>;
 629: def : InstRW<[WLat1, FXU3, DFU4, LSU, GroupAlone3], (instregex "CVDG$")>;
 630: def : InstRW<[WLat1, FXU2, DFU, LSU, GroupAlone], (instregex "CVD(Y)?$")>;
 631: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "MV(N|O|Z)$")>;
 632: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "(PACK|PKA|PKU)$")>;
 633: def : InstRW<[WLat10, LSU5, GroupAlone], (instregex "UNPK(A|U)$")>;
 634: def : InstRW<[WLat1, FXU, LSU2, GroupAlone], (instregex "UNPK$")>;
 635: 
 636: def : InstRW<[WLat11LSU, FXU, DFU4, LSU2, GroupAlone],
 637:              (instregex "(A|S|ZA)P$")>;
 638: def : InstRW<[WLat1, FXU, DFU4, LSU2, GroupAlone], (instregex "(M|D)P$")>;
 639: def : InstRW<[WLat15, FXU2, DFU4, LSU3, GroupAlone], (instregex "SRP$")>;
 640: def : InstRW<[WLat11, DFU4, LSU2, GroupAlone], (instregex "CP$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 641-680 / 第 641-680 行
```tablegen
 641: def : InstRW<[WLat5LSU, DFU2, LSU2, GroupAlone], (instregex "TP$")>;
 642: def : InstRW<[WLat30, MCD], (instregex "ED(MK)?$")>;
 643: 
 644: //===----------------------------------------------------------------------===//
 645: // Access registers
 646: //===----------------------------------------------------------------------===//
 647: 
 648: // Extract/set/copy access register
 649: def : InstRW<[WLat3, LSU, NormalGr], (instregex "(EAR|SAR|CPYA)$")>;
 650: 
 651: // Load address extended
 652: def : InstRW<[WLat5, LSU, FXU, GroupAlone], (instregex "LAE(Y)?$")>;
 653: 
 654: // Load/store access multiple (not modeled precisely)
 655: def : InstRW<[WLat10, WLat10, LSU5, GroupAlone], (instregex "LAM(Y)?$")>;
 656: def : InstRW<[WLat1, FXU5, LSU5, GroupAlone], (instregex "STAM(Y)?$")>;
 657: 
 658: //===----------------------------------------------------------------------===//
 659: // Program mask and addressing mode
 660: //===----------------------------------------------------------------------===//
 661: 
 662: // Insert Program Mask
 663: def : InstRW<[WLat3, FXU, EndGroup], (instregex "IPM$")>;
 664: 
 665: // Set Program Mask
 666: def : InstRW<[WLat3, LSU, EndGroup], (instregex "SPM$")>;
 667: 
 668: // Branch and link
 669: def : InstRW<[WLat1, FXU2, LSU, GroupAlone], (instregex "BAL(R)?$")>;
 670: 
 671: // Test addressing mode
 672: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TAM$")>;
 673: 
 674: // Set addressing mode
 675: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SAM(24|31|64)$")>;
 676: 
 677: // Branch (and save) and set mode.
 678: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "BSM$")>;
 679: def : InstRW<[WLat1, FXU2, LSU, GroupAlone], (instregex "BASSM$")>;
 680: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 681-720 / 第 681-720 行
```tablegen
 681: //===----------------------------------------------------------------------===//
 682: // Transactional execution
 683: //===----------------------------------------------------------------------===//
 684: 
 685: // Transaction begin
 686: def : InstRW<[WLat9, LSU2, FXU5, GroupAlone], (instregex "TBEGIN(C)?$")>;
 687: 
 688: // Transaction end
 689: def : InstRW<[WLat4, LSU, GroupAlone], (instregex "TEND$")>;
 690: 
 691: // Transaction abort
 692: def : InstRW<[WLat30, MCD], (instregex "TABORT$")>;
 693: 
 694: // Extract Transaction Nesting Depth
 695: def : InstRW<[WLat30, MCD], (instregex "ETND$")>;
 696: 
 697: // Nontransactional store
 698: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "NTSTG$")>;
 699: 
 700: //===----------------------------------------------------------------------===//
 701: // Processor assist
 702: //===----------------------------------------------------------------------===//
 703: 
 704: def : InstRW<[WLat30, MCD], (instregex "PPA$")>;
 705: 
 706: //===----------------------------------------------------------------------===//
 707: // Miscellaneous Instructions.
 708: //===----------------------------------------------------------------------===//
 709: 
 710: // Find leftmost one
 711: def : InstRW<[WLat7, WLat7, FXU2, GroupAlone], (instregex "FLOGR$")>;
 712: 
 713: // Population count
 714: def : InstRW<[WLat3, WLat3, FXU, NormalGr], (instregex "POPCNT$")>;
 715: 
 716: // String instructions
 717: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "SRST(U)?$")>;
 718: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CUSE$")>;
 719: 
 720: // Various complex instructions
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CFC$")>;
 722: def : InstRW<[WLat30, WLat30, WLat30, WLat30, WLat30, WLat30, MCD],
 723:              (instregex "UPT$")>;
 724: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CKSM$")>;
 725: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CMPSC$")>;
 726: 
 727: // Execute
 728: def : InstRW<[LSU, GroupAlone], (instregex "EX(RL)?$")>;
 729: 
 730: //===----------------------------------------------------------------------===//
 731: // .insn directive instructions
 732: //===----------------------------------------------------------------------===//
 733: 
 734: // An "empty" sched-class will be assigned instead of the "invalid sched-class".
 735: // getNumDecoderSlots() will then return 1 instead of 0.
 736: def : InstRW<[], (instregex "Insn.*")>;
 737: 
 738: 
 739: // ----------------------------- Floating point ----------------------------- //
 740: 
 741: //===----------------------------------------------------------------------===//
 742: // FP: Move instructions
 743: //===----------------------------------------------------------------------===//
 744: 
 745: // Load zero
 746: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LZ(DR|ER|ER_16)$")>;
 747: def : InstRW<[WLat2, FXU2, GroupAlone], (instregex "LZXR$")>;
 748: 
 749: // Load
 750: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LER(16)?$")>;
 751: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LD(R|R16|R32|GR)$")>;
 752: def : InstRW<[WLat3, FXU, NormalGr], (instregex "LGDR$")>;
 753: def : InstRW<[WLat2, FXU2, GroupAlone], (instregex "LXR$")>;
 754: 
 755: // Load and Test
 756: def : InstRW<[WLat9, WLat9, FPU, NormalGr], (instregex "LT(E|D)BR$")>;
 757: def : InstRW<[WLat10, WLat10, FPU4, GroupAlone], (instregex "LTXBR$")>;
 758: 
 759: // Copy sign
 760: def : InstRW<[WLat5, FXU2, GroupAlone], (instregex "CPSDR(d|s|h)(d|s|h)$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 761-800 / 第 761-800 行
```tablegen
 761: 
 762: //===----------------------------------------------------------------------===//
 763: // FP: Load instructions
 764: //===----------------------------------------------------------------------===//
 765: 
 766: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L(E16|E|D)(Y|E32)?$")>;
 767: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LX$")>;
 768: 
 769: //===----------------------------------------------------------------------===//
 770: // FP: Store instructions
 771: //===----------------------------------------------------------------------===//
 772: 
 773: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "ST(E16|E|D)(Y)?$")>;
 774: def : InstRW<[WLat1, FXU, LSU, NormalGr], (instregex "STX$")>;
 775: 
 776: //===----------------------------------------------------------------------===//
 777: // FP: Conversion instructions
 778: //===----------------------------------------------------------------------===//
 779: 
 780: // Load rounded
 781: def : InstRW<[WLat7, FPU, NormalGr], (instregex "LEDBR(A)?$")>;
 782: def : InstRW<[WLat9, FPU2, NormalGr], (instregex "L(E|D)XBR(A)?$")>;
 783: 
 784: // Load lengthened
 785: def : InstRW<[WLat7LSU, FPU, LSU, NormalGr], (instregex "LDEB$")>;
 786: def : InstRW<[WLat7, FPU, NormalGr], (instregex "LDEBR$")>;
 787: def : InstRW<[WLat11LSU, FPU4, LSU, GroupAlone], (instregex "LX(E|D)B$")>;
 788: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "LX(E|D)BR$")>;
 789: 
 790: // Convert from fixed / logical
 791: def : InstRW<[WLat8, FXU, FPU, GroupAlone], (instregex "C(E|D)(F|G)BR(A)?$")>;
 792: def : InstRW<[WLat11, FXU, FPU4, GroupAlone2], (instregex "CX(F|G)BR(A?)$")>;
 793: def : InstRW<[WLat8, FXU, FPU, GroupAlone], (instregex "CEL(F|G)BR$")>;
 794: def : InstRW<[WLat8, FXU, FPU, GroupAlone], (instregex "CDL(F|G)BR$")>;
 795: def : InstRW<[WLat11, FXU, FPU4, GroupAlone2], (instregex "CXL(F|G)BR$")>;
 796: 
 797: // Convert to fixed / logical
 798: def : InstRW<[WLat12, WLat12, FXU, FPU, GroupAlone],
 799:              (instregex "C(F|G)(E|D)BR(A?)$")>;
 800: def : InstRW<[WLat12, WLat12, FXU, FPU2, GroupAlone],
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 801-840 / 第 801-840 行
```tablegen
 801:              (instregex "C(F|G)XBR(A?)$")>;
 802: def : InstRW<[WLat12, WLat12, FXU, FPU, GroupAlone],
 803:              (instregex "CL(F|G)(E|D)BR$")>;
 804: def : InstRW<[WLat12, WLat12, FXU, FPU2, GroupAlone], (instregex "CL(F|G)XBR$")>;
 805: 
 806: //===----------------------------------------------------------------------===//
 807: // FP: Unary arithmetic
 808: //===----------------------------------------------------------------------===//
 809: 
 810: // Load Complement / Negative / Positive
 811: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "L(C|N|P)(E|D)BR$")>;
 812: def : InstRW<[WLat1, FXU, NormalGr], (instregex "L(C|N|P)DFR(_32|_16)?$")>;
 813: def : InstRW<[WLat10, WLat10, FPU4, GroupAlone], (instregex "L(C|N|P)XBR$")>;
 814: 
 815: // Square root
 816: def : InstRW<[WLat30, FPU, LSU, NormalGr], (instregex "SQ(E|D)B$")>;
 817: def : InstRW<[WLat30, FPU, NormalGr], (instregex "SQ(E|D)BR$")>;
 818: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "SQXBR$")>;
 819: 
 820: // Load FP integer
 821: def : InstRW<[WLat7, FPU, NormalGr], (instregex "FI(E|D)BR(A)?$")>;
 822: def : InstRW<[WLat15, FPU4, GroupAlone], (instregex "FIXBR(A)?$")>;
 823: 
 824: //===----------------------------------------------------------------------===//
 825: // FP: Binary arithmetic
 826: //===----------------------------------------------------------------------===//
 827: 
 828: // Addition
 829: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 830:              (instregex "A(E|D)B$")>;
 831: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "A(E|D)BR$")>;
 832: def : InstRW<[WLat20, WLat20, FPU4, GroupAlone], (instregex "AXBR$")>;
 833: 
 834: // Subtraction
 835: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 836:              (instregex "S(E|D)B$")>;
 837: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "S(E|D)BR$")>;
 838: def : InstRW<[WLat20, WLat20, FPU4, GroupAlone], (instregex "SXBR$")>;
 839: 
 840: // Multiply
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: def : InstRW<[WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 842:              (instregex "M(D|DE|EE)B$")>;
 843: def : InstRW<[WLat7, FPU, NormalGr], (instregex "M(D|DE|EE)BR$")>;
 844: def : InstRW<[WLat11LSU, RegReadAdv, FPU4, LSU, GroupAlone],
 845:              (instregex "MXDB$")>;
 846: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "MXDBR$")>;
 847: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "MXBR$")>;
 848: 
 849: // Multiply and add / subtract
 850: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, FPU2, LSU, GroupAlone],
 851:              (instregex "M(A|S)EB$")>;
 852: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "M(A|S)EBR$")>;
 853: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, FPU2, LSU, GroupAlone],
 854:              (instregex "M(A|S)DB$")>;
 855: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "M(A|S)DBR$")>;
 856: 
 857: // Division
 858: def : InstRW<[WLat30, RegReadAdv, FPU, LSU, NormalGr], (instregex "D(E|D)B$")>;
 859: def : InstRW<[WLat30, FPU, NormalGr], (instregex "D(E|D)BR$")>;
 860: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "DXBR$")>;
 861: 
 862: // Divide to integer
 863: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "DI(E|D)BR$")>;
 864: 
 865: //===----------------------------------------------------------------------===//
 866: // FP: Comparisons
 867: //===----------------------------------------------------------------------===//
 868: 
 869: // Compare
 870: def : InstRW<[WLat11LSU, RegReadAdv, FPU, LSU, NormalGr],
 871:              (instregex "(K|C)(E|D)B$")>;
 872: def : InstRW<[WLat9, FPU, NormalGr], (instregex "(K|C)(E|D)BR$")>;
 873: def : InstRW<[WLat30, FPU2, NormalGr], (instregex "(K|C)XBR$")>;
 874: 
 875: // Test Data Class
 876: def : InstRW<[WLat15, FPU, LSU, NormalGr], (instregex "TC(E|D)B$")>;
 877: def : InstRW<[WLat15, FPU4, LSU, GroupAlone], (instregex "TCXB$")>;
 878: 
 879: //===----------------------------------------------------------------------===//
 880: // FP: Floating-point control register instructions
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 881-920 / 第 881-920 行
```tablegen
 881: //===----------------------------------------------------------------------===//
 882: 
 883: def : InstRW<[WLat4, FXU, LSU, GroupAlone], (instregex "EFPC$")>;
 884: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "STFPC$")>;
 885: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "SFPC$")>;
 886: def : InstRW<[WLat1, LSU2, GroupAlone], (instregex "LFPC$")>;
 887: def : InstRW<[WLat30, MCD], (instregex "SFASR$")>;
 888: def : InstRW<[WLat30, MCD], (instregex "LFAS$")>;
 889: def : InstRW<[WLat2, FXU, GroupAlone], (instregex "SRNM(B|T)?$")>;
 890: 
 891: 
 892: // --------------------- Hexadecimal floating point ------------------------- //
 893: 
 894: //===----------------------------------------------------------------------===//
 895: // HFP: Move instructions
 896: //===----------------------------------------------------------------------===//
 897: 
 898: // Load and Test
 899: def : InstRW<[WLat9, WLat9, FPU, NormalGr], (instregex "LT(E|D)R$")>;
 900: def : InstRW<[WLat9, WLat9, FPU4, GroupAlone], (instregex "LTXR$")>;
 901: 
 902: //===----------------------------------------------------------------------===//
 903: // HFP: Conversion instructions
 904: //===----------------------------------------------------------------------===//
 905: 
 906: // Load rounded
 907: def : InstRW<[WLat7, FPU, NormalGr], (instregex "(LEDR|LRER)$")>;
 908: def : InstRW<[WLat7, FPU, NormalGr], (instregex "LEXR$")>;
 909: def : InstRW<[WLat9, FPU, NormalGr], (instregex "(LDXR|LRDR)$")>;
 910: 
 911: // Load lengthened
 912: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LDE$")>;
 913: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LDER$")>;
 914: def : InstRW<[WLat11LSU, FPU4, LSU, GroupAlone], (instregex "LX(E|D)$")>;
 915: def : InstRW<[WLat9, FPU4, GroupAlone], (instregex "LX(E|D)R$")>;
 916: 
 917: // Convert from fixed
 918: def : InstRW<[WLat8, FXU, FPU, GroupAlone], (instregex "C(E|D)(F|G)R$")>;
 919: def : InstRW<[WLat10, FXU, FPU4, GroupAlone2], (instregex "CX(F|G)R$")>;
 920: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: // Convert to fixed
 922: def : InstRW<[WLat12, WLat12, FXU, FPU, GroupAlone],
 923:              (instregex "C(F|G)(E|D)R$")>;
 924: def : InstRW<[WLat30, WLat30, FXU, FPU2, GroupAlone], (instregex "C(F|G)XR$")>;
 925: 
 926: // Convert BFP to HFP / HFP to BFP.
 927: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "THD(E)?R$")>;
 928: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "TB(E)?DR$")>;
 929: 
 930: //===----------------------------------------------------------------------===//
 931: // HFP: Unary arithmetic
 932: //===----------------------------------------------------------------------===//
 933: 
 934: // Load Complement / Negative / Positive
 935: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "L(C|N|P)(E|D)R$")>;
 936: def : InstRW<[WLat9, WLat9, FPU4, GroupAlone], (instregex "L(C|N|P)XR$")>;
 937: 
 938: // Halve
 939: def : InstRW<[WLat7, FPU, NormalGr], (instregex "H(E|D)R$")>;
 940: 
 941: // Square root
 942: def : InstRW<[WLat30, FPU, LSU, NormalGr], (instregex "SQ(E|D)$")>;
 943: def : InstRW<[WLat30, FPU, NormalGr], (instregex "SQ(E|D)R$")>;
 944: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "SQXR$")>;
 945: 
 946: // Load FP integer
 947: def : InstRW<[WLat7, FPU, NormalGr], (instregex "FI(E|D)R$")>;
 948: def : InstRW<[WLat15, FPU4, GroupAlone], (instregex "FIXR$")>;
 949: 
 950: //===----------------------------------------------------------------------===//
 951: // HFP: Binary arithmetic
 952: //===----------------------------------------------------------------------===//
 953: 
 954: // Addition
 955: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 956:              (instregex "A(E|D|U|W)$")>;
 957: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "A(E|D|U|W)R$")>;
 958: def : InstRW<[WLat15, WLat15, FPU4, GroupAlone], (instregex "AXR$")>;
 959: 
 960: // Subtraction
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, FPU, LSU, NormalGr],
 962:              (instregex "S(E|D|U|W)$")>;
 963: def : InstRW<[WLat7, WLat7, FPU, NormalGr], (instregex "S(E|D|U|W)R$")>;
 964: def : InstRW<[WLat15, WLat15, FPU4, GroupAlone], (instregex "SXR$")>;
 965: 
 966: // Multiply
 967: def : InstRW<[WLat7LSU, RegReadAdv, FPU, LSU, NormalGr], (instregex "M(D|EE)$")>;
 968: def : InstRW<[WLat8LSU, RegReadAdv, FPU, LSU, NormalGr], (instregex "M(DE|E)$")>;
 969: def : InstRW<[WLat7, FPU, NormalGr], (instregex "M(D|EE)R$")>;
 970: def : InstRW<[WLat8, FPU, NormalGr], (instregex "M(DE|E)R$")>;
 971: def : InstRW<[WLat11LSU, RegReadAdv, FPU4, LSU, GroupAlone], (instregex "MXD$")>;
 972: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "MXDR$")>;
 973: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "MXR$")>;
 974: def : InstRW<[WLat11LSU, RegReadAdv, FPU4, LSU, GroupAlone], (instregex "MY$")>;
 975: def : InstRW<[WLat7LSU, RegReadAdv, FPU2, LSU, GroupAlone],
 976:              (instregex "MY(H|L)$")>;
 977: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "MYR$")>;
 978: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "MY(H|L)R$")>;
 979: 
 980: // Multiply and add / subtract
 981: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, FPU2, LSU, GroupAlone],
 982:              (instregex "M(A|S)(E|D)$")>;
 983: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "M(A|S)(E|D)R$")>;
 984: def : InstRW<[WLat11LSU, RegReadAdv, RegReadAdv, FPU4, LSU, GroupAlone],
 985:              (instregex "MAY$")>;
 986: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, FPU2, LSU, GroupAlone],
 987:              (instregex "MAY(H|L)$")>;
 988: def : InstRW<[WLat10, FPU4, GroupAlone], (instregex "MAYR$")>;
 989: def : InstRW<[WLat7, FPU, GroupAlone], (instregex "MAY(H|L)R$")>;
 990: 
 991: // Division
 992: def : InstRW<[WLat30, RegReadAdv, FPU, LSU, NormalGr], (instregex "D(E|D)$")>;
 993: def : InstRW<[WLat30, FPU, NormalGr], (instregex "D(E|D)R$")>;
 994: def : InstRW<[WLat30, FPU4, GroupAlone], (instregex "DXR$")>;
 995: 
 996: //===----------------------------------------------------------------------===//
 997: // HFP: Comparisons
 998: //===----------------------------------------------------------------------===//
 999: 
1000: // Compare
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: def : InstRW<[WLat11LSU, RegReadAdv, FPU, LSU, NormalGr], (instregex "C(E|D)$")>;
1002: def : InstRW<[WLat9, FPU, NormalGr], (instregex "C(E|D)R$")>;
1003: def : InstRW<[WLat15, FPU2, NormalGr], (instregex "CXR$")>;
1004: 
1005: 
1006: // ------------------------ Decimal floating point -------------------------- //
1007: 
1008: //===----------------------------------------------------------------------===//
1009: // DFP: Move instructions
1010: //===----------------------------------------------------------------------===//
1011: 
1012: // Load and Test
1013: def : InstRW<[WLat4, WLat4, DFU, NormalGr], (instregex "LTDTR$")>;
1014: def : InstRW<[WLat6, WLat6, DFU4, GroupAlone], (instregex "LTXTR$")>;
1015: 
1016: //===----------------------------------------------------------------------===//
1017: // DFP: Conversion instructions
1018: //===----------------------------------------------------------------------===//
1019: 
1020: // Load rounded
1021: def : InstRW<[WLat30, DFU, NormalGr], (instregex "LEDTR$")>;
1022: def : InstRW<[WLat30, DFU2, NormalGr], (instregex "LDXTR$")>;
1023: 
1024: // Load lengthened
1025: def : InstRW<[WLat7, DFU, NormalGr], (instregex "LDETR$")>;
1026: def : InstRW<[WLat6, DFU4, GroupAlone], (instregex "LXDTR$")>;
1027: 
1028: // Convert from fixed / logical
1029: def : InstRW<[WLat9, FXU, DFU, GroupAlone], (instregex "CDFTR$")>;
1030: def : InstRW<[WLat30, FXU, DFU, GroupAlone], (instregex "CDGTR(A)?$")>;
1031: def : InstRW<[WLat5, FXU, DFU4, GroupAlone2], (instregex "CXFTR(A)?$")>;
1032: def : InstRW<[WLat30, FXU, DFU4, GroupAlone2], (instregex "CXGTR(A)?$")>;
1033: def : InstRW<[WLat9, FXU, DFU, GroupAlone], (instregex "CDL(F|G)TR$")>;
1034: def : InstRW<[WLat9, FXU, DFU4, GroupAlone2], (instregex "CXLFTR$")>;
1035: def : InstRW<[WLat5, FXU, DFU4, GroupAlone2], (instregex "CXLGTR$")>;
1036: 
1037: // Convert to fixed / logical
1038: def : InstRW<[WLat11, WLat11, FXU, DFU, GroupAlone], (instregex "CFDTR(A)?$")>;
1039: def : InstRW<[WLat30, WLat30, FXU, DFU, GroupAlone], (instregex "CGDTR(A)?$")>;
1040: def : InstRW<[WLat7, WLat7, FXU, DFU2, GroupAlone], (instregex "CFXTR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: def : InstRW<[WLat30, WLat30, FXU, DFU2, GroupAlone], (instregex "CGXTR(A)?$")>;
1042: def : InstRW<[WLat11, WLat11, FXU, DFU, GroupAlone], (instregex "CL(F|G)DTR$")>;
1043: def : InstRW<[WLat7, WLat7, FXU, DFU2, GroupAlone], (instregex "CL(F|G)XTR$")>;
1044: 
1045: // Convert from / to signed / unsigned packed
1046: def : InstRW<[WLat5, FXU, DFU, GroupAlone], (instregex "CD(S|U)TR$")>;
1047: def : InstRW<[WLat8, FXU2, DFU4, GroupAlone2], (instregex "CX(S|U)TR$")>;
1048: def : InstRW<[WLat7, FXU, DFU, GroupAlone], (instregex "C(S|U)DTR$")>;
1049: def : InstRW<[WLat12, FXU2, DFU4, GroupAlone2], (instregex "C(S|U)XTR$")>;
1050: 
1051: // Convert from / to zoned
1052: def : InstRW<[WLat4LSU, LSU, DFU2, GroupAlone], (instregex "CDZT$")>;
1053: def : InstRW<[WLat11LSU, LSU2, DFU4, GroupAlone3], (instregex "CXZT$")>;
1054: def : InstRW<[WLat1, FXU, LSU, DFU2, GroupAlone], (instregex "CZDT$")>;
1055: def : InstRW<[WLat1, FXU, LSU, DFU2, GroupAlone], (instregex "CZXT$")>;
1056: 
1057: // Perform floating-point operation
1058: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "PFPO$")>;
1059: 
1060: //===----------------------------------------------------------------------===//
1061: // DFP: Unary arithmetic
1062: //===----------------------------------------------------------------------===//
1063: 
1064: // Load FP integer
1065: def : InstRW<[WLat8, DFU, NormalGr], (instregex "FIDTR$")>;
1066: def : InstRW<[WLat10, DFU4, GroupAlone], (instregex "FIXTR$")>;
1067: 
1068: // Extract biased exponent
1069: def : InstRW<[WLat7, FXU, DFU, GroupAlone], (instregex "EEDTR$")>;
1070: def : InstRW<[WLat8, FXU, DFU2, GroupAlone], (instregex "EEXTR$")>;
1071: 
1072: // Extract significance
1073: def : InstRW<[WLat7, FXU, DFU, GroupAlone], (instregex "ESDTR$")>;
1074: def : InstRW<[WLat8, FXU, DFU2, GroupAlone], (instregex "ESXTR$")>;
1075: 
1076: //===----------------------------------------------------------------------===//
1077: // DFP: Binary arithmetic
1078: //===----------------------------------------------------------------------===//
1079: 
1080: // Addition
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: def : InstRW<[WLat9, WLat9, DFU, NormalGr], (instregex "ADTR(A)?$")>;
1082: def : InstRW<[WLat30, WLat30, DFU4, GroupAlone], (instregex "AXTR(A)?$")>;
1083: 
1084: // Subtraction
1085: def : InstRW<[WLat9, WLat9, DFU, NormalGr], (instregex "SDTR(A)?$")>;
1086: def : InstRW<[WLat30, WLat30, DFU4, GroupAlone], (instregex "SXTR(A)?$")>;
1087: 
1088: // Multiply
1089: def : InstRW<[WLat30, DFU, NormalGr], (instregex "MDTR(A)?$")>;
1090: def : InstRW<[WLat30, DFU4, GroupAlone], (instregex "MXTR(A)?$")>;
1091: 
1092: // Division
1093: def : InstRW<[WLat30, DFU, NormalGr], (instregex "DDTR(A)?$")>;
1094: def : InstRW<[WLat30, DFU4, GroupAlone], (instregex "DXTR(A)?$")>;
1095: 
1096: // Quantize
1097: def : InstRW<[WLat8, WLat8, DFU, NormalGr], (instregex "QADTR$")>;
1098: def : InstRW<[WLat10, WLat10, DFU4, GroupAlone], (instregex "QAXTR$")>;
1099: 
1100: // Reround
1101: def : InstRW<[WLat11, WLat11, FXU, DFU, GroupAlone], (instregex "RRDTR$")>;
1102: def : InstRW<[WLat30, WLat30, FXU, DFU4, GroupAlone2], (instregex "RRXTR$")>;
1103: 
1104: // Shift significand left/right
1105: def : InstRW<[WLat7LSU, LSU, DFU, GroupAlone], (instregex "S(L|R)DT$")>;
1106: def : InstRW<[WLat11LSU, LSU, DFU4, GroupAlone], (instregex "S(L|R)XT$")>;
1107: 
1108: // Insert biased exponent
1109: def : InstRW<[WLat5, FXU, DFU, GroupAlone], (instregex "IEDTR$")>;
1110: def : InstRW<[WLat7, FXU, DFU4, GroupAlone2], (instregex "IEXTR$")>;
1111: 
1112: //===----------------------------------------------------------------------===//
1113: // DFP: Comparisons
1114: //===----------------------------------------------------------------------===//
1115: 
1116: // Compare
1117: def : InstRW<[WLat9, DFU, NormalGr], (instregex "(K|C)DTR$")>;
1118: def : InstRW<[WLat10, DFU2, NormalGr], (instregex "(K|C)XTR$")>;
1119: 
1120: // Compare biased exponent
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: def : InstRW<[WLat4, DFU, NormalGr], (instregex "CEDTR$")>;
1122: def : InstRW<[WLat5, DFU2, NormalGr], (instregex "CEXTR$")>;
1123: 
1124: // Test Data Class/Group
1125: def : InstRW<[WLat9, LSU, DFU, NormalGr], (instregex "TD(C|G)DT$")>;
1126: def : InstRW<[WLat10, LSU, DFU, NormalGr], (instregex "TD(C|G)ET$")>;
1127: def : InstRW<[WLat10, LSU, DFU2, NormalGr], (instregex "TD(C|G)XT$")>;
1128: 
1129: 
1130: // -------------------------------- System ---------------------------------- //
1131: 
1132: //===----------------------------------------------------------------------===//
1133: // System: Program-Status Word Instructions
1134: //===----------------------------------------------------------------------===//
1135: 
1136: def : InstRW<[WLat30, WLat30, MCD], (instregex "EPSW$")>;
1137: def : InstRW<[WLat30, MCD], (instregex "LPSW(E)?$")>;
1138: def : InstRW<[WLat3, FXU, GroupAlone], (instregex "IPK$")>;
1139: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SPKA$")>;
1140: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SSM$")>;
1141: def : InstRW<[WLat1, FXU, LSU, GroupAlone], (instregex "ST(N|O)SM$")>;
1142: def : InstRW<[WLat3, FXU, NormalGr], (instregex "IAC$")>;
1143: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SAC(F)?$")>;
1144: 
1145: //===----------------------------------------------------------------------===//
1146: // System: Control Register Instructions
1147: //===----------------------------------------------------------------------===//
1148: 
1149: def : InstRW<[WLat10, WLat10, FXU, LSU, NormalGr], (instregex "LCTL(G)?$")>;
1150: def : InstRW<[WLat1, FXU5, LSU5, GroupAlone], (instregex "STCT(L|G)$")>;
1151: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "E(P|S)A(I)?R$")>;
1152: def : InstRW<[WLat30, MCD], (instregex "SSA(I)?R$")>;
1153: def : InstRW<[WLat30, MCD], (instregex "ESEA$")>;
1154: 
1155: //===----------------------------------------------------------------------===//
1156: // System: Prefix-Register Instructions
1157: //===----------------------------------------------------------------------===//
1158: 
1159: def : InstRW<[WLat30, MCD], (instregex "S(T)?PX$")>;
1160: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161: //===----------------------------------------------------------------------===//
1162: // System: Storage-Key and Real Memory Instructions
1163: //===----------------------------------------------------------------------===//
1164: 
1165: def : InstRW<[WLat30, MCD], (instregex "ISKE$")>;
1166: def : InstRW<[WLat30, MCD], (instregex "IVSK$")>;
1167: def : InstRW<[WLat30, MCD], (instregex "SSKE(Opt)?$")>;
1168: def : InstRW<[WLat30, MCD], (instregex "RRB(E|M)$")>;
1169: def : InstRW<[WLat30, MCD], (instregex "PFMF$")>;
1170: def : InstRW<[WLat30, WLat30, MCD], (instregex "TB$")>;
1171: def : InstRW<[WLat30, MCD], (instregex "PGIN$")>;
1172: def : InstRW<[WLat30, MCD], (instregex "PGOUT$")>;
1173: 
1174: //===----------------------------------------------------------------------===//
1175: // System: Dynamic-Address-Translation Instructions
1176: //===----------------------------------------------------------------------===//
1177: 
1178: def : InstRW<[WLat30, MCD], (instregex "IPTE(Opt)?(Opt)?$")>;
1179: def : InstRW<[WLat30, MCD], (instregex "IDTE(Opt)?$")>;
1180: def : InstRW<[WLat30, MCD], (instregex "CRDTE(Opt)?$")>;
1181: def : InstRW<[WLat30, MCD], (instregex "PTLB$")>;
1182: def : InstRW<[WLat30, WLat30, MCD], (instregex "CSP(G)?$")>;
1183: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "LPTEA$")>;
1184: def : InstRW<[WLat30, WLat30, MCD], (instregex "LRA(Y|G)?$")>;
1185: def : InstRW<[WLat30, MCD], (instregex "STRAG$")>;
1186: def : InstRW<[WLat30, MCD], (instregex "LURA(G)?$")>;
1187: def : InstRW<[WLat30, MCD], (instregex "STUR(A|G)$")>;
1188: def : InstRW<[WLat30, MCD], (instregex "TPROT$")>;
1189: 
1190: //===----------------------------------------------------------------------===//
1191: // System: Memory-move Instructions
1192: //===----------------------------------------------------------------------===//
1193: 
1194: def : InstRW<[WLat30, MCD], (instregex "MVC(K|P|S)$")>;
1195: def : InstRW<[WLat30, MCD], (instregex "MVC(S|D)K$")>;
1196: def : InstRW<[WLat30, MCD], (instregex "MVCOS$")>;
1197: def : InstRW<[WLat30, MCD], (instregex "MVPG$")>;
1198: 
1199: //===----------------------------------------------------------------------===//
1200: // System: Address-Space Instructions
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201: //===----------------------------------------------------------------------===//
1202: 
1203: def : InstRW<[WLat30, MCD], (instregex "LASP$")>;
1204: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "PALB$")>;
1205: def : InstRW<[WLat30, MCD], (instregex "PC$")>;
1206: def : InstRW<[WLat30, MCD], (instregex "PR$")>;
1207: def : InstRW<[WLat30, MCD], (instregex "PT(I)?$")>;
1208: def : InstRW<[WLat30, MCD], (instregex "RP$")>;
1209: def : InstRW<[WLat30, MCD], (instregex "BS(G|A)$")>;
1210: def : InstRW<[WLat30, MCD], (instregex "TAR$")>;
1211: 
1212: //===----------------------------------------------------------------------===//
1213: // System: Linkage-Stack Instructions
1214: //===----------------------------------------------------------------------===//
1215: 
1216: def : InstRW<[WLat30, MCD], (instregex "BAKR$")>;
1217: def : InstRW<[WLat30, MCD], (instregex "EREG(G)?$")>;
1218: def : InstRW<[WLat30, WLat30, MCD], (instregex "(E|M)STA$")>;
1219: 
1220: //===----------------------------------------------------------------------===//
1221: // System: Time-Related Instructions
1222: //===----------------------------------------------------------------------===//
1223: 
1224: def : InstRW<[WLat30, MCD], (instregex "PTFF$")>;
1225: def : InstRW<[WLat30, MCD], (instregex "SCK$")>;
1226: def : InstRW<[WLat30, MCD], (instregex "SCKPF$")>;
1227: def : InstRW<[WLat30, MCD], (instregex "SCKC$")>;
1228: def : InstRW<[WLat30, MCD], (instregex "SPT$")>;
1229: def : InstRW<[WLat9, FXU, LSU2, GroupAlone], (instregex "STCK(F)?$")>;
1230: def : InstRW<[WLat20, LSU4, FXU2, GroupAlone2], (instregex "STCKE$")>;
1231: def : InstRW<[WLat30, MCD], (instregex "STCKC$")>;
1232: def : InstRW<[WLat30, MCD], (instregex "STPT$")>;
1233: 
1234: //===----------------------------------------------------------------------===//
1235: // System: CPU-Related Instructions
1236: //===----------------------------------------------------------------------===//
1237: 
1238: def : InstRW<[WLat30, MCD], (instregex "STAP$")>;
1239: def : InstRW<[WLat30, MCD], (instregex "STIDP$")>;
1240: def : InstRW<[WLat30, WLat30, MCD], (instregex "STSI$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241: def : InstRW<[WLat30, WLat30, MCD], (instregex "STFL(E)?$")>;
1242: def : InstRW<[WLat30, MCD], (instregex "ECAG$")>;
1243: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECTG$")>;
1244: def : InstRW<[WLat30, MCD], (instregex "PTF$")>;
1245: def : InstRW<[WLat30, MCD], (instregex "PCKMO$")>;
1246: 
1247: //===----------------------------------------------------------------------===//
1248: // System: Miscellaneous Instructions
1249: //===----------------------------------------------------------------------===//
1250: 
1251: def : InstRW<[WLat30, MCD], (instregex "SVC$")>;
1252: def : InstRW<[WLat1, FXU, GroupAlone], (instregex "MC$")>;
1253: def : InstRW<[WLat30, MCD], (instregex "DIAG$")>;
1254: def : InstRW<[WLat1, FXU, NormalGr], (instregex "TRAC(E|G)$")>;
1255: def : InstRW<[WLat30, MCD], (instregex "TRAP(2|4)$")>;
1256: def : InstRW<[WLat30, MCD], (instregex "SIG(P|A)$")>;
1257: def : InstRW<[WLat30, MCD], (instregex "SIE$")>;
1258: 
1259: //===----------------------------------------------------------------------===//
1260: // System: CPU-Measurement Facility Instructions
1261: //===----------------------------------------------------------------------===//
1262: 
1263: def : InstRW<[WLat1, FXU, NormalGr], (instregex "LPP$")>;
1264: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECPGA$")>;
1265: def : InstRW<[WLat30, WLat30, MCD], (instregex "E(C|P)CTR$")>;
1266: def : InstRW<[WLat30, MCD], (instregex "LCCTL$")>;
1267: def : InstRW<[WLat30, MCD], (instregex "L(P|S)CTL$")>;
1268: def : InstRW<[WLat30, MCD], (instregex "Q(S|CTR)I$")>;
1269: def : InstRW<[WLat30, MCD], (instregex "S(C|P)CTR$")>;
1270: 
1271: //===----------------------------------------------------------------------===//
1272: // System: I/O Instructions
1273: //===----------------------------------------------------------------------===//
1274: 
1275: def : InstRW<[WLat30, MCD], (instregex "(C|H|R|X)SCH$")>;
1276: def : InstRW<[WLat30, MCD], (instregex "(M|S|ST|T)SCH$")>;
1277: def : InstRW<[WLat30, MCD], (instregex "RCHP$")>;
1278: def : InstRW<[WLat30, MCD], (instregex "SCHM$")>;
1279: def : InstRW<[WLat30, MCD], (instregex "STC(PS|RW)$")>;
1280: def : InstRW<[WLat30, MCD], (instregex "TPI$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1281-1290 / 第 1281-1290 行
```tablegen
1281: def : InstRW<[WLat30, MCD], (instregex "SAL$")>;
1282: 
1283: //===----------------------------------------------------------------------===//
1284: // NOPs
1285: //===----------------------------------------------------------------------===//
1286: 
1287: def : InstRW<[WLat1, LSU, NormalGr], (instregex "NOP(R)?(Opt)?$")>;
1288: def : InstRW<[WLat1, VBU, NormalGr], (instregex "J(G)?NOP$")>;
1289: }
1290: 
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
