# SystemZScheduleZ14.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZScheduleZ14.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //-- SystemZScheduleZ14.td - SystemZ Scheduling Definitions ----*- tblgen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the machine model for Z14 to support instruction
  10: // scheduling and other instruction cost heuristics.
  11: //
  12: // Pseudos expanded right after isel do not need to be modelled here.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: def Z14Model : SchedMachineModel {
  17: 
  18:     let UnsupportedFeatures = Arch12UnsupportedFeatures.List;
  19: 
  20:     let IssueWidth = 6;             // Number of instructions decoded per cycle.
  21:     let MicroOpBufferSize = 60;     // Issue queues
  22:     let LoadLatency = 1;            // Optimistic load latency.
  23: 
  24:     let PostRAScheduler = 1;
  25: 
  26:     // Extra cycles for a mispredicted branch.
  27:     let MispredictPenalty = 20;
  28: }
  29: 
  30: let SchedModel = Z14Model in  {
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
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `Z14Model`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `Z14Model` 等 TableGen 记录。

### Lines 41-80 / 第 41-80 行
```tablegen
  41: def : WriteRes<Cracked, []> {
  42:   let NumMicroOps = 2;
  43:   let BeginGroup  = 1;
  44: }
  45: def : WriteRes<GroupAlone, []> {
  46:   let NumMicroOps = 3;
  47:   let BeginGroup  = 1;
  48:   let EndGroup    = 1;
  49: }
  50: def : WriteRes<GroupAlone2, []> {
  51:   let NumMicroOps = 6;
  52:   let BeginGroup  = 1;
  53:   let EndGroup    = 1;
  54: }
  55: def : WriteRes<GroupAlone3, []> {
  56:   let NumMicroOps = 9;
  57:   let BeginGroup  = 1;
  58:   let EndGroup    = 1;
  59: }
  60: 
  61: // Incoming latency removed from the register operand which is used together
  62: // with a memory operand by the instruction.
  63: def : ReadAdvance<RegReadAdv, 4>;
  64: 
  65: // LoadLatency (above) is not used for instructions in this file. This is
  66: // instead the role of LSULatency, which is the latency value added to the
  67: // result of loads and instructions with folded memory operands.
  68: def : WriteRes<LSULatency, []> { let Latency = 4; let NumMicroOps = 0; }
  69: 
  70: let NumMicroOps = 0 in {
  71:   foreach L = 1-30 in
  72:     def : WriteRes<!cast<SchedWrite>("WLat"#L), []> { let Latency = L; }
  73: }
  74: 
  75: // Execution units.
  76: def Z14_FXaUnit     : ProcResource<2>;
  77: def Z14_FXbUnit     : ProcResource<2>;
  78: def Z14_LSUnit      : ProcResource<2>;
  79: def Z14_VecUnit     : ProcResource<2>;
  80: def Z14_VecFPdUnit  : ProcResource<2> { let BufferSize = 1; /* blocking */ }
```
- **EN**: This block declares or refines TableGen records such as `Z14_FXaUnit`, `Z14_FXbUnit`, `Z14_LSUnit`, `Z14_VecUnit`, `Z14_VecFPdUnit`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `Z14_FXaUnit`, `Z14_FXbUnit`, `Z14_LSUnit`, `Z14_VecUnit`, `Z14_VecFPdUnit` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 81-120 / 第 81-120 行
```tablegen
  81: def Z14_VBUnit      : ProcResource<2>;
  82: def Z14_MCD         : ProcResource<1>;
  83: 
  84: // Subtarget specific definitions of scheduling resources.
  85: let NumMicroOps = 0 in {
  86:   def : WriteRes<FXa, [Z14_FXaUnit]>;
  87:   def : WriteRes<FXb, [Z14_FXbUnit]>;
  88:   def : WriteRes<LSU, [Z14_LSUnit]>;
  89:   def : WriteRes<VecBF,  [Z14_VecUnit]>;
  90:   def : WriteRes<VecDF,  [Z14_VecUnit]>;
  91:   def : WriteRes<VecDFX, [Z14_VecUnit]>;
  92:   def : WriteRes<VecMul,  [Z14_VecUnit]>;
  93:   def : WriteRes<VecStr,  [Z14_VecUnit]>;
  94:   def : WriteRes<VecXsPm, [Z14_VecUnit]>;
  95:   foreach Num = 2-5 in { let ReleaseAtCycles = [Num] in {
  96:     def : WriteRes<!cast<SchedWrite>("FXa"#Num), [Z14_FXaUnit]>;
  97:     def : WriteRes<!cast<SchedWrite>("FXb"#Num), [Z14_FXbUnit]>;
  98:     def : WriteRes<!cast<SchedWrite>("LSU"#Num), [Z14_LSUnit]>;
  99:     def : WriteRes<!cast<SchedWrite>("VecBF"#Num), [Z14_VecUnit]>;
 100:     def : WriteRes<!cast<SchedWrite>("VecDF"#Num), [Z14_VecUnit]>;
 101:     def : WriteRes<!cast<SchedWrite>("VecDFX"#Num), [Z14_VecUnit]>;
 102:     def : WriteRes<!cast<SchedWrite>("VecMul"#Num), [Z14_VecUnit]>;
 103:     def : WriteRes<!cast<SchedWrite>("VecStr"#Num), [Z14_VecUnit]>;
 104:     def : WriteRes<!cast<SchedWrite>("VecXsPm"#Num), [Z14_VecUnit]>;
 105:   }}
 106: 
 107:   def : WriteRes<VecFPd,  [Z14_VecFPdUnit]> { let ReleaseAtCycles = [30]; }
 108: 
 109:   def : WriteRes<VBU,     [Z14_VBUnit]>; // Virtual Branching Unit
 110: }
 111: 
 112: def : WriteRes<MCD, [Z14_MCD]> { let NumMicroOps = 3;
 113:                                  let BeginGroup  = 1;
 114:                                  let EndGroup    = 1; }
 115: 
 116: // -------------------------- INSTRUCTIONS ---------------------------------- //
 117: 
 118: // InstRW constructs have been used in order to preserve the
 119: // readability of the InstrInfo files.
 120: 
```
- **EN**: This block declares or refines TableGen records such as `Z14_VBUnit`, `Z14_MCD`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `Z14_VBUnit`, `Z14_MCD` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 121-160 / 第 121-160 行
```tablegen
 121: // For each instruction, as matched by a regexp, provide a list of
 122: // resources that it needs. These will be combined into a SchedClass.
 123: 
 124: //===----------------------------------------------------------------------===//
 125: // Stack allocation
 126: //===----------------------------------------------------------------------===//
 127: 
 128: // Pseudo -> LA / LAY
 129: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ADJDYNALLOC$")>;
 130: 
 131: //===----------------------------------------------------------------------===//
 132: // Branch instructions
 133: //===----------------------------------------------------------------------===//
 134: 
 135: // Branch
 136: def : InstRW<[WLat1, VBU, NormalGr], (instregex "(Call)?BRC(L)?(Asm.*)?$")>;
 137: def : InstRW<[WLat1, VBU, NormalGr], (instregex "(Call)?J(G)?(Asm.*)?$")>;
 138: def : InstRW<[WLat1, FXb, NormalGr], (instregex "(Call)?BC(R)?(Asm.*)?$")>;
 139: def : InstRW<[WLat1, FXb, NormalGr], (instregex "(Call)?B(R)?(Asm.*)?$")>;
 140: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "BI(C)?(Asm.*)?$")>;
 141: def : InstRW<[WLat1, FXa, EndGroup], (instregex "BRCT(G)?$")>;
 142: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BRCTH$")>;
 143: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BCT(G)?(R)?$")>;
 144: def : InstRW<[WLat1, FXa2, FXb2, GroupAlone2],
 145:              (instregex "B(R)?X(H|L).*$")>;
 146: 
 147: // Compare and branch
 148: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(L)?(G)?(I|R)J(Asm.*)?$")>;
 149: def : InstRW<[WLat1, FXb2, GroupAlone],
 150:              (instregex "C(L)?(G)?(I|R)B(Call|Return|Asm.*)?$")>;
 151: 
 152: //===----------------------------------------------------------------------===//
 153: // Trap instructions
 154: //===----------------------------------------------------------------------===//
 155: 
 156: // Trap
 157: def : InstRW<[WLat1, VBU, NormalGr], (instregex "(Cond)?Trap$")>;
 158: 
 159: // Compare and trap
 160: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(G)?(I|R)T(Asm.*)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 161-200 / 第 161-200 行
```tablegen
 161: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CL(G)?RT(Asm.*)?$")>;
 162: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CL(F|G)IT(Asm.*)?$")>;
 163: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "CL(G)?T(Asm.*)?$")>;
 164: 
 165: //===----------------------------------------------------------------------===//
 166: // Call and return instructions
 167: //===----------------------------------------------------------------------===//
 168: 
 169: // Call
 170: def : InstRW<[WLat1, VBU, FXa2, GroupAlone], (instregex "(Call)?BRAS$")>;
 171: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "(Call)?BRASL(_XPLINK64)?$")>;
 172: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "(Call)?BAS(R)?(_XPLINK64|_STACKEXT)?$")>;
 173: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "TLS_(G|L)DCALL$")>;
 174: 
 175: // Return
 176: def : InstRW<[WLat1, FXb, EndGroup], (instregex "Return(_XPLINK)?$")>;
 177: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CondReturn(_XPLINK)?$")>;
 178: 
 179: //===----------------------------------------------------------------------===//
 180: // Move instructions
 181: //===----------------------------------------------------------------------===//
 182: 
 183: // Moves
 184: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "MV(G|H)?HI$")>;
 185: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "MVI(Y)?$")>;
 186: 
 187: // Move character
 188: def : InstRW<[WLat1, FXb, LSU3, GroupAlone], (instregex "MVC$")>;
 189: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "MVCL(E|U)?$")>;
 190: 
 191: // Pseudo -> reg move
 192: def : InstRW<[WLat1, FXa, NormalGr], (instregex "COPY(_TO_REGCLASS)?$")>;
 193: def : InstRW<[WLat1, FXa, NormalGr], (instregex "EXTRACT_SUBREG$")>;
 194: def : InstRW<[WLat1, FXa, NormalGr], (instregex "INSERT_SUBREG$")>;
 195: def : InstRW<[WLat1, FXa, NormalGr], (instregex "REG_SEQUENCE$")>;
 196: 
 197: // Loads
 198: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L(Y|FH|RL|Mux)?$")>;
 199: def : InstRW<[LSULatency, LSULatency, LSU, NormalGr], (instregex "LCBB$")>;
 200: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LG(RL)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 201-240 / 第 201-240 行
```tablegen
 201: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L128$")>;
 202: 
 203: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLIH(F|H|L)$")>;
 204: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLIL(F|H|L)$")>;
 205: 
 206: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LG(F|H)I$")>;
 207: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LHI(Mux)?$")>;
 208: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LR$")>;
 209: 
 210: // Load and zero rightmost byte
 211: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LZR(F|G)$")>;
 212: 
 213: // Load and trap
 214: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "L(FH|G)?AT$")>;
 215: 
 216: // Load and test
 217: def : InstRW<[WLat1LSU, WLat1LSU, LSU, FXa, NormalGr], (instregex "LT(G)?$")>;
 218: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LT(G)?R$")>;
 219: 
 220: // Stores
 221: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STG(RL)?$")>;
 222: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST128$")>;
 223: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST(Y|FH|RL|Mux)?$")>;
 224: 
 225: // String moves.
 226: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "MVST$")>;
 227: 
 228: //===----------------------------------------------------------------------===//
 229: // Conditional move instructions
 230: //===----------------------------------------------------------------------===//
 231: 
 232: def : InstRW<[WLat2, FXa, NormalGr], (instregex "LOCRMux$")>;
 233: def : InstRW<[WLat2, FXa, NormalGr], (instregex "LOC(G|FH)?R(Asm.*)?$")>;
 234: def : InstRW<[WLat2, FXa, NormalGr], (instregex "LOC(G|H)?HI(Mux|(Asm.*))?$")>;
 235: def : InstRW<[WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 236:              (instregex "LOC(G|FH|Mux)?(Asm.*)?$")>;
 237: def : InstRW<[WLat1, FXb, LSU, NormalGr],
 238:              (instregex "STOC(G|FH|Mux)?(Asm.*)?$")>;
 239: 
 240: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 241-280 / 第 241-280 行
```tablegen
 241: // Sign extensions
 242: //===----------------------------------------------------------------------===//
 243: 
 244: def : InstRW<[WLat1, FXa, NormalGr], (instregex "L(B|H|G)R$")>;
 245: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LG(B|H|F)R$")>;
 246: 
 247: def : InstRW<[WLat1LSU, WLat1LSU, FXa, LSU, NormalGr], (instregex "LTGF$")>;
 248: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LTGFR$")>;
 249: 
 250: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LB(H|Mux)?$")>;
 251: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LH(Y)?$")>;
 252: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LH(H|Mux|RL)$")>;
 253: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LG(B|H|F)$")>;
 254: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LG(H|F)RL$")>;
 255: 
 256: //===----------------------------------------------------------------------===//
 257: // Zero extensions
 258: //===----------------------------------------------------------------------===//
 259: 
 260: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLCR(Mux)?$")>;
 261: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLHR(Mux)?$")>;
 262: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLG(C|H|F|T)R$")>;
 263: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLC(Mux)?$")>;
 264: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLH(Mux)?$")>;
 265: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LL(C|H)H$")>;
 266: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLHRL$")>;
 267: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLG(C|H|F|T|HRL|FRL)$")>;
 268: 
 269: // Load and zero rightmost byte
 270: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLZRGF$")>;
 271: 
 272: // Load and trap
 273: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "LLG(F|T)?AT$")>;
 274: 
 275: //===----------------------------------------------------------------------===//
 276: // Truncations
 277: //===----------------------------------------------------------------------===//
 278: 
 279: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STC(H|Y|Mux)?$")>;
 280: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STH(H|Y|RL|Mux)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 281-320 / 第 281-320 行
```tablegen
 281: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STCM(H|Y)?$")>;
 282: 
 283: //===----------------------------------------------------------------------===//
 284: // Multi-register moves
 285: //===----------------------------------------------------------------------===//
 286: 
 287: // Load multiple (estimated average of 5 ops)
 288: def : InstRW<[WLat10, WLat10, LSU5, GroupAlone], (instregex "LM(H|Y|G)?$")>;
 289: 
 290: // Load multiple disjoint
 291: def : InstRW<[WLat30, WLat30, MCD], (instregex "LMD$")>;
 292: 
 293: // Store multiple
 294: def : InstRW<[WLat1, LSU2, FXb3, GroupAlone], (instregex "STM(G|H|Y)?$")>;
 295: 
 296: //===----------------------------------------------------------------------===//
 297: // Byte swaps
 298: //===----------------------------------------------------------------------===//
 299: 
 300: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LRV(G)?R$")>;
 301: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LRV(G|H)?$")>;
 302: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STRV(G|H)?$")>;
 303: def : InstRW<[WLat30, MCD], (instregex "MVCIN$")>;
 304: 
 305: //===----------------------------------------------------------------------===//
 306: // Load address instructions
 307: //===----------------------------------------------------------------------===//
 308: 
 309: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LA(Y|RL)?$")>;
 310: 
 311: // Load the Global Offset Table address ( -> larl )
 312: def : InstRW<[WLat1, FXa, NormalGr], (instregex "GOT$")>;
 313: 
 314: //===----------------------------------------------------------------------===//
 315: // Absolute and Negation
 316: //===----------------------------------------------------------------------===//
 317: 
 318: def : InstRW<[WLat1, WLat1, FXa, NormalGr], (instregex "LP(G)?R$")>;
 319: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "L(N|P)GFR$")>;
 320: def : InstRW<[WLat1, WLat1, FXa, NormalGr], (instregex "LN(R|GR)$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 321-360 / 第 321-360 行
```tablegen
 321: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LC(R|GR)$")>;
 322: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "LCGFR$")>;
 323: 
 324: //===----------------------------------------------------------------------===//
 325: // Insertion
 326: //===----------------------------------------------------------------------===//
 327: 
 328: def : InstRW<[WLat1LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "IC(Y)?$")>;
 329: def : InstRW<[WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 330:              (instregex "IC32(Y)?$")>;
 331: def : InstRW<[WLat1LSU, RegReadAdv, WLat1LSU, FXa, LSU, NormalGr],
 332:              (instregex "ICM(H|Y)?$")>;
 333: def : InstRW<[WLat1, FXa, NormalGr], (instregex "II(F|H|L)Mux$")>;
 334: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHF(64)?$")>;
 335: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHH(64)?$")>;
 336: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHL(64)?$")>;
 337: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILF(64)?$")>;
 338: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILH(64)?$")>;
 339: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILL(64)?$")>;
 340: 
 341: //===----------------------------------------------------------------------===//
 342: // Addition
 343: //===----------------------------------------------------------------------===//
 344: 
 345: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 346:              (instregex "A(Y)?$")>;
 347: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 348:              (instregex "AH(Y)?$")>;
 349: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AIH$")>;
 350: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AFI(Mux)?$")>;
 351: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 352:              (instregex "AG$")>;
 353: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGFI$")>;
 354: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGHI(K)?$")>;
 355: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGR(K)?$")>;
 356: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AHI(K)?$")>;
 357: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AHIMux(K)?$")>;
 358: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 359:              (instregex "AL(Y)?$")>;
 360: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AL(FI|HSIK)$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 361-400 / 第 361-400 行
```tablegen
 361: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 362:              (instregex "ALG(F)?$")>;
 363: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGHSIK$")>;
 364: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGF(I|R)$")>;
 365: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGR(K)?$")>;
 366: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALR(K)?$")>;
 367: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AR(K)?$")>;
 368: def : InstRW<[WLat1, FXa, NormalGr], (instregex "A(L)?HHHR$")>;
 369: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "A(L)?HHLR$")>;
 370: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALSIH(N)?$")>;
 371: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "A(L)?(G)?SI$")>;
 372: 
 373: // Logical addition with carry
 374: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, GroupAlone],
 375:              (instregex "ALC(G)?$")>;
 376: def : InstRW<[WLat2, WLat2, FXa, GroupAlone], (instregex "ALC(G)?R$")>;
 377: 
 378: // Add with sign extension (16/32 -> 64)
 379: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 380:              (instregex "AG(F|H)$")>;
 381: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "AGFR$")>;
 382: 
 383: //===----------------------------------------------------------------------===//
 384: // Subtraction
 385: //===----------------------------------------------------------------------===//
 386: 
 387: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 388:              (instregex "S(G|Y)?$")>;
 389: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 390:              (instregex "SH(Y)?$")>;
 391: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SGR(K)?$")>;
 392: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLFI$")>;
 393: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 394:              (instregex "SL(G|GF|Y)?$")>;
 395: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLGF(I|R)$")>;
 396: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLGR(K)?$")>;
 397: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLR(K)?$")>;
 398: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SR(K)?$")>;
 399: def : InstRW<[WLat1, FXa, NormalGr], (instregex "S(L)?HHHR$")>;
 400: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "S(L)?HHLR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 401-440 / 第 401-440 行
```tablegen
 401: 
 402: // Subtraction with borrow
 403: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, GroupAlone],
 404:              (instregex "SLB(G)?$")>;
 405: def : InstRW<[WLat2, WLat2, FXa, GroupAlone], (instregex "SLB(G)?R$")>;
 406: 
 407: // Subtraction with sign extension (16/32 -> 64)
 408: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 409:              (instregex "SG(F|H)$")>;
 410: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "SGFR$")>;
 411: 
 412: //===----------------------------------------------------------------------===//
 413: // AND
 414: //===----------------------------------------------------------------------===//
 415: 
 416: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 417:              (instregex "N(G|Y)?$")>;
 418: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NGR(K)?$")>;
 419: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NI(FMux|HMux|LMux)$")>;
 420: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "NI(Y)?$")>;
 421: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHF(64)?$")>;
 422: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHH(64)?$")>;
 423: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHL(64)?$")>;
 424: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILF(64)?$")>;
 425: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILH(64)?$")>;
 426: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILL(64)?$")>;
 427: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NR(K)?$")>;
 428: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "NC$")>;
 429: 
 430: //===----------------------------------------------------------------------===//
 431: // OR
 432: //===----------------------------------------------------------------------===//
 433: 
 434: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 435:              (instregex "O(G|Y)?$")>;
 436: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OGR(K)?$")>;
 437: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "OI(Y)?$")>;
 438: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OI(FMux|HMux|LMux)$")>;
 439: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHF(64)?$")>;
 440: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHH(64)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 441-480 / 第 441-480 行
```tablegen
 441: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHL(64)?$")>;
 442: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILF(64)?$")>;
 443: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILH(64)?$")>;
 444: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILL(64)?$")>;
 445: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OR(K)?$")>;
 446: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "OC$")>;
 447: 
 448: //===----------------------------------------------------------------------===//
 449: // XOR
 450: //===----------------------------------------------------------------------===//
 451: 
 452: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 453:              (instregex "X(G|Y)?$")>;
 454: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "XI(Y)?$")>;
 455: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XIFMux$")>;
 456: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XGR(K)?$")>;
 457: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XIHF(64)?$")>;
 458: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XILF(64)?$")>;
 459: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XR(K)?$")>;
 460: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "XC$")>;
 461: 
 462: //===----------------------------------------------------------------------===//
 463: // Multiplication
 464: //===----------------------------------------------------------------------===//
 465: 
 466: def : InstRW<[WLat5LSU, RegReadAdv, FXa, LSU, NormalGr],
 467:              (instregex "MS(GF|Y)?$")>;
 468: def : InstRW<[WLat5, FXa, NormalGr], (instregex "MS(R|FI)$")>;
 469: def : InstRW<[WLat7LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "MSG$")>;
 470: def : InstRW<[WLat7, FXa, NormalGr], (instregex "MSGR$")>;
 471: def : InstRW<[WLat5, FXa, NormalGr], (instregex "MSGF(I|R)$")>;
 472: def : InstRW<[WLat8LSU, RegReadAdv, FXa2, LSU, GroupAlone], (instregex "MLG$")>;
 473: def : InstRW<[WLat8, FXa2, GroupAlone], (instregex "MLGR$")>;
 474: def : InstRW<[WLat4, FXa, NormalGr], (instregex "MGHI$")>;
 475: def : InstRW<[WLat4, FXa, NormalGr], (instregex "MHI$")>;
 476: def : InstRW<[WLat4LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "MH(Y)?$")>;
 477: def : InstRW<[WLat6, FXa2, GroupAlone], (instregex "M(L)?R$")>;
 478: def : InstRW<[WLat6LSU, RegReadAdv, FXa2, LSU, GroupAlone],
 479:              (instregex "M(FY|L)?$")>;
 480: def : InstRW<[WLat8, RegReadAdv, FXa, LSU, NormalGr], (instregex "MGH$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: def : InstRW<[WLat12, RegReadAdv, FXa2, LSU, GroupAlone], (instregex "MG$")>;
 482: def : InstRW<[WLat8, FXa2, GroupAlone], (instregex "MGRK$")>;
 483: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, FXa, LSU, NormalGr],
 484:              (instregex "MSC$")>;
 485: def : InstRW<[WLat8LSU, WLat8LSU, RegReadAdv, FXa, LSU, NormalGr],
 486:              (instregex "MSGC$")>;
 487: def : InstRW<[WLat6, WLat6, FXa, NormalGr], (instregex "MSRKC$")>;
 488: def : InstRW<[WLat8, WLat8, FXa, NormalGr], (instregex "MSGRKC$")>;
 489: 
 490: //===----------------------------------------------------------------------===//
 491: // Division and remainder
 492: //===----------------------------------------------------------------------===//
 493: 
 494: def : InstRW<[WLat20, FXa4, GroupAlone], (instregex "DR$")>;
 495: def : InstRW<[WLat30, RegReadAdv, FXa4, LSU, GroupAlone2], (instregex "D$")>;
 496: def : InstRW<[WLat30, FXa2, GroupAlone], (instregex "DSG(F)?R$")>;
 497: def : InstRW<[WLat30, RegReadAdv, FXa2, LSU, GroupAlone2],
 498:              (instregex "DSG(F)?$")>;
 499: def : InstRW<[WLat20, FXa4, GroupAlone], (instregex "DLR$")>;
 500: def : InstRW<[WLat30, FXa4, GroupAlone], (instregex "DLGR$")>;
 501: def : InstRW<[WLat30, RegReadAdv, FXa4, LSU, GroupAlone2],
 502:              (instregex "DL(G)?$")>;
 503: 
 504: //===----------------------------------------------------------------------===//
 505: // Shifts
 506: //===----------------------------------------------------------------------===//
 507: 
 508: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLL(G|K)?$")>;
 509: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SRL(G|K)?$")>;
 510: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SRA(G|K)?$")>;
 511: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLA(G|K)?$")>;
 512: def : InstRW<[WLat5LSU, WLat5LSU, FXa4, LSU, GroupAlone2],
 513:              (instregex "S(L|R)D(A|L)$")>;
 514: 
 515: // Rotate
 516: def : InstRW<[WLat2LSU, FXa, LSU, NormalGr], (instregex "RLL(G)?$")>;
 517: 
 518: // Rotate and insert
 519: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBH(G|H|L)(Opt)?$")>;
 520: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBL(G|H|L)(Opt)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 521-560 / 第 521-560 行
```tablegen
 521: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBG(N|32)?(Z)?(Opt)?$")>;
 522: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBMux$")>;
 523: 
 524: // Rotate and Select
 525: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "R(N|O|X)SBG(Opt)?$")>;
 526: 
 527: //===----------------------------------------------------------------------===//
 528: // Comparison
 529: //===----------------------------------------------------------------------===//
 530: 
 531: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr],
 532:              (instregex "C(G|Y|Mux)?$")>;
 533: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CRL$")>;
 534: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(F|H)I(Mux)?$")>;
 535: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CG(F|H)I$")>;
 536: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CG(HSI|RL)$")>;
 537: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(G)?R$")>;
 538: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CIH$")>;
 539: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CHF$")>;
 540: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CHSI$")>;
 541: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr],
 542:              (instregex "CL(Y|Mux)?$")>;
 543: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLFHSI$")>;
 544: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLFI(Mux)?$")>;
 545: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLG$")>;
 546: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLG(HRL|HSI)$")>;
 547: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLGF$")>;
 548: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLGFRL$")>;
 549: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLGF(I|R)$")>;
 550: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLGR$")>;
 551: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLGRL$")>;
 552: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLHF$")>;
 553: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLH(RL|HSI)$")>;
 554: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLIH$")>;
 555: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLI(Y)?$")>;
 556: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLR$")>;
 557: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLRL$")>;
 558: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(L)?HHR$")>;
 559: def : InstRW<[WLat2, FXb, NormalGr], (instregex "C(L)?HLR$")>;
 560: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: // Compare halfword
 562: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CH(Y)?$")>;
 563: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CHRL$")>;
 564: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CGH$")>;
 565: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CGHRL$")>;
 566: def : InstRW<[WLat2LSU, FXa, FXb, LSU, Cracked], (instregex "CHHSI$")>;
 567: 
 568: // Compare with sign extension (32 -> 64)
 569: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CGF$")>;
 570: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CGFRL$")>;
 571: def : InstRW<[WLat2, FXb, NormalGr], (instregex "CGFR$")>;
 572: 
 573: // Compare logical character
 574: def : InstRW<[WLat6, FXb, LSU2, Cracked], (instregex "CLC$")>;
 575: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLCL(E|U)?$")>;
 576: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLST$")>;
 577: 
 578: // Test under mask
 579: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "TM(Y)?$")>;
 580: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TM(H|L)Mux$")>;
 581: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMHH(64)?$")>;
 582: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMHL(64)?$")>;
 583: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMLH(64)?$")>;
 584: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMLL(64)?$")>;
 585: 
 586: // Compare logical characters under mask
 587: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr],
 588:              (instregex "CLM(H|Y)?$")>;
 589: 
 590: //===----------------------------------------------------------------------===//
 591: // Prefetch and execution hint
 592: //===----------------------------------------------------------------------===//
 593: 
 594: def : InstRW<[WLat1, LSU, NormalGr], (instregex "PFD(RL)?$")>;
 595: def : InstRW<[WLat1, FXb, NormalGr], (instregex "BPP$")>;
 596: def : InstRW<[FXb, EndGroup], (instregex "BPRP$")>;
 597: def : InstRW<[WLat1, FXb, NormalGr], (instregex "NIAI$")>;
 598: 
 599: //===----------------------------------------------------------------------===//
 600: // Atomic operations
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 601-640 / 第 601-640 行
```tablegen
 601: //===----------------------------------------------------------------------===//
 602: 
 603: def : InstRW<[WLat1, FXb, EndGroup], (instregex "Serialize$")>;
 604: 
 605: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAA(G)?$")>;
 606: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAAL(G)?$")>;
 607: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAN(G)?$")>;
 608: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAO(G)?$")>;
 609: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAX(G)?$")>;
 610: 
 611: // Test and set
 612: def : InstRW<[WLat2LSU, FXb, LSU, EndGroup], (instregex "TS$")>;
 613: 
 614: // Compare and swap
 615: def : InstRW<[WLat3LSU, WLat3LSU, FXa, FXb, LSU, GroupAlone],
 616:              (instregex "CS(G|Y)?$")>;
 617: 
 618: // Compare double and swap
 619: def : InstRW<[WLat6LSU, WLat6LSU, FXa3, FXb2, LSU, GroupAlone2],
 620:              (instregex "CDS(Y)?$")>;
 621: def : InstRW<[WLat15, WLat15, FXa2, FXb4, LSU3,
 622:               GroupAlone3], (instregex "CDSG$")>;
 623: 
 624: // Compare and swap and store
 625: def : InstRW<[WLat30, MCD], (instregex "CSST$")>;
 626: 
 627: // Perform locked operation
 628: def : InstRW<[WLat30, MCD], (instregex "PLO$")>;
 629: 
 630: // Load/store pair from/to quadword
 631: def : InstRW<[WLat4LSU, LSU2, GroupAlone], (instregex "LPQ$")>;
 632: def : InstRW<[WLat1, FXb2, LSU, GroupAlone], (instregex "STPQ$")>;
 633: 
 634: // Load pair disjoint
 635: def : InstRW<[WLat1LSU, WLat1LSU, LSU2, GroupAlone], (instregex "LPD(G)?$")>;
 636: 
 637: //===----------------------------------------------------------------------===//
 638: // Translate and convert
 639: //===----------------------------------------------------------------------===//
 640: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 641-680 / 第 641-680 行
```tablegen
 641: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "TR$")>;
 642: def : InstRW<[WLat30, WLat30, WLat30, FXa3, LSU2, GroupAlone2],
 643:              (instregex "TRT$")>;
 644: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRTR$")>;
 645: def : InstRW<[WLat30, WLat30, MCD], (instregex "TRE$")>;
 646: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRT(R)?E(Opt)?$")>;
 647: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TR(T|O)(T|O)(Opt)?$")>;
 648: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 649:              (instregex "CU(12|14|21|24|41|42)(Opt)?$")>;
 650: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "(CUUTF|CUTFU)(Opt)?$")>;
 651: 
 652: //===----------------------------------------------------------------------===//
 653: // Message-security assist
 654: //===----------------------------------------------------------------------===//
 655: 
 656: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD],
 657:              (instregex "KM(C|F|O|CTR|A)?$")>;
 658: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 659:              (instregex "(KIMD|KLMD|KMAC)$")>;
 660: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 661:              (instregex "(PCC|PPNO|PRNO)$")>;
 662: 
 663: //===----------------------------------------------------------------------===//
 664: // Guarded storage
 665: //===----------------------------------------------------------------------===//
 666: 
 667: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LGG$")>;
 668: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLGFSG$")>;
 669: def : InstRW<[WLat30, MCD], (instregex "(L|ST)GSC$")>;
 670: 
 671: //===----------------------------------------------------------------------===//
 672: // Decimal arithmetic
 673: //===----------------------------------------------------------------------===//
 674: 
 675: def : InstRW<[WLat30, RegReadAdv, FXb, VecDF2, LSU2, GroupAlone2],
 676:              (instregex "CVBG$")>;
 677: def : InstRW<[WLat30, RegReadAdv, FXb, VecDF, LSU, GroupAlone2],
 678:              (instregex "CVB(Y)?$")>;
 679: def : InstRW<[WLat1, FXb3, VecDF4, LSU, GroupAlone3], (instregex "CVDG$")>;
 680: def : InstRW<[WLat1, FXb2, VecDF, LSU, GroupAlone2], (instregex "CVD(Y)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 681-720 / 第 681-720 行
```tablegen
 681: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "MV(N|O|Z)$")>;
 682: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "(PACK|PKA|PKU)$")>;
 683: def : InstRW<[WLat12, LSU5, GroupAlone], (instregex "UNPK(A|U)$")>;
 684: def : InstRW<[WLat1, FXb, LSU2, Cracked], (instregex "UNPK$")>;
 685: 
 686: def : InstRW<[WLat5LSU, FXb, VecDFX, LSU3, GroupAlone2],
 687:              (instregex "(A|S|ZA)P$")>;
 688: def : InstRW<[WLat1, FXb, VecDFX4, LSU3, GroupAlone2], (instregex "(M|D)P$")>;
 689: def : InstRW<[WLat15, FXb, VecDFX2, LSU2, GroupAlone3], (instregex "SRP$")>;
 690: def : InstRW<[WLat8, VecDFX, LSU, LSU, GroupAlone], (instregex "CP$")>;
 691: def : InstRW<[WLat3LSU, VecDFX, LSU, Cracked], (instregex "TP$")>;
 692: def : InstRW<[WLat30, MCD], (instregex "ED(MK)?$")>;
 693: 
 694: //===----------------------------------------------------------------------===//
 695: // Access registers
 696: //===----------------------------------------------------------------------===//
 697: 
 698: // Extract/set/copy access register
 699: def : InstRW<[WLat3, LSU, NormalGr], (instregex "(EAR|SAR|CPYA)$")>;
 700: 
 701: // Load address extended
 702: def : InstRW<[WLat5, LSU, FXa, Cracked], (instregex "LAE(Y)?$")>;
 703: 
 704: // Load/store access multiple (not modeled precisely)
 705: def : InstRW<[WLat20, WLat20, LSU5, GroupAlone], (instregex "LAM(Y)?$")>;
 706: def : InstRW<[WLat1, LSU5, FXb, GroupAlone2], (instregex "STAM(Y)?$")>;
 707: 
 708: //===----------------------------------------------------------------------===//
 709: // Program mask and addressing mode
 710: //===----------------------------------------------------------------------===//
 711: 
 712: // Insert Program Mask
 713: def : InstRW<[WLat3, FXa, EndGroup], (instregex "IPM$")>;
 714: 
 715: // Set Program Mask
 716: def : InstRW<[WLat3, LSU, EndGroup], (instregex "SPM$")>;
 717: 
 718: // Branch and link
 719: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "BAL(R)?$")>;
 720: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: // Test addressing mode
 722: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TAM$")>;
 723: 
 724: // Set addressing mode
 725: def : InstRW<[WLat1, FXb, EndGroup], (instregex "SAM(24|31|64)$")>;
 726: 
 727: // Branch (and save) and set mode.
 728: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BSM$")>;
 729: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "BASSM$")>;
 730: 
 731: //===----------------------------------------------------------------------===//
 732: // Transactional execution
 733: //===----------------------------------------------------------------------===//
 734: 
 735: // Transaction begin
 736: def : InstRW<[WLat9, LSU2, FXb5, GroupAlone2], (instregex "TBEGIN(C)?$")>;
 737: 
 738: // Transaction end
 739: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "TEND$")>;
 740: 
 741: // Transaction abort
 742: def : InstRW<[WLat30, MCD], (instregex "TABORT$")>;
 743: 
 744: // Extract Transaction Nesting Depth
 745: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ETND$")>;
 746: 
 747: // Nontransactional store
 748: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "NTSTG$")>;
 749: 
 750: //===----------------------------------------------------------------------===//
 751: // Processor assist
 752: //===----------------------------------------------------------------------===//
 753: 
 754: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "PPA$")>;
 755: 
 756: //===----------------------------------------------------------------------===//
 757: // Miscellaneous Instructions.
 758: //===----------------------------------------------------------------------===//
 759: 
 760: // Find leftmost one
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 761-800 / 第 761-800 行
```tablegen
 761: def : InstRW<[WLat5, WLat5, FXa2, GroupAlone], (instregex "FLOGR$")>;
 762: 
 763: // Population count
 764: def : InstRW<[WLat3, WLat3, FXa, NormalGr], (instregex "POPCNT$")>;
 765: 
 766: // String instructions
 767: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "SRST(U)?$")>;
 768: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CUSE$")>;
 769: 
 770: // Various complex instructions
 771: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CFC$")>;
 772: def : InstRW<[WLat30, WLat30, WLat30, WLat30, WLat30, WLat30, MCD],
 773:              (instregex "UPT$")>;
 774: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CKSM$")>;
 775: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CMPSC$")>;
 776: 
 777: // Execute
 778: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "EX(RL)?$")>;
 779: 
 780: //===----------------------------------------------------------------------===//
 781: // .insn directive instructions
 782: //===----------------------------------------------------------------------===//
 783: 
 784: // An "empty" sched-class will be assigned instead of the "invalid sched-class".
 785: // getNumDecoderSlots() will then return 1 instead of 0.
 786: def : InstRW<[], (instregex "Insn.*")>;
 787: 
 788: 
 789: // ----------------------------- Floating point ----------------------------- //
 790: 
 791: //===----------------------------------------------------------------------===//
 792: // FP: Move instructions
 793: //===----------------------------------------------------------------------===//
 794: 
 795: // Load zero
 796: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LZ(DR|ER|ER_16)$")>;
 797: def : InstRW<[WLat2, FXb2, Cracked], (instregex "LZXR$")>;
 798: 
 799: // Load
 800: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "LER(16)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LD(R|R16|R32|GR)$")>;
 802: def : InstRW<[WLat3, FXb, NormalGr], (instregex "LGDR$")>;
 803: def : InstRW<[WLat2, FXb2, GroupAlone], (instregex "LXR$")>;
 804: 
 805: // Load and Test
 806: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "LT(E|D)BR$")>;
 807: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXBR$")>;
 808: 
 809: // Copy sign
 810: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "CPSDR(d|s|h)(d|s|h)$")>;
 811: 
 812: //===----------------------------------------------------------------------===//
 813: // FP: Load instructions
 814: //===----------------------------------------------------------------------===//
 815: 
 816: def : InstRW<[WLat2LSU, VecXsPm, LSU, NormalGr], (instregex "L(E16|E)(Y)?$")>;
 817: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LD(Y|E32)?$")>;
 818: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LX$")>;
 819: 
 820: //===----------------------------------------------------------------------===//
 821: // FP: Store instructions
 822: //===----------------------------------------------------------------------===//
 823: 
 824: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST(E16|E|D)(Y)?$")>;
 825: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STX$")>;
 826: 
 827: //===----------------------------------------------------------------------===//
 828: // FP: Conversion instructions
 829: //===----------------------------------------------------------------------===//
 830: 
 831: // Load rounded
 832: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "LEDBR(A)?$")>;
 833: def : InstRW<[WLat9, VecDF2, NormalGr], (instregex "L(E|D)XBR(A)?$")>;
 834: 
 835: // Load lengthened
 836: def : InstRW<[WLat7LSU, VecBF, LSU, NormalGr], (instregex "LDEB$")>;
 837: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "LDEBR$")>;
 838: def : InstRW<[WLat8LSU, VecBF4, LSU, GroupAlone], (instregex "LX(E|D)B$")>;
 839: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "LX(E|D)BR$")>;
 840: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: // Convert from fixed / logical
 842: def : InstRW<[WLat8, FXb, VecBF, Cracked], (instregex "C(E|D)(F|G)BR(A)?$")>;
 843: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)BR(A)?$")>;
 844: def : InstRW<[WLat8, FXb, VecBF, Cracked], (instregex "C(E|D)L(F|G)BR$")>;
 845: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CXL(F|G)BR$")>;
 846: 
 847: // Convert to fixed / logical
 848: def : InstRW<[WLat10, WLat10, FXb, VecBF, Cracked],
 849:              (instregex "C(F|G)(E|D)BR(A)?$")>;
 850: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked],
 851:              (instregex "C(F|G)XBR(A)?$")>;
 852: def : InstRW<[WLat10, WLat10, FXb, VecBF, GroupAlone], (instregex "CLFEBR$")>;
 853: def : InstRW<[WLat10, WLat10, FXb, VecBF, Cracked], (instregex "CLFDBR$")>;
 854: def : InstRW<[WLat10, WLat10, FXb, VecBF, Cracked], (instregex "CLG(E|D)BR$")>;
 855: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked], (instregex "CL(F|G)XBR$")>;
 856: 
 857: //===----------------------------------------------------------------------===//
 858: // FP: Unary arithmetic
 859: //===----------------------------------------------------------------------===//
 860: 
 861: // Load Complement / Negative / Positive
 862: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "L(C|N|P)(E|D)BR$")>;
 863: def : InstRW<[WLat1, FXb, NormalGr], (instregex "L(C|N|P)DFR(_32|_16)?$")>;
 864: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "L(C|N|P)XBR$")>;
 865: 
 866: // Square root
 867: def : InstRW<[WLat30, VecFPd, LSU, NormalGr], (instregex "SQ(E|D)B$")>;
 868: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "SQ(E|D)BR$")>;
 869: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "SQXBR$")>;
 870: 
 871: // Load FP integer
 872: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "FI(E|D)BR(A)?$")>;
 873: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXBR(A)?$")>;
 874: 
 875: //===----------------------------------------------------------------------===//
 876: // FP: Binary arithmetic
 877: //===----------------------------------------------------------------------===//
 878: 
 879: // Addition
 880: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 881-920 / 第 881-920 行
```tablegen
 881:              (instregex "A(E|D)B$")>;
 882: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "A(E|D)BR$")>;
 883: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXBR$")>;
 884: 
 885: // Subtraction
 886: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
 887:              (instregex "S(E|D)B$")>;
 888: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "S(E|D)BR$")>;
 889: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXBR$")>;
 890: 
 891: // Multiply
 892: def : InstRW<[WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
 893:              (instregex "M(D|DE|EE)B$")>;
 894: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "M(D|DE|EE)BR$")>;
 895: def : InstRW<[WLat8LSU, RegReadAdv, VecBF4, LSU, GroupAlone],
 896:              (instregex "MXDB$")>;
 897: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "MXDBR$")>;
 898: def : InstRW<[WLat20, VecDF4, GroupAlone], (instregex "MXBR$")>;
 899: 
 900: // Multiply and add / subtract
 901: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
 902:              (instregex "M(A|S)EB$")>;
 903: def : InstRW<[WLat7, VecBF, GroupAlone], (instregex "M(A|S)EBR$")>;
 904: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
 905:              (instregex "M(A|S)DB$")>;
 906: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "M(A|S)DBR$")>;
 907: 
 908: // Division
 909: def : InstRW<[WLat30, RegReadAdv, VecFPd, LSU, NormalGr],
 910:              (instregex "D(E|D)B$")>;
 911: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "D(E|D)BR$")>;
 912: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "DXBR$")>;
 913: 
 914: // Divide to integer
 915: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "DI(E|D)BR$")>;
 916: 
 917: //===----------------------------------------------------------------------===//
 918: // FP: Comparisons
 919: //===----------------------------------------------------------------------===//
 920: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: // Compare
 922: def : InstRW<[WLat3LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
 923:              (instregex "(K|C)(E|D)B$")>;
 924: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "(K|C)(E|D)BR$")>;
 925: def : InstRW<[WLat9, VecDF2, GroupAlone], (instregex "(K|C)XBR$")>;
 926: 
 927: // Test Data Class
 928: def : InstRW<[WLat5, LSU, VecXsPm, NormalGr], (instregex "TC(E|D)B$")>;
 929: def : InstRW<[WLat10, LSU, VecDF4, GroupAlone], (instregex "TCXB$")>;
 930: 
 931: //===----------------------------------------------------------------------===//
 932: // FP: Floating-point control register instructions
 933: //===----------------------------------------------------------------------===//
 934: 
 935: def : InstRW<[WLat4, FXa, LSU, GroupAlone], (instregex "EFPC$")>;
 936: def : InstRW<[WLat1, FXb, LSU, GroupAlone], (instregex "STFPC$")>;
 937: def : InstRW<[WLat3, LSU, GroupAlone], (instregex "SFPC$")>;
 938: def : InstRW<[WLat3LSU, LSU2, GroupAlone], (instregex "LFPC$")>;
 939: def : InstRW<[WLat30, MCD], (instregex "SFASR$")>;
 940: def : InstRW<[WLat30, MCD], (instregex "LFAS$")>;
 941: def : InstRW<[WLat3, FXb, GroupAlone], (instregex "SRNM(B|T)?$")>;
 942: 
 943: 
 944: // --------------------- Hexadecimal floating point ------------------------- //
 945: 
 946: //===----------------------------------------------------------------------===//
 947: // HFP: Move instructions
 948: //===----------------------------------------------------------------------===//
 949: 
 950: // Load and Test
 951: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "LT(E|D)R$")>;
 952: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXR$")>;
 953: 
 954: //===----------------------------------------------------------------------===//
 955: // HFP: Conversion instructions
 956: //===----------------------------------------------------------------------===//
 957: 
 958: // Load rounded
 959: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "(LEDR|LRER)$")>;
 960: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "LEXR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: def : InstRW<[WLat9, VecDF2, NormalGr], (instregex "(LDXR|LRDR)$")>;
 962: 
 963: // Load lengthened
 964: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LDE$")>;
 965: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LDER$")>;
 966: def : InstRW<[WLat8LSU, VecBF4, LSU, GroupAlone], (instregex "LX(E|D)$")>;
 967: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "LX(E|D)R$")>;
 968: 
 969: // Convert from fixed
 970: def : InstRW<[WLat8, FXb, VecBF, Cracked], (instregex "C(E|D)(F|G)R$")>;
 971: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)R$")>;
 972: 
 973: // Convert to fixed
 974: def : InstRW<[WLat10, WLat10, FXb, VecBF, Cracked], (instregex "C(F|G)(E|D)R$")>;
 975: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked], (instregex "C(F|G)XR$")>;
 976: 
 977: // Convert BFP to HFP / HFP to BFP.
 978: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "THD(E)?R$")>;
 979: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "TB(E)?DR$")>;
 980: 
 981: //===----------------------------------------------------------------------===//
 982: // HFP: Unary arithmetic
 983: //===----------------------------------------------------------------------===//
 984: 
 985: // Load Complement / Negative / Positive
 986: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "L(C|N|P)(E|D)R$")>;
 987: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "L(C|N|P)XR$")>;
 988: 
 989: // Halve
 990: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "H(E|D)R$")>;
 991: 
 992: // Square root
 993: def : InstRW<[WLat30, VecFPd, LSU, NormalGr], (instregex "SQ(E|D)$")>;
 994: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "SQ(E|D)R$")>;
 995: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "SQXR$")>;
 996: 
 997: // Load FP integer
 998: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "FI(E|D)R$")>;
 999: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXR$")>;
1000: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: //===----------------------------------------------------------------------===//
1002: // HFP: Binary arithmetic
1003: //===----------------------------------------------------------------------===//
1004: 
1005: // Addition
1006: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
1007:              (instregex "A(E|D|U|W)$")>;
1008: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "A(E|D|U|W)R$")>;
1009: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXR$")>;
1010: 
1011: // Subtraction
1012: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
1013:              (instregex "S(E|D|U|W)$")>;
1014: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "S(E|D|U|W)R$")>;
1015: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXR$")>;
1016: 
1017: // Multiply
1018: def : InstRW<[WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
1019:              (instregex "M(D|DE|E|EE)$")>;
1020: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "M(D|DE|E|EE)R$")>;
1021: def : InstRW<[WLat8LSU, RegReadAdv, VecBF4, LSU, GroupAlone],
1022:              (instregex "MXD$")>;
1023: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "MXDR$")>;
1024: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "MXR$")>;
1025: def : InstRW<[WLat8LSU, RegReadAdv, VecBF4, LSU, GroupAlone], (instregex "MY$")>;
1026: def : InstRW<[WLat7LSU, RegReadAdv, VecBF2, LSU, GroupAlone],
1027:              (instregex "MY(H|L)$")>;
1028: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "MYR$")>;
1029: def : InstRW<[WLat7, VecBF, GroupAlone], (instregex "MY(H|L)R$")>;
1030: 
1031: // Multiply and add / subtract
1032: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
1033:              (instregex "M(A|S)(E|D)$")>;
1034: def : InstRW<[WLat7, VecBF, GroupAlone], (instregex "M(A|S)(E|D)R$")>;
1035: def : InstRW<[WLat8LSU, RegReadAdv, RegReadAdv, VecBF4, LSU, GroupAlone],
1036:              (instregex "MAY$")>;
1037: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
1038:              (instregex "MAY(H|L)$")>;
1039: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "MAYR$")>;
1040: def : InstRW<[WLat7, VecBF, GroupAlone], (instregex "MAY(H|L)R$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: 
1042: // Division
1043: def : InstRW<[WLat30, RegReadAdv, VecFPd, LSU, NormalGr], (instregex "D(E|D)$")>;
1044: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "D(E|D)R$")>;
1045: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "DXR$")>;
1046: 
1047: //===----------------------------------------------------------------------===//
1048: // HFP: Comparisons
1049: //===----------------------------------------------------------------------===//
1050: 
1051: // Compare
1052: def : InstRW<[WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
1053:              (instregex "C(E|D)$")>;
1054: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "C(E|D)R$")>;
1055: def : InstRW<[WLat10, VecDF2, GroupAlone], (instregex "CXR$")>;
1056: 
1057: 
1058: // ------------------------ Decimal floating point -------------------------- //
1059: 
1060: //===----------------------------------------------------------------------===//
1061: // DFP: Move instructions
1062: //===----------------------------------------------------------------------===//
1063: 
1064: // Load and Test
1065: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "LTDTR$")>;
1066: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXTR$")>;
1067: 
1068: //===----------------------------------------------------------------------===//
1069: // DFP: Conversion instructions
1070: //===----------------------------------------------------------------------===//
1071: 
1072: // Load rounded
1073: def : InstRW<[WLat15, VecDF, NormalGr], (instregex "LEDTR$")>;
1074: def : InstRW<[WLat15, VecDF2, NormalGr], (instregex "LDXTR$")>;
1075: 
1076: // Load lengthened
1077: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "LDETR$")>;
1078: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "LXDTR$")>;
1079: 
1080: // Convert from fixed / logical
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: def : InstRW<[WLat30, FXb, VecDF, Cracked], (instregex "CD(F|G)TR(A)?$")>;
1082: def : InstRW<[WLat30, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)TR(A)?$")>;
1083: def : InstRW<[WLat30, FXb, VecDF, Cracked], (instregex "CDL(F|G)TR$")>;
1084: def : InstRW<[WLat30, FXb, VecDF4, GroupAlone2], (instregex "CXL(F|G)TR$")>;
1085: 
1086: // Convert to fixed / logical
1087: def : InstRW<[WLat30, WLat30, FXb, VecDF, Cracked],
1088:              (instregex "C(F|G)DTR(A)?$")>;
1089: def : InstRW<[WLat30, WLat30, FXb, VecDF2, Cracked],
1090:              (instregex "C(F|G)XTR(A)?$")>;
1091: def : InstRW<[WLat30, WLat30, FXb, VecDF, Cracked], (instregex "CL(F|G)DTR$")>;
1092: def : InstRW<[WLat30, WLat30, FXb, VecDF2, Cracked], (instregex "CL(F|G)XTR$")>;
1093: 
1094: // Convert from / to signed / unsigned packed
1095: def : InstRW<[WLat9, FXb, VecDF, Cracked], (instregex "CD(S|U)TR$")>;
1096: def : InstRW<[WLat12, FXb2, VecDF4, GroupAlone2], (instregex "CX(S|U)TR$")>;
1097: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "C(S|U)DTR$")>;
1098: def : InstRW<[WLat15, FXb2, VecDF4, GroupAlone2], (instregex "C(S|U)XTR$")>;
1099: 
1100: // Convert from / to zoned
1101: def : InstRW<[WLat8LSU, LSU, VecDF, Cracked], (instregex "CDZT$")>;
1102: def : InstRW<[WLat16LSU, LSU2, VecDF4, GroupAlone3], (instregex "CXZT$")>;
1103: def : InstRW<[WLat1, FXb, LSU, VecDF, Cracked], (instregex "CZDT$")>;
1104: def : InstRW<[WLat1, FXb, LSU, VecDF2, GroupAlone], (instregex "CZXT$")>;
1105: 
1106: // Convert from / to packed
1107: def : InstRW<[WLat8LSU, LSU, VecDF, Cracked], (instregex "CDPT$")>;
1108: def : InstRW<[WLat16LSU, LSU2, VecDF4, GroupAlone3], (instregex "CXPT$")>;
1109: def : InstRW<[WLat1, FXb, LSU, VecDF, Cracked], (instregex "CPDT$")>;
1110: def : InstRW<[WLat1, FXb, LSU, VecDF2, GroupAlone], (instregex "CPXT$")>;
1111: 
1112: // Perform floating-point operation
1113: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "PFPO$")>;
1114: 
1115: //===----------------------------------------------------------------------===//
1116: // DFP: Unary arithmetic
1117: //===----------------------------------------------------------------------===//
1118: 
1119: // Load FP integer
1120: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "FIDTR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXTR$")>;
1122: 
1123: // Extract biased exponent
1124: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "EEDTR$")>;
1125: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "EEXTR$")>;
1126: 
1127: // Extract significance
1128: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "ESDTR$")>;
1129: def : InstRW<[WLat12, FXb, VecDF2, Cracked], (instregex "ESXTR$")>;
1130: 
1131: //===----------------------------------------------------------------------===//
1132: // DFP: Binary arithmetic
1133: //===----------------------------------------------------------------------===//
1134: 
1135: // Addition
1136: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "ADTR(A)?$")>;
1137: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXTR(A)?$")>;
1138: 
1139: // Subtraction
1140: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "SDTR(A)?$")>;
1141: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXTR(A)?$")>;
1142: 
1143: // Multiply
1144: def : InstRW<[WLat30, VecDF, NormalGr], (instregex "MDTR(A)?$")>;
1145: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "MXTR(A)?$")>;
1146: 
1147: // Division
1148: def : InstRW<[WLat30, VecDF, NormalGr], (instregex "DDTR(A)?$")>;
1149: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "DXTR(A)?$")>;
1150: 
1151: // Quantize
1152: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "QADTR$")>;
1153: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "QAXTR$")>;
1154: 
1155: // Reround
1156: def : InstRW<[WLat9, WLat9, FXb, VecDF, Cracked], (instregex "RRDTR$")>;
1157: def : InstRW<[WLat11, WLat11, FXb, VecDF4, GroupAlone2], (instregex "RRXTR$")>;
1158: 
1159: // Shift significand left/right
1160: def : InstRW<[WLat11LSU, LSU, VecDF, GroupAlone], (instregex "S(L|R)DT$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161: def : InstRW<[WLat11LSU, LSU, VecDF4, GroupAlone], (instregex "S(L|R)XT$")>;
1162: 
1163: // Insert biased exponent
1164: def : InstRW<[WLat9, FXb, VecDF, Cracked], (instregex "IEDTR$")>;
1165: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "IEXTR$")>;
1166: 
1167: //===----------------------------------------------------------------------===//
1168: // DFP: Comparisons
1169: //===----------------------------------------------------------------------===//
1170: 
1171: // Compare
1172: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "(K|C)DTR$")>;
1173: def : InstRW<[WLat9, VecDF2, GroupAlone], (instregex "(K|C)XTR$")>;
1174: 
1175: // Compare biased exponent
1176: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "CEDTR$")>;
1177: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "CEXTR$")>;
1178: 
1179: // Test Data Class/Group
1180: def : InstRW<[WLat15, LSU, VecDF, NormalGr], (instregex "TD(C|G)(E|D)T$")>;
1181: def : InstRW<[WLat15, LSU, VecDF2, GroupAlone], (instregex "TD(C|G)XT$")>;
1182: 
1183: 
1184: // --------------------------------- Vector --------------------------------- //
1185: 
1186: //===----------------------------------------------------------------------===//
1187: // Vector: Move instructions
1188: //===----------------------------------------------------------------------===//
1189: 
1190: def : InstRW<[WLat1, FXb, NormalGr], (instregex "VLR(16|32|64)?$")>;
1191: def : InstRW<[WLat3, FXb, NormalGr], (instregex "VLGV(B|F|G|H)?$")>;
1192: def : InstRW<[WLat1, FXb, NormalGr], (instregex "VLVG(B|F|G|H)?$")>;
1193: def : InstRW<[WLat3, FXb, NormalGr], (instregex "VLVGP(32)?$")>;
1194: 
1195: //===----------------------------------------------------------------------===//
1196: // Vector: Immediate instructions
1197: //===----------------------------------------------------------------------===//
1198: 
1199: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VZERO$")>;
1200: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VONE$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VGBM$")>;
1202: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VGM(B|F|G|H)?$")>;
1203: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VREPI(B|F|G|H)?$")>;
1204: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLEI(B|F|G|H)$")>;
1205: 
1206: //===----------------------------------------------------------------------===//
1207: // Vector: Loads
1208: //===----------------------------------------------------------------------===//
1209: 
1210: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(Align)?$")>;
1211: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(L|BB)$")>;
1212: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(16|32|64)$")>;
1213: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLLEZ(B|F|G|H|LF)?$")>;
1214: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLREP(B|F|G|H)?$")>;
1215: def : InstRW<[WLat2LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
1216:              (instregex "VLE(B|F|G|H)$")>;
1217: def : InstRW<[WLat5LSU, RegReadAdv, FXb, LSU, VecXsPm, Cracked],
1218:              (instregex "VGE(F|G)$")>;
1219: def : InstRW<[WLat4LSU, WLat4LSU, LSU5, GroupAlone],
1220:              (instregex "VLM(Align)?$")>;
1221: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLRL(R)?$")>;
1222: 
1223: //===----------------------------------------------------------------------===//
1224: // Vector: Stores
1225: //===----------------------------------------------------------------------===//
1226: 
1227: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VST(Align|L|16|32|64)?$")>;
1228: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTE(F|G)$")>;
1229: def : InstRW<[WLat1, FXb, LSU, VecXsPm, Cracked], (instregex "VSTE(B|H)$")>;
1230: def : InstRW<[WLat1, LSU2, FXb3, GroupAlone2], (instregex "VSTM(Align)?$")>;
1231: def : InstRW<[WLat1, FXb2, LSU, Cracked], (instregex "VSCE(F|G)$")>;
1232: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTRL(R)?$")>;
1233: 
1234: //===----------------------------------------------------------------------===//
1235: // Vector: Selects and permutes
1236: //===----------------------------------------------------------------------===//
1237: 
1238: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMRH(B|F|G|H)?$")>;
1239: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMRL(B|F|G|H)?$")>;
1240: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPERM$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPDI$")>;
1242: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VBPERM$")>;
1243: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VREP(B|F|G|H)?$")>;
1244: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSEL$")>;
1245: 
1246: //===----------------------------------------------------------------------===//
1247: // Vector: Widening and narrowing
1248: //===----------------------------------------------------------------------===//
1249: 
1250: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPK(F|G|H)?$")>;
1251: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPKS(F|G|H)?$")>;
1252: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VPKS(F|G|H)S$")>;
1253: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPKLS(F|G|H)?$")>;
1254: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VPKLS(F|G|H)S$")>;
1255: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSEG(B|F|H)?$")>;
1256: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPH(B|F|H)?$")>;
1257: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPL(B|F)?$")>;
1258: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPLH(B|F|H|W)?$")>;
1259: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPLL(B|F|H)?$")>;
1260: 
1261: //===----------------------------------------------------------------------===//
1262: // Vector: Integer arithmetic
1263: //===----------------------------------------------------------------------===//
1264: 
1265: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VA(B|F|G|H|Q|C|CQ)?$")>;
1266: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VACC(B|F|G|H|Q|C|CQ)?$")>;
1267: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VAVG(B|F|G|H)?$")>;
1268: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VAVGL(B|F|G|H)?$")>;
1269: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VN(C|O|N|X)?$")>;
1270: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VO(C)?$")>;
1271: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VCKSM$")>;
1272: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCLZ(B|F|G|H)?$")>;
1273: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCTZ(B|F|G|H)?$")>;
1274: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VX$")>;
1275: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFM?$")>;
1276: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFMA(B|F|G|H)?$")>;
1277: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFM(B|F|G|H)$")>;
1278: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLC(B|F|G|H)?$")>;
1279: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLP(B|F|G|H)?$")>;
1280: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMX(B|F|G|H)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMXL(B|F|G|H)?$")>;
1282: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMN(B|F|G|H)?$")>;
1283: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMNL(B|F|G|H)?$")>;
1284: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAL(B|F)?$")>;
1285: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALE(B|F|H)?$")>;
1286: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALH(B|F|H|W)?$")>;
1287: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALO(B|F|H)?$")>;
1288: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAO(B|F|H)?$")>;
1289: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAE(B|F|H)?$")>;
1290: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAH(B|F|H)?$")>;
1291: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VME(B|F|H)?$")>;
1292: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMH(B|F|H)?$")>;
1293: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VML(B|F)?$")>;
1294: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLE(B|F|H)?$")>;
1295: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLH(B|F|H|W)?$")>;
1296: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLO(B|F|H)?$")>;
1297: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMO(B|F|H)?$")>;
1298: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VMSL(G)?$")>;
1299: 
1300: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPOPCT(B|F|G|H)?$")>;
1301: 
1302: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERLL(B|F|G|H)?$")>;
1303: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERLLV(B|F|G|H)?$")>;
1304: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERIM(B|F|G|H)?$")>;
1305: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESL(B|F|G|H)?$")>;
1306: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESLV(B|F|G|H)?$")>;
1307: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRA(B|F|G|H)?$")>;
1308: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRAV(B|F|G|H)?$")>;
1309: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRL(B|F|G|H)?$")>;
1310: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRLV(B|F|G|H)?$")>;
1311: 
1312: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSL(DB)?$")>;
1313: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSLB$")>;
1314: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSR(A|L)$")>;
1315: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSR(A|L)B$")>;
1316: 
1317: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSB(I|IQ|CBI|CBIQ)?$")>;
1318: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSCBI(B|F|G|H|Q)?$")>;
1319: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VS(F|G|H|Q)?$")>;
1320: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1321-1360 / 第 1321-1360 行
```tablegen
1321: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUM(B|H)?$")>;
1322: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUMG(F|H)?$")>;
1323: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUMQ(F|G)?$")>;
1324: 
1325: //===----------------------------------------------------------------------===//
1326: // Vector: Integer comparison
1327: //===----------------------------------------------------------------------===//
1328: 
1329: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "VEC(B|F|G|H)?$")>;
1330: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "VECL(B|F|G|H)?$")>;
1331: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCEQ(B|F|G|H)?$")>;
1332: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCEQ(B|F|G|H)S$")>;
1333: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCH(B|F|G|H)?$")>;
1334: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCH(B|F|G|H)S$")>;
1335: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCHL(B|F|G|H)?$")>;
1336: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCHL(B|F|G|H)S$")>;
1337: def : InstRW<[WLat4, VecStr, NormalGr], (instregex "VTM$")>;
1338: 
1339: //===----------------------------------------------------------------------===//
1340: // Vector: Floating-point arithmetic
1341: //===----------------------------------------------------------------------===//
1342: 
1343: // Conversion and rounding
1344: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VCD(L)?G$")>;
1345: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VCD(L)?GB$")>;
1346: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WCD(L)?GB$")>;
1347: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VC(L)?GD$")>;
1348: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VC(L)?GDB$")>;
1349: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WC(L)?GDB$")>;
1350: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VL(DE|ED)$")>;
1351: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VL(DE|ED)B$")>;
1352: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WL(DE|ED)B$")>;
1353: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VFL(L|R)$")>;
1354: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VFL(LS|RD)$")>;
1355: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WFL(LS|RD)$")>;
1356: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "WFLLD$")>;
1357: def : InstRW<[WLat10, VecDF2, NormalGr], (instregex "WFLRX$")>;
1358: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFI$")>;
1359: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VFIDB$")>;
1360: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WFIDB$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1361-1400 / 第 1361-1400 行
```tablegen
1361: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFISB$")>;
1362: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WFISB$")>;
1363: def : InstRW<[WLat10, VecDF2, NormalGr], (instregex "WFIXB$")>;
1364: 
1365: // Sign operations
1366: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VFPSO$")>;
1367: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FPSODB$")>;
1368: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FPSOSB$")>;
1369: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFPSOXB$")>;
1370: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FL(C|N|P)DB$")>;
1371: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FL(C|N|P)SB$")>;
1372: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFL(C|N|P)XB$")>;
1373: 
1374: // Minimum / maximum
1375: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(MAX|MIN)$")>;
1376: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(MAX|MIN)DB$")>;
1377: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WF(MAX|MIN)DB$")>;
1378: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(MAX|MIN)SB$")>;
1379: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WF(MAX|MIN)SB$")>;
1380: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "WF(MAX|MIN)XB$")>;
1381: 
1382: // Test data class
1383: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFTCI$")>;
1384: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "(V|W)FTCIDB$")>;
1385: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "(V|W)FTCISB$")>;
1386: def : InstRW<[WLat3, WLat3, VecDFX, NormalGr], (instregex "WFTCIXB$")>;
1387: 
1388: // Add / subtract
1389: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VF(A|S)$")>;
1390: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VF(A|S)DB$")>;
1391: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WF(A|S)DB$")>;
1392: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VF(A|S)SB$")>;
1393: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WF(A|S)SB$")>;
1394: def : InstRW<[WLat10, VecDF2, NormalGr], (instregex "WF(A|S)XB$")>;
1395: 
1396: // Multiply / multiply-and-add/subtract
1397: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFM$")>;
1398: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VFMDB$")>;
1399: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WFM(D|S)B$")>;
1400: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFMSB$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1401-1440 / 第 1401-1440 行
```tablegen
1401: def : InstRW<[WLat20, VecDF2, NormalGr], (instregex "WFMXB$")>;
1402: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VF(N)?M(A|S)$")>;
1403: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "VF(N)?M(A|S)DB$")>;
1404: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WF(N)?M(A|S)DB$")>;
1405: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VF(N)?M(A|S)SB$")>;
1406: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WF(N)?M(A|S)SB$")>;
1407: def : InstRW<[WLat30, VecDF2, NormalGr], (instregex "WF(N)?M(A|S)XB$")>;
1408: 
1409: // Divide / square root
1410: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFD$")>;
1411: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FDDB$")>;
1412: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FDSB$")>;
1413: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "WFDXB$")>;
1414: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFSQ$")>;
1415: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FSQDB$")>;
1416: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FSQSB$")>;
1417: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "WFSQXB$")>;
1418: 
1419: //===----------------------------------------------------------------------===//
1420: // Vector: Floating-point comparison
1421: //===----------------------------------------------------------------------===//
1422: 
1423: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(C|K)(E|H|HE)$")>;
1424: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(C|K)(E|H|HE)DB$")>;
1425: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)DB$")>;
1426: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFK(E|H|HE)DB$")>;
1427: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(C|K)(E|H|HE)SB$")>;
1428: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)SB$")>;
1429: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFK(E|H|HE)SB$")>;
1430: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "WFC(E|H|HE)XB$")>;
1431: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "WFK(E|H|HE)XB$")>;
1432: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFC(E|H|HE)DBS$")>;
1433: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFK(E|H|HE)DBS$")>;
1434: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr],
1435:              (instregex "WF(C|K)(E|H|HE)DBS$")>;
1436: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr],
1437:              (instregex "VF(C|K)(E|H|HE)SBS$")>;
1438: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)SBS$")>;
1439: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "WFK(E|H|HE)SBS$")>;
1440: def : InstRW<[WLat3, WLat3, VecDFX, NormalGr], (instregex "WFC(E|H|HE)XBS$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1441-1480 / 第 1441-1480 行
```tablegen
1441: def : InstRW<[WLat3, WLat3, VecDFX, NormalGr], (instregex "WFK(E|H|HE)XBS$")>;
1442: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)$")>;
1443: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)DB$")>;
1444: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)SB$")>;
1445: def : InstRW<[WLat3, VecDFX, NormalGr], (instregex "WF(C|K)XB$")>;
1446: 
1447: //===----------------------------------------------------------------------===//
1448: // Vector: Floating-point insertion and extraction
1449: //===----------------------------------------------------------------------===//
1450: 
1451: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LEFR(_16)?$")>;
1452: def : InstRW<[WLat3, FXb, NormalGr], (instregex "LFER(_16)?$")>;
1453: 
1454: //===----------------------------------------------------------------------===//
1455: // Vector: String instructions
1456: //===----------------------------------------------------------------------===//
1457: 
1458: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAE(B)?$")>;
1459: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAE(F|H)$")>;
1460: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VFAE(B|F|H)S$")>;
1461: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAEZ(B|F|H)$")>;
1462: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VFAEZ(B|F|H)S$")>;
1463: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFEE(B|F|H|ZB|ZF|ZH)?$")>;
1464: def : InstRW<[WLat4, WLat4, VecStr, NormalGr],
1465:              (instregex "VFEE(B|F|H|ZB|ZF|ZH)S$")>;
1466: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFENE(B|F|H|ZB|ZF|ZH)?$")>;
1467: def : InstRW<[WLat4, WLat4, VecStr, NormalGr],
1468:              (instregex "VFENE(B|F|H|ZB|ZF|ZH)S$")>;
1469: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VISTR(B|F|H)?$")>;
1470: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VISTR(B|F|H)S$")>;
1471: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VSTRC(B|F|H)?$")>;
1472: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRC(B|F|H)S$")>;
1473: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VSTRCZ(B|F|H)$")>;
1474: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRCZ(B|F|H)S$")>;
1475: 
1476: //===----------------------------------------------------------------------===//
1477: // Vector: Packed-decimal instructions
1478: //===----------------------------------------------------------------------===//
1479: 
1480: def : InstRW<[WLat10, VecDF2, NormalGr], (instregex "VLIP$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1481-1520 / 第 1481-1520 行
```tablegen
1481: def : InstRW<[WLat6, VecDFX, LSU, GroupAlone2], (instregex "VPKZ$")>;
1482: def : InstRW<[WLat1, VecDFX, FXb, LSU, Cracked], (instregex "VUPKZ$")>;
1483: def : InstRW<[WLat20, WLat20, VecDF2, FXb, GroupAlone], (instregex "VCVB(G)?$")>;
1484: def : InstRW<[WLat20, WLat20, VecDF2, FXb, GroupAlone], (instregex "VCVD(G)?$")>;
1485: def : InstRW<[WLat4, WLat4, VecDFX, NormalGr], (instregex "V(A|S)P$")>;
1486: def : InstRW<[WLat30, WLat30, VecDF2, GroupAlone], (instregex "VM(S)?P$")>;
1487: def : InstRW<[WLat30, WLat30, VecDF2, GroupAlone], (instregex "V(D|R)P$")>;
1488: def : InstRW<[WLat30, WLat30, MCD], (instregex "VSDP$")>;
1489: def : InstRW<[WLat10, WLat10, VecDF2, NormalGr], (instregex "VSRP$")>;
1490: def : InstRW<[WLat4, WLat4, VecDFX, NormalGr], (instregex "VPSOP$")>;
1491: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "V(T|C)P$")>;
1492: 
1493: 
1494: // -------------------------------- System ---------------------------------- //
1495: 
1496: //===----------------------------------------------------------------------===//
1497: // System: Program-Status Word Instructions
1498: //===----------------------------------------------------------------------===//
1499: 
1500: def : InstRW<[WLat30, WLat30, MCD], (instregex "EPSW$")>;
1501: def : InstRW<[WLat20, GroupAlone3], (instregex "LPSW(E)?$")>;
1502: def : InstRW<[WLat3, FXa, GroupAlone], (instregex "IPK$")>;
1503: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SPKA$")>;
1504: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SSM$")>;
1505: def : InstRW<[WLat1, FXb, LSU, GroupAlone], (instregex "ST(N|O)SM$")>;
1506: def : InstRW<[WLat3, FXa, NormalGr], (instregex "IAC$")>;
1507: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SAC(F)?$")>;
1508: 
1509: //===----------------------------------------------------------------------===//
1510: // System: Control Register Instructions
1511: //===----------------------------------------------------------------------===//
1512: 
1513: def : InstRW<[WLat4LSU, WLat4LSU, LSU2, GroupAlone], (instregex "LCTL(G)?$")>;
1514: def : InstRW<[WLat1, LSU5, FXb, GroupAlone2], (instregex "STCT(L|G)$")>;
1515: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "E(P|S)A(I)?R$")>;
1516: def : InstRW<[WLat30, MCD], (instregex "SSA(I)?R$")>;
1517: def : InstRW<[WLat30, MCD], (instregex "ESEA$")>;
1518: 
1519: //===----------------------------------------------------------------------===//
1520: // System: Prefix-Register Instructions
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1521-1560 / 第 1521-1560 行
```tablegen
1521: //===----------------------------------------------------------------------===//
1522: 
1523: def : InstRW<[WLat30, MCD], (instregex "S(T)?PX$")>;
1524: 
1525: //===----------------------------------------------------------------------===//
1526: // System: Storage-Key and Real Memory Instructions
1527: //===----------------------------------------------------------------------===//
1528: 
1529: def : InstRW<[WLat30, MCD], (instregex "ISKE$")>;
1530: def : InstRW<[WLat30, MCD], (instregex "IVSK$")>;
1531: def : InstRW<[WLat30, MCD], (instregex "SSKE(Opt)?$")>;
1532: def : InstRW<[WLat30, MCD], (instregex "RRB(E|M)$")>;
1533: def : InstRW<[WLat30, MCD], (instregex "IRBM$")>;
1534: def : InstRW<[WLat30, MCD], (instregex "PFMF$")>;
1535: def : InstRW<[WLat30, WLat30, MCD], (instregex "TB$")>;
1536: def : InstRW<[WLat30, MCD], (instregex "PGIN$")>;
1537: def : InstRW<[WLat30, MCD], (instregex "PGOUT$")>;
1538: 
1539: //===----------------------------------------------------------------------===//
1540: // System: Dynamic-Address-Translation Instructions
1541: //===----------------------------------------------------------------------===//
1542: 
1543: def : InstRW<[WLat30, MCD], (instregex "IPTE(Opt)?(Opt)?$")>;
1544: def : InstRW<[WLat30, MCD], (instregex "IDTE(Opt)?$")>;
1545: def : InstRW<[WLat30, MCD], (instregex "CRDTE(Opt)?$")>;
1546: def : InstRW<[WLat30, MCD], (instregex "PTLB$")>;
1547: def : InstRW<[WLat30, WLat30, MCD], (instregex "CSP(G)?$")>;
1548: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "LPTEA$")>;
1549: def : InstRW<[WLat30, WLat30, MCD], (instregex "LRA(Y|G)?$")>;
1550: def : InstRW<[WLat30, MCD], (instregex "STRAG$")>;
1551: def : InstRW<[WLat30, MCD], (instregex "LURA(G)?$")>;
1552: def : InstRW<[WLat30, MCD], (instregex "STUR(A|G)$")>;
1553: def : InstRW<[WLat30, MCD], (instregex "TPROT$")>;
1554: 
1555: //===----------------------------------------------------------------------===//
1556: // System: Memory-move Instructions
1557: //===----------------------------------------------------------------------===//
1558: 
1559: def : InstRW<[WLat4LSU, FXa2, FXb, LSU5, GroupAlone2], (instregex "MVC(K|P|S)$")>;
1560: def : InstRW<[WLat1, FXa, LSU5, GroupAlone2], (instregex "MVC(S|D)K$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1561-1600 / 第 1561-1600 行
```tablegen
1561: def : InstRW<[WLat30, MCD], (instregex "MVCOS$")>;
1562: def : InstRW<[WLat30, MCD], (instregex "MVPG$")>;
1563: 
1564: //===----------------------------------------------------------------------===//
1565: // System: Address-Space Instructions
1566: //===----------------------------------------------------------------------===//
1567: 
1568: def : InstRW<[WLat30, MCD], (instregex "LASP$")>;
1569: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "PALB$")>;
1570: def : InstRW<[WLat30, MCD], (instregex "PC$")>;
1571: def : InstRW<[WLat30, MCD], (instregex "PR$")>;
1572: def : InstRW<[WLat30, MCD], (instregex "PT(I)?$")>;
1573: def : InstRW<[WLat30, MCD], (instregex "RP$")>;
1574: def : InstRW<[WLat30, MCD], (instregex "BS(G|A)$")>;
1575: def : InstRW<[WLat30, MCD], (instregex "TAR$")>;
1576: 
1577: //===----------------------------------------------------------------------===//
1578: // System: Linkage-Stack Instructions
1579: //===----------------------------------------------------------------------===//
1580: 
1581: def : InstRW<[WLat30, MCD], (instregex "BAKR$")>;
1582: def : InstRW<[WLat30, MCD], (instregex "EREG(G)?$")>;
1583: def : InstRW<[WLat30, WLat30, MCD], (instregex "(E|M)STA$")>;
1584: 
1585: //===----------------------------------------------------------------------===//
1586: // System: Time-Related Instructions
1587: //===----------------------------------------------------------------------===//
1588: 
1589: def : InstRW<[WLat30, MCD], (instregex "PTFF$")>;
1590: def : InstRW<[WLat30, MCD], (instregex "SCK(PF|C)?$")>;
1591: def : InstRW<[WLat1, LSU2, GroupAlone], (instregex "SPT$")>;
1592: def : InstRW<[WLat15, LSU3, FXa2, FXb, GroupAlone2], (instregex "STCK(F)?$")>;
1593: def : InstRW<[WLat20, LSU4, FXa2, FXb2, GroupAlone3], (instregex "STCKE$")>;
1594: def : InstRW<[WLat30, MCD], (instregex "STCKC$")>;
1595: def : InstRW<[WLat1, LSU2, FXb, Cracked], (instregex "STPT$")>;
1596: 
1597: //===----------------------------------------------------------------------===//
1598: // System: CPU-Related Instructions
1599: //===----------------------------------------------------------------------===//
1600: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1601-1640 / 第 1601-1640 行
```tablegen
1601: def : InstRW<[WLat30, MCD], (instregex "STAP$")>;
1602: def : InstRW<[WLat30, MCD], (instregex "STIDP$")>;
1603: def : InstRW<[WLat30, WLat30, MCD], (instregex "STSI$")>;
1604: def : InstRW<[WLat30, WLat30, MCD], (instregex "STFL(E)?$")>;
1605: def : InstRW<[WLat30, MCD], (instregex "ECAG$")>;
1606: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECTG$")>;
1607: def : InstRW<[WLat30, MCD], (instregex "PTF$")>;
1608: def : InstRW<[WLat30, MCD], (instregex "PCKMO$")>;
1609: 
1610: //===----------------------------------------------------------------------===//
1611: // System: Miscellaneous Instructions
1612: //===----------------------------------------------------------------------===//
1613: 
1614: def : InstRW<[WLat30, MCD], (instregex "SVC$")>;
1615: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "MC$")>;
1616: def : InstRW<[WLat30, MCD], (instregex "DIAG$")>;
1617: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TRAC(E|G)$")>;
1618: def : InstRW<[WLat30, MCD], (instregex "TRAP(2|4)$")>;
1619: def : InstRW<[WLat30, MCD], (instregex "SIG(P|A)$")>;
1620: def : InstRW<[WLat30, MCD], (instregex "SIE$")>;
1621: 
1622: //===----------------------------------------------------------------------===//
1623: // System: CPU-Measurement Facility Instructions
1624: //===----------------------------------------------------------------------===//
1625: 
1626: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LPP$")>;
1627: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECPGA$")>;
1628: def : InstRW<[WLat30, WLat30, MCD], (instregex "E(C|P)CTR$")>;
1629: def : InstRW<[WLat30, MCD], (instregex "LCCTL$")>;
1630: def : InstRW<[WLat30, MCD], (instregex "L(P|S)CTL$")>;
1631: def : InstRW<[WLat30, MCD], (instregex "Q(S|CTR)I$")>;
1632: def : InstRW<[WLat30, MCD], (instregex "S(C|P)CTR$")>;
1633: 
1634: //===----------------------------------------------------------------------===//
1635: // System: I/O Instructions
1636: //===----------------------------------------------------------------------===//
1637: 
1638: def : InstRW<[WLat30, MCD], (instregex "(C|H|R|X)SCH$")>;
1639: def : InstRW<[WLat30, MCD], (instregex "(M|S|ST|T)SCH$")>;
1640: def : InstRW<[WLat30, MCD], (instregex "RCHP$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1641-1653 / 第 1641-1653 行
```tablegen
1641: def : InstRW<[WLat30, MCD], (instregex "SCHM$")>;
1642: def : InstRW<[WLat30, MCD], (instregex "STC(PS|RW)$")>;
1643: def : InstRW<[WLat30, MCD], (instregex "TPE?I$")>;
1644: def : InstRW<[WLat30, MCD], (instregex "SAL$")>;
1645: 
1646: //===----------------------------------------------------------------------===//
1647: // NOPs
1648: //===----------------------------------------------------------------------===//
1649: 
1650: def : InstRW<[WLat1, FXb, NormalGr], (instregex "NOP(R)?(Opt)?$")>;
1651: def : InstRW<[WLat1, VBU, NormalGr], (instregex "J(G)?NOP$")>;
1652: }
1653: 
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
