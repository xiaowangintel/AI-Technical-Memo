# SystemZScheduleZ15.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZScheduleZ15.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //-- SystemZScheduleZ15.td - SystemZ Scheduling Definitions ----*- tblgen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the machine model for Z15 to support instruction
  10: // scheduling and other instruction cost heuristics.
  11: //
  12: // Pseudos expanded right after isel do not need to be modelled here.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: def Z15Model : SchedMachineModel {
  17: 
  18:     let UnsupportedFeatures = Arch13UnsupportedFeatures.List;
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
  30: let SchedModel = Z15Model in  {
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
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `Z15Model`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `Z15Model` 等 TableGen 记录。

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
  76: def Z15_FXaUnit     : ProcResource<2>;
  77: def Z15_FXbUnit     : ProcResource<2>;
  78: def Z15_LSUnit      : ProcResource<2>;
  79: def Z15_VecUnit     : ProcResource<2>;
  80: def Z15_VecFPdUnit  : ProcResource<2> { let BufferSize = 1; /* blocking */ }
```
- **EN**: This block declares or refines TableGen records such as `Z15_FXaUnit`, `Z15_FXbUnit`, `Z15_LSUnit`, `Z15_VecUnit`, `Z15_VecFPdUnit`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `Z15_FXaUnit`, `Z15_FXbUnit`, `Z15_LSUnit`, `Z15_VecUnit`, `Z15_VecFPdUnit` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 81-120 / 第 81-120 行
```tablegen
  81: def Z15_VBUnit      : ProcResource<2>;
  82: def Z15_MCD         : ProcResource<1>;
  83: 
  84: // Subtarget specific definitions of scheduling resources.
  85: let NumMicroOps = 0 in {
  86:   def : WriteRes<FXa, [Z15_FXaUnit]>;
  87:   def : WriteRes<FXb, [Z15_FXbUnit]>;
  88:   def : WriteRes<LSU, [Z15_LSUnit]>;
  89:   def : WriteRes<VecBF,  [Z15_VecUnit]>;
  90:   def : WriteRes<VecDF,  [Z15_VecUnit]>;
  91:   def : WriteRes<VecDFX, [Z15_VecUnit]>;
  92:   def : WriteRes<VecMul,  [Z15_VecUnit]>;
  93:   def : WriteRes<VecStr,  [Z15_VecUnit]>;
  94:   def : WriteRes<VecXsPm, [Z15_VecUnit]>;
  95:   foreach Num = 2-5 in { let ReleaseAtCycles = [Num] in {
  96:     def : WriteRes<!cast<SchedWrite>("FXa"#Num), [Z15_FXaUnit]>;
  97:     def : WriteRes<!cast<SchedWrite>("FXb"#Num), [Z15_FXbUnit]>;
  98:     def : WriteRes<!cast<SchedWrite>("LSU"#Num), [Z15_LSUnit]>;
  99:     def : WriteRes<!cast<SchedWrite>("VecBF"#Num), [Z15_VecUnit]>;
 100:     def : WriteRes<!cast<SchedWrite>("VecDF"#Num), [Z15_VecUnit]>;
 101:     def : WriteRes<!cast<SchedWrite>("VecDFX"#Num), [Z15_VecUnit]>;
 102:     def : WriteRes<!cast<SchedWrite>("VecMul"#Num), [Z15_VecUnit]>;
 103:     def : WriteRes<!cast<SchedWrite>("VecStr"#Num), [Z15_VecUnit]>;
 104:     def : WriteRes<!cast<SchedWrite>("VecXsPm"#Num), [Z15_VecUnit]>;
 105:   }}
 106: 
 107:   def : WriteRes<VecFPd,  [Z15_VecFPdUnit]> { let ReleaseAtCycles = [30]; }
 108: 
 109:   def : WriteRes<VBU,     [Z15_VBUnit]>; // Virtual Branching Unit
 110: }
 111: 
 112: def : WriteRes<MCD, [Z15_MCD]> { let NumMicroOps = 3;
 113:                                     let BeginGroup  = 1;
 114:                                     let EndGroup    = 1; }
 115: 
 116: // -------------------------- INSTRUCTIONS ---------------------------------- //
 117: 
 118: // InstRW constructs have been used in order to preserve the
 119: // readability of the InstrInfo files.
 120: 
```
- **EN**: This block declares or refines TableGen records such as `Z15_VBUnit`, `Z15_MCD`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `Z15_VBUnit`, `Z15_MCD` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

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
 190: def : InstRW<[WLat1, LSU2, GroupAlone], (instregex "MVCRL$")>;
 191: 
 192: // Pseudo -> reg move
 193: def : InstRW<[WLat1, FXa, NormalGr], (instregex "COPY(_TO_REGCLASS)?$")>;
 194: def : InstRW<[WLat1, FXa, NormalGr], (instregex "EXTRACT_SUBREG$")>;
 195: def : InstRW<[WLat1, FXa, NormalGr], (instregex "INSERT_SUBREG$")>;
 196: def : InstRW<[WLat1, FXa, NormalGr], (instregex "REG_SEQUENCE$")>;
 197: 
 198: // Loads
 199: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L(Y|FH|RL|Mux)?$")>;
 200: def : InstRW<[LSULatency, LSULatency, LSU, NormalGr], (instregex "LCBB$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 201-240 / 第 201-240 行
```tablegen
 201: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LG(RL)?$")>;
 202: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L128$")>;
 203: 
 204: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLIH(F|H|L)$")>;
 205: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLIL(F|H|L)$")>;
 206: 
 207: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LG(F|H)I$")>;
 208: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LHI(Mux)?$")>;
 209: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LR$")>;
 210: 
 211: // Load and zero rightmost byte
 212: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LZR(F|G)$")>;
 213: 
 214: // Load and trap
 215: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "L(FH|G)?AT$")>;
 216: 
 217: // Load and test
 218: def : InstRW<[WLat1LSU, WLat1LSU, LSU, FXa, NormalGr], (instregex "LT(G)?$")>;
 219: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LT(G)?R$")>;
 220: 
 221: // Stores
 222: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STG(RL)?$")>;
 223: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST128$")>;
 224: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST(Y|FH|RL|Mux)?$")>;
 225: 
 226: // String moves.
 227: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "MVST$")>;
 228: 
 229: //===----------------------------------------------------------------------===//
 230: // Conditional move instructions
 231: //===----------------------------------------------------------------------===//
 232: 
 233: def : InstRW<[WLat2, FXa, NormalGr], (instregex "LOCRMux$")>;
 234: def : InstRW<[WLat2, FXa, NormalGr], (instregex "LOC(G|FH)?R(Asm.*)?$")>;
 235: def : InstRW<[WLat2, FXa, NormalGr], (instregex "LOC(G|H)?HI(Mux|(Asm.*))?$")>;
 236: def : InstRW<[WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 237:              (instregex "LOC(G|FH|Mux)?(Asm.*)?$")>;
 238: def : InstRW<[WLat1, FXb, LSU, NormalGr],
 239:              (instregex "STOC(G|FH|Mux)?(Asm.*)?$")>;
 240: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 241-280 / 第 241-280 行
```tablegen
 241: def : InstRW<[WLat2, FXa, NormalGr], (instregex "SELRMux$")>;
 242: def : InstRW<[WLat2, FXa, NormalGr], (instregex "SEL(G|FH)?R(Asm.*)?$")>;
 243: 
 244: //===----------------------------------------------------------------------===//
 245: // Sign extensions
 246: //===----------------------------------------------------------------------===//
 247: 
 248: def : InstRW<[WLat1, FXa, NormalGr], (instregex "L(B|H|G)R$")>;
 249: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LG(B|H|F)R$")>;
 250: 
 251: def : InstRW<[WLat1LSU, WLat1LSU, FXa, LSU, NormalGr], (instregex "LTGF$")>;
 252: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LTGFR$")>;
 253: 
 254: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LB(H|Mux)?$")>;
 255: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LH(Y)?$")>;
 256: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LH(H|Mux|RL)$")>;
 257: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LG(B|H|F)$")>;
 258: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LG(H|F)RL$")>;
 259: 
 260: //===----------------------------------------------------------------------===//
 261: // Zero extensions
 262: //===----------------------------------------------------------------------===//
 263: 
 264: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLCR(Mux)?$")>;
 265: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLHR(Mux)?$")>;
 266: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLG(C|H|F|T)R$")>;
 267: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLC(Mux)?$")>;
 268: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLH(Mux)?$")>;
 269: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LL(C|H)H$")>;
 270: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLHRL$")>;
 271: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLG(C|H|F|T|HRL|FRL)$")>;
 272: 
 273: // Load and zero rightmost byte
 274: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLZRGF$")>;
 275: 
 276: // Load and trap
 277: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "LLG(F|T)?AT$")>;
 278: 
 279: //===----------------------------------------------------------------------===//
 280: // Truncations
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 281-320 / 第 281-320 行
```tablegen
 281: //===----------------------------------------------------------------------===//
 282: 
 283: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STC(H|Y|Mux)?$")>;
 284: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STH(H|Y|RL|Mux)?$")>;
 285: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STCM(H|Y)?$")>;
 286: 
 287: //===----------------------------------------------------------------------===//
 288: // Multi-register moves
 289: //===----------------------------------------------------------------------===//
 290: 
 291: // Load multiple (estimated average of 5 ops)
 292: def : InstRW<[WLat10, WLat10, LSU5, GroupAlone], (instregex "LM(H|Y|G)?$")>;
 293: 
 294: // Load multiple disjoint
 295: def : InstRW<[WLat30, WLat30, MCD], (instregex "LMD$")>;
 296: 
 297: // Store multiple
 298: def : InstRW<[WLat1, LSU2, FXb3, GroupAlone], (instregex "STM(G|H|Y)?$")>;
 299: 
 300: //===----------------------------------------------------------------------===//
 301: // Byte swaps
 302: //===----------------------------------------------------------------------===//
 303: 
 304: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LRV(G)?R$")>;
 305: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LRV(G|H)?$")>;
 306: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STRV(G|H)?$")>;
 307: def : InstRW<[WLat30, MCD], (instregex "MVCIN$")>;
 308: 
 309: //===----------------------------------------------------------------------===//
 310: // Load address instructions
 311: //===----------------------------------------------------------------------===//
 312: 
 313: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LA(Y|RL)?$")>;
 314: 
 315: // Load the Global Offset Table address ( -> larl )
 316: def : InstRW<[WLat1, FXa, NormalGr], (instregex "GOT$")>;
 317: 
 318: //===----------------------------------------------------------------------===//
 319: // Absolute and Negation
 320: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 321-360 / 第 321-360 行
```tablegen
 321: 
 322: def : InstRW<[WLat1, WLat1, FXa, NormalGr], (instregex "LP(G)?R$")>;
 323: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "L(N|P)GFR$")>;
 324: def : InstRW<[WLat1, WLat1, FXa, NormalGr], (instregex "LN(R|GR)$")>;
 325: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LC(R|GR)$")>;
 326: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "LCGFR$")>;
 327: 
 328: //===----------------------------------------------------------------------===//
 329: // Insertion
 330: //===----------------------------------------------------------------------===//
 331: 
 332: def : InstRW<[WLat1LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "IC(Y)?$")>;
 333: def : InstRW<[WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 334:              (instregex "IC32(Y)?$")>;
 335: def : InstRW<[WLat1LSU, RegReadAdv, WLat1LSU, FXa, LSU, NormalGr],
 336:              (instregex "ICM(H|Y)?$")>;
 337: def : InstRW<[WLat1, FXa, NormalGr], (instregex "II(F|H|L)Mux$")>;
 338: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHF(64)?$")>;
 339: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHH(64)?$")>;
 340: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHL(64)?$")>;
 341: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILF(64)?$")>;
 342: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILH(64)?$")>;
 343: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILL(64)?$")>;
 344: 
 345: //===----------------------------------------------------------------------===//
 346: // Addition
 347: //===----------------------------------------------------------------------===//
 348: 
 349: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 350:              (instregex "A(Y)?$")>;
 351: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 352:              (instregex "AH(Y)?$")>;
 353: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AIH$")>;
 354: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AFI(Mux)?$")>;
 355: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 356:              (instregex "AG$")>;
 357: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGFI$")>;
 358: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGHI(K)?$")>;
 359: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGR(K)?$")>;
 360: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AHI(K)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 361-400 / 第 361-400 行
```tablegen
 361: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AHIMux(K)?$")>;
 362: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 363:              (instregex "AL(Y)?$")>;
 364: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AL(FI|HSIK)$")>;
 365: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 366:              (instregex "ALG(F)?$")>;
 367: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGHSIK$")>;
 368: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGF(I|R)$")>;
 369: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGR(K)?$")>;
 370: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALR(K)?$")>;
 371: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AR(K)?$")>;
 372: def : InstRW<[WLat1, FXa, NormalGr], (instregex "A(L)?HHHR$")>;
 373: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "A(L)?HHLR$")>;
 374: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALSIH(N)?$")>;
 375: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "A(L)?(G)?SI$")>;
 376: 
 377: // Logical addition with carry
 378: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, GroupAlone],
 379:              (instregex "ALC(G)?$")>;
 380: def : InstRW<[WLat2, WLat2, FXa, GroupAlone], (instregex "ALC(G)?R$")>;
 381: 
 382: // Add with sign extension (16/32 -> 64)
 383: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 384:              (instregex "AG(F|H)$")>;
 385: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "AGFR$")>;
 386: 
 387: //===----------------------------------------------------------------------===//
 388: // Subtraction
 389: //===----------------------------------------------------------------------===//
 390: 
 391: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 392:              (instregex "S(G|Y)?$")>;
 393: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 394:              (instregex "SH(Y)?$")>;
 395: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SGR(K)?$")>;
 396: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLFI$")>;
 397: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 398:              (instregex "SL(G|GF|Y)?$")>;
 399: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLGF(I|R)$")>;
 400: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLGR(K)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 401-440 / 第 401-440 行
```tablegen
 401: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLR(K)?$")>;
 402: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SR(K)?$")>;
 403: def : InstRW<[WLat1, FXa, NormalGr], (instregex "S(L)?HHHR$")>;
 404: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "S(L)?HHLR$")>;
 405: 
 406: // Subtraction with borrow
 407: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, GroupAlone],
 408:              (instregex "SLB(G)?$")>;
 409: def : InstRW<[WLat2, WLat2, FXa, GroupAlone], (instregex "SLB(G)?R$")>;
 410: 
 411: // Subtraction with sign extension (16/32 -> 64)
 412: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 413:              (instregex "SG(F|H)$")>;
 414: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "SGFR$")>;
 415: 
 416: //===----------------------------------------------------------------------===//
 417: // AND
 418: //===----------------------------------------------------------------------===//
 419: 
 420: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 421:              (instregex "N(G|Y)?$")>;
 422: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NGR(K)?$")>;
 423: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NI(FMux|HMux|LMux)$")>;
 424: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "NI(Y)?$")>;
 425: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHF(64)?$")>;
 426: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHH(64)?$")>;
 427: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHL(64)?$")>;
 428: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILF(64)?$")>;
 429: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILH(64)?$")>;
 430: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILL(64)?$")>;
 431: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NR(K)?$")>;
 432: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "NC$")>;
 433: 
 434: //===----------------------------------------------------------------------===//
 435: // OR
 436: //===----------------------------------------------------------------------===//
 437: 
 438: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 439:              (instregex "O(G|Y)?$")>;
 440: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OGR(K)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 441-480 / 第 441-480 行
```tablegen
 441: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "OI(Y)?$")>;
 442: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OI(FMux|HMux|LMux)$")>;
 443: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHF(64)?$")>;
 444: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHH(64)?$")>;
 445: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHL(64)?$")>;
 446: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILF(64)?$")>;
 447: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILH(64)?$")>;
 448: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILL(64)?$")>;
 449: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OR(K)?$")>;
 450: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "OC$")>;
 451: 
 452: //===----------------------------------------------------------------------===//
 453: // XOR
 454: //===----------------------------------------------------------------------===//
 455: 
 456: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 457:              (instregex "X(G|Y)?$")>;
 458: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "XI(Y)?$")>;
 459: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XIFMux$")>;
 460: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XGR(K)?$")>;
 461: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XIHF(64)?$")>;
 462: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XILF(64)?$")>;
 463: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XR(K)?$")>;
 464: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "XC$")>;
 465: 
 466: //===----------------------------------------------------------------------===//
 467: // Combined logical operations
 468: //===----------------------------------------------------------------------===//
 469: 
 470: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NC(G)?RK$")>;
 471: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OC(G)?RK$")>;
 472: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NN(G)?RK$")>;
 473: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NO(G)?RK$")>;
 474: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NOT(G)?R$")>;
 475: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NX(G)?RK$")>;
 476: 
 477: //===----------------------------------------------------------------------===//
 478: // Multiplication
 479: //===----------------------------------------------------------------------===//
 480: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: def : InstRW<[WLat5LSU, RegReadAdv, FXa, LSU, NormalGr],
 482:              (instregex "MS(GF|Y)?$")>;
 483: def : InstRW<[WLat5, FXa, NormalGr], (instregex "MS(R|FI)$")>;
 484: def : InstRW<[WLat7LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "MSG$")>;
 485: def : InstRW<[WLat7, FXa, NormalGr], (instregex "MSGR$")>;
 486: def : InstRW<[WLat5, FXa, NormalGr], (instregex "MSGF(I|R)$")>;
 487: def : InstRW<[WLat8LSU, RegReadAdv, FXa2, LSU, GroupAlone], (instregex "MLG$")>;
 488: def : InstRW<[WLat8, FXa2, GroupAlone], (instregex "MLGR$")>;
 489: def : InstRW<[WLat4, FXa, NormalGr], (instregex "MGHI$")>;
 490: def : InstRW<[WLat4, FXa, NormalGr], (instregex "MHI$")>;
 491: def : InstRW<[WLat4LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "MH(Y)?$")>;
 492: def : InstRW<[WLat6, FXa2, GroupAlone], (instregex "M(L)?R$")>;
 493: def : InstRW<[WLat6LSU, RegReadAdv, FXa2, LSU, GroupAlone],
 494:              (instregex "M(FY|L)?$")>;
 495: def : InstRW<[WLat8, RegReadAdv, FXa, LSU, NormalGr], (instregex "MGH$")>;
 496: def : InstRW<[WLat12, RegReadAdv, FXa2, LSU, GroupAlone], (instregex "MG$")>;
 497: def : InstRW<[WLat8, FXa2, GroupAlone], (instregex "MGRK$")>;
 498: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, FXa, LSU, NormalGr],
 499:              (instregex "MSC$")>;
 500: def : InstRW<[WLat8LSU, WLat8LSU, RegReadAdv, FXa, LSU, NormalGr],
 501:              (instregex "MSGC$")>;
 502: def : InstRW<[WLat6, WLat6, FXa, NormalGr], (instregex "MSRKC$")>;
 503: def : InstRW<[WLat8, WLat8, FXa, NormalGr], (instregex "MSGRKC$")>;
 504: 
 505: //===----------------------------------------------------------------------===//
 506: // Division and remainder
 507: //===----------------------------------------------------------------------===//
 508: 
 509: def : InstRW<[WLat20, FXa4, GroupAlone], (instregex "DR$")>;
 510: def : InstRW<[WLat30, RegReadAdv, FXa4, LSU, GroupAlone2], (instregex "D$")>;
 511: def : InstRW<[WLat30, FXa2, GroupAlone], (instregex "DSG(F)?R$")>;
 512: def : InstRW<[WLat30, RegReadAdv, FXa2, LSU, GroupAlone2],
 513:              (instregex "DSG(F)?$")>;
 514: def : InstRW<[WLat20, FXa4, GroupAlone], (instregex "DLR$")>;
 515: def : InstRW<[WLat30, FXa4, GroupAlone], (instregex "DLGR$")>;
 516: def : InstRW<[WLat30, RegReadAdv, FXa4, LSU, GroupAlone2],
 517:              (instregex "DL(G)?$")>;
 518: 
 519: //===----------------------------------------------------------------------===//
 520: // Shifts
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 521-560 / 第 521-560 行
```tablegen
 521: //===----------------------------------------------------------------------===//
 522: 
 523: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLL(G|K)?$")>;
 524: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SRL(G|K)?$")>;
 525: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SRA(G|K)?$")>;
 526: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLA(G|K)?$")>;
 527: def : InstRW<[WLat5LSU, WLat5LSU, FXa4, LSU, GroupAlone2],
 528:              (instregex "S(L|R)D(A|L)$")>;
 529: 
 530: // Rotate
 531: def : InstRW<[WLat2LSU, FXa, LSU, NormalGr], (instregex "RLL(G)?$")>;
 532: 
 533: // Rotate and insert
 534: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBH(G|H|L)(Opt)?$")>;
 535: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBL(G|H|L)(Opt)?$")>;
 536: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBG(N|32)?(Z)?(Opt)?$")>;
 537: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBMux$")>;
 538: 
 539: // Rotate and Select
 540: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "R(N|O|X)SBG(Opt)?$")>;
 541: 
 542: //===----------------------------------------------------------------------===//
 543: // Comparison
 544: //===----------------------------------------------------------------------===//
 545: 
 546: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr],
 547:              (instregex "C(G|Y|Mux)?$")>;
 548: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CRL$")>;
 549: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(F|H)I(Mux)?$")>;
 550: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CG(F|H)I$")>;
 551: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CG(HSI|RL)$")>;
 552: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(G)?R$")>;
 553: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CIH$")>;
 554: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CHF$")>;
 555: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CHSI$")>;
 556: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr],
 557:              (instregex "CL(Y|Mux)?$")>;
 558: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLFHSI$")>;
 559: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLFI(Mux)?$")>;
 560: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLG$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLG(HRL|HSI)$")>;
 562: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLGF$")>;
 563: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLGFRL$")>;
 564: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLGF(I|R)$")>;
 565: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLGR$")>;
 566: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLGRL$")>;
 567: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLHF$")>;
 568: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLH(RL|HSI)$")>;
 569: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLIH$")>;
 570: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLI(Y)?$")>;
 571: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLR$")>;
 572: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLRL$")>;
 573: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(L)?HHR$")>;
 574: def : InstRW<[WLat2, FXb, NormalGr], (instregex "C(L)?HLR$")>;
 575: 
 576: // Compare halfword
 577: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CH(Y)?$")>;
 578: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CHRL$")>;
 579: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CGH$")>;
 580: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CGHRL$")>;
 581: def : InstRW<[WLat2LSU, FXa, FXb, LSU, Cracked], (instregex "CHHSI$")>;
 582: 
 583: // Compare with sign extension (32 -> 64)
 584: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CGF$")>;
 585: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CGFRL$")>;
 586: def : InstRW<[WLat2, FXb, NormalGr], (instregex "CGFR$")>;
 587: 
 588: // Compare logical character
 589: def : InstRW<[WLat6, FXb, LSU2, Cracked], (instregex "CLC$")>;
 590: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLCL(E|U)?$")>;
 591: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLST$")>;
 592: 
 593: // Test under mask
 594: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "TM(Y)?$")>;
 595: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TM(H|L)Mux$")>;
 596: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMHH(64)?$")>;
 597: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMHL(64)?$")>;
 598: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMLH(64)?$")>;
 599: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMLL(64)?$")>;
 600: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 601-640 / 第 601-640 行
```tablegen
 601: // Compare logical characters under mask
 602: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr],
 603:              (instregex "CLM(H|Y)?$")>;
 604: 
 605: //===----------------------------------------------------------------------===//
 606: // Prefetch and execution hint
 607: //===----------------------------------------------------------------------===//
 608: 
 609: def : InstRW<[WLat1, LSU, NormalGr], (instregex "PFD(RL)?$")>;
 610: def : InstRW<[WLat1, FXb, NormalGr], (instregex "BPP$")>;
 611: def : InstRW<[FXb, EndGroup], (instregex "BPRP$")>;
 612: def : InstRW<[WLat1, FXb, NormalGr], (instregex "NIAI$")>;
 613: 
 614: //===----------------------------------------------------------------------===//
 615: // Atomic operations
 616: //===----------------------------------------------------------------------===//
 617: 
 618: def : InstRW<[WLat1, FXb, EndGroup], (instregex "Serialize$")>;
 619: 
 620: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAA(G)?$")>;
 621: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAAL(G)?$")>;
 622: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAN(G)?$")>;
 623: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAO(G)?$")>;
 624: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAX(G)?$")>;
 625: 
 626: // Test and set
 627: def : InstRW<[WLat2LSU, FXb, LSU, EndGroup], (instregex "TS$")>;
 628: 
 629: // Compare and swap
 630: def : InstRW<[WLat3LSU, WLat3LSU, FXa, FXb, LSU, GroupAlone],
 631:              (instregex "CS(G|Y)?$")>;
 632: 
 633: // Compare double and swap
 634: def : InstRW<[WLat6LSU, WLat6LSU, FXa3, FXb2, LSU, GroupAlone2],
 635:              (instregex "CDS(Y)?$")>;
 636: def : InstRW<[WLat15, WLat15, FXa2, FXb4, LSU3,
 637:               GroupAlone3], (instregex "CDSG$")>;
 638: 
 639: // Compare and swap and store
 640: def : InstRW<[WLat30, MCD], (instregex "CSST$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 641-680 / 第 641-680 行
```tablegen
 641: 
 642: // Perform locked operation
 643: def : InstRW<[WLat30, MCD], (instregex "PLO$")>;
 644: 
 645: // Load/store pair from/to quadword
 646: def : InstRW<[WLat4LSU, LSU2, GroupAlone], (instregex "LPQ$")>;
 647: def : InstRW<[WLat1, FXb2, LSU, GroupAlone], (instregex "STPQ$")>;
 648: 
 649: // Load pair disjoint
 650: def : InstRW<[WLat1LSU, WLat1LSU, LSU2, GroupAlone], (instregex "LPD(G)?$")>;
 651: 
 652: //===----------------------------------------------------------------------===//
 653: // Translate and convert
 654: //===----------------------------------------------------------------------===//
 655: 
 656: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "TR$")>;
 657: def : InstRW<[WLat30, WLat30, WLat30, FXa3, LSU2, GroupAlone2],
 658:              (instregex "TRT$")>;
 659: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRTR$")>;
 660: def : InstRW<[WLat30, WLat30, MCD], (instregex "TRE$")>;
 661: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRT(R)?E(Opt)?$")>;
 662: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TR(T|O)(T|O)(Opt)?$")>;
 663: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 664:              (instregex "CU(12|14|21|24|41|42)(Opt)?$")>;
 665: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "(CUUTF|CUTFU)(Opt)?$")>;
 666: 
 667: //===----------------------------------------------------------------------===//
 668: // Message-security assist
 669: //===----------------------------------------------------------------------===//
 670: 
 671: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD],
 672:              (instregex "KM(C|F|O|CTR|A)?$")>;
 673: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 674:              (instregex "(KIMD|KLMD|KMAC|KDSA)$")>;
 675: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 676:              (instregex "(PCC|PPNO|PRNO)$")>;
 677: 
 678: //===----------------------------------------------------------------------===//
 679: // Guarded storage
 680: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 681-720 / 第 681-720 行
```tablegen
 681: 
 682: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LGG$")>;
 683: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLGFSG$")>;
 684: def : InstRW<[WLat30, MCD], (instregex "(L|ST)GSC$")>;
 685: 
 686: //===----------------------------------------------------------------------===//
 687: // Decimal arithmetic
 688: //===----------------------------------------------------------------------===//
 689: 
 690: def : InstRW<[WLat20, RegReadAdv, FXb, VecDF2, LSU2, GroupAlone2],
 691:              (instregex "CVBG$")>;
 692: def : InstRW<[WLat20, RegReadAdv, FXb, VecDF, LSU, GroupAlone2],
 693:              (instregex "CVB(Y)?$")>;
 694: def : InstRW<[WLat1, FXb3, VecDF4, LSU, GroupAlone3], (instregex "CVDG$")>;
 695: def : InstRW<[WLat1, FXb2, VecDF, LSU, GroupAlone2], (instregex "CVD(Y)?$")>;
 696: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "MV(N|O|Z)$")>;
 697: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "(PACK|PKA|PKU)$")>;
 698: def : InstRW<[WLat12, LSU5, GroupAlone], (instregex "UNPK(A|U)$")>;
 699: def : InstRW<[WLat1, FXb, LSU2, Cracked], (instregex "UNPK$")>;
 700: 
 701: def : InstRW<[WLat5LSU, FXb, VecDFX, LSU3, GroupAlone2],
 702:              (instregex "(A|S|ZA)P$")>;
 703: def : InstRW<[WLat1, FXb, VecDFX2, LSU3, GroupAlone2], (instregex "MP$")>;
 704: def : InstRW<[WLat1, FXb, VecDFX4, LSU3, GroupAlone2], (instregex "DP$")>;
 705: def : InstRW<[WLat15, FXb, VecDFX2, LSU2, GroupAlone3], (instregex "SRP$")>;
 706: def : InstRW<[WLat8, VecDFX, LSU, LSU, GroupAlone], (instregex "CP$")>;
 707: def : InstRW<[WLat3LSU, VecDFX, LSU, Cracked], (instregex "TP$")>;
 708: def : InstRW<[WLat30, MCD], (instregex "ED(MK)?$")>;
 709: 
 710: //===----------------------------------------------------------------------===//
 711: // Access registers
 712: //===----------------------------------------------------------------------===//
 713: 
 714: // Extract/set/copy access register
 715: def : InstRW<[WLat3, LSU, NormalGr], (instregex "(EAR|SAR|CPYA)$")>;
 716: 
 717: // Load address extended
 718: def : InstRW<[WLat5, LSU, FXa, Cracked], (instregex "LAE(Y)?$")>;
 719: 
 720: // Load/store access multiple (not modeled precisely)
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: def : InstRW<[WLat20, WLat20, LSU5, GroupAlone], (instregex "LAM(Y)?$")>;
 722: def : InstRW<[WLat1, LSU5, FXb, GroupAlone2], (instregex "STAM(Y)?$")>;
 723: 
 724: //===----------------------------------------------------------------------===//
 725: // Program mask and addressing mode
 726: //===----------------------------------------------------------------------===//
 727: 
 728: // Insert Program Mask
 729: def : InstRW<[WLat3, FXa, EndGroup], (instregex "IPM$")>;
 730: 
 731: // Set Program Mask
 732: def : InstRW<[WLat3, LSU, EndGroup], (instregex "SPM$")>;
 733: 
 734: // Branch and link
 735: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "BAL(R)?$")>;
 736: 
 737: // Test addressing mode
 738: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TAM$")>;
 739: 
 740: // Set addressing mode
 741: def : InstRW<[WLat1, FXb, EndGroup], (instregex "SAM(24|31|64)$")>;
 742: 
 743: // Branch (and save) and set mode.
 744: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BSM$")>;
 745: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "BASSM$")>;
 746: 
 747: //===----------------------------------------------------------------------===//
 748: // Transactional execution
 749: //===----------------------------------------------------------------------===//
 750: 
 751: // Transaction begin
 752: def : InstRW<[WLat9, LSU2, FXb5, GroupAlone2], (instregex "TBEGIN(C)?$")>;
 753: 
 754: // Transaction end
 755: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "TEND$")>;
 756: 
 757: // Transaction abort
 758: def : InstRW<[WLat30, MCD], (instregex "TABORT$")>;
 759: 
 760: // Extract Transaction Nesting Depth
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 761-800 / 第 761-800 行
```tablegen
 761: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ETND$")>;
 762: 
 763: // Nontransactional store
 764: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "NTSTG$")>;
 765: 
 766: //===----------------------------------------------------------------------===//
 767: // Processor assist
 768: //===----------------------------------------------------------------------===//
 769: 
 770: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "PPA$")>;
 771: 
 772: //===----------------------------------------------------------------------===//
 773: // Miscellaneous Instructions.
 774: //===----------------------------------------------------------------------===//
 775: 
 776: // Find leftmost one
 777: def : InstRW<[WLat5, WLat5, FXa2, GroupAlone], (instregex "FLOGR$")>;
 778: 
 779: // Population count
 780: def : InstRW<[WLat3, WLat3, FXa, NormalGr], (instregex "POPCNT(Opt)?$")>;
 781: 
 782: // String instructions
 783: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "SRST(U)?$")>;
 784: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CUSE$")>;
 785: 
 786: // Various complex instructions
 787: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CFC$")>;
 788: def : InstRW<[WLat30, WLat30, WLat30, WLat30, WLat30, WLat30, MCD],
 789:              (instregex "UPT$")>;
 790: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CKSM$")>;
 791: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CMPSC$")>;
 792: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "SORTL$")>;
 793: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "DFLTCC$")>;
 794: 
 795: // Execute
 796: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "EX(RL)?$")>;
 797: 
 798: //===----------------------------------------------------------------------===//
 799: // .insn directive instructions
 800: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: 
 802: // An "empty" sched-class will be assigned instead of the "invalid sched-class".
 803: // getNumDecoderSlots() will then return 1 instead of 0.
 804: def : InstRW<[], (instregex "Insn.*")>;
 805: 
 806: 
 807: // ----------------------------- Floating point ----------------------------- //
 808: 
 809: //===----------------------------------------------------------------------===//
 810: // FP: Move instructions
 811: //===----------------------------------------------------------------------===//
 812: 
 813: // Load zero
 814: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LZ(DR|ER|ER_16)$")>;
 815: def : InstRW<[WLat2, FXb2, Cracked], (instregex "LZXR$")>;
 816: 
 817: // Load
 818: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "LER(16)?$")>;
 819: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LD(R|R16|R32|GR)$")>;
 820: def : InstRW<[WLat3, FXb, NormalGr], (instregex "LGDR$")>;
 821: def : InstRW<[WLat2, FXb2, GroupAlone], (instregex "LXR$")>;
 822: 
 823: // Load and Test
 824: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "LT(E|D)BR$")>;
 825: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXBR$")>;
 826: 
 827: // Copy sign
 828: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "CPSDR(d|s|h)(d|s|h)$")>;
 829: 
 830: //===----------------------------------------------------------------------===//
 831: // FP: Load instructions
 832: //===----------------------------------------------------------------------===//
 833: 
 834: def : InstRW<[WLat2LSU, VecXsPm, LSU, NormalGr], (instregex "L(E16|E)(Y)?$")>;
 835: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LD(Y|E32)?$")>;
 836: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LX$")>;
 837: 
 838: //===----------------------------------------------------------------------===//
 839: // FP: Store instructions
 840: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: 
 842: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST(E16|E|D)(Y)?$")>;
 843: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STX$")>;
 844: 
 845: //===----------------------------------------------------------------------===//
 846: // FP: Conversion instructions
 847: //===----------------------------------------------------------------------===//
 848: 
 849: // Load rounded
 850: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "LEDBR(A)?$")>;
 851: def : InstRW<[WLat9, VecDF2, NormalGr], (instregex "L(E|D)XBR(A)?$")>;
 852: 
 853: // Load lengthened
 854: def : InstRW<[WLat6LSU, VecBF, LSU, NormalGr], (instregex "LDEB$")>;
 855: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "LDEBR$")>;
 856: def : InstRW<[WLat7LSU, VecBF4, LSU, GroupAlone], (instregex "LX(E|D)B$")>;
 857: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "LX(E|D)BR$")>;
 858: 
 859: // Convert from fixed / logical
 860: def : InstRW<[WLat7, FXb, VecBF, Cracked], (instregex "C(E|D)(F|G)BR(A)?$")>;
 861: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)BR(A)?$")>;
 862: def : InstRW<[WLat7, FXb, VecBF, Cracked], (instregex "C(E|D)L(F|G)BR$")>;
 863: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CXL(F|G)BR$")>;
 864: 
 865: // Convert to fixed / logical
 866: def : InstRW<[WLat9, WLat9, FXb, VecBF, Cracked],
 867:              (instregex "C(F|G)(E|D)BR(A)?$")>;
 868: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked],
 869:              (instregex "C(F|G)XBR(A)?$")>;
 870: def : InstRW<[WLat9, WLat9, FXb, VecBF, GroupAlone], (instregex "CLFEBR$")>;
 871: def : InstRW<[WLat9, WLat9, FXb, VecBF, Cracked], (instregex "CLFDBR$")>;
 872: def : InstRW<[WLat9, WLat9, FXb, VecBF, Cracked], (instregex "CLG(E|D)BR$")>;
 873: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked], (instregex "CL(F|G)XBR$")>;
 874: 
 875: //===----------------------------------------------------------------------===//
 876: // FP: Unary arithmetic
 877: //===----------------------------------------------------------------------===//
 878: 
 879: // Load Complement / Negative / Positive
 880: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "L(C|N|P)(E|D)BR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 881-920 / 第 881-920 行
```tablegen
 881: def : InstRW<[WLat1, FXb, NormalGr], (instregex "L(C|N|P)DFR(_32|_16)?$")>;
 882: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "L(C|N|P)XBR$")>;
 883: 
 884: // Square root
 885: def : InstRW<[WLat30, VecFPd, LSU, NormalGr], (instregex "SQ(E|D)B$")>;
 886: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "SQ(E|D)BR$")>;
 887: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "SQXBR$")>;
 888: 
 889: // Load FP integer
 890: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "FI(E|D)BR(A)?$")>;
 891: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXBR(A)?$")>;
 892: 
 893: //===----------------------------------------------------------------------===//
 894: // FP: Binary arithmetic
 895: //===----------------------------------------------------------------------===//
 896: 
 897: // Addition
 898: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
 899:              (instregex "A(E|D)B$")>;
 900: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "A(E|D)BR$")>;
 901: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXBR$")>;
 902: 
 903: // Subtraction
 904: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
 905:              (instregex "S(E|D)B$")>;
 906: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "S(E|D)BR$")>;
 907: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXBR$")>;
 908: 
 909: // Multiply
 910: def : InstRW<[WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
 911:              (instregex "M(D|DE|EE)B$")>;
 912: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "M(D|DE|EE)BR$")>;
 913: def : InstRW<[WLat7LSU, RegReadAdv, VecBF4, LSU, GroupAlone],
 914:              (instregex "MXDB$")>;
 915: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "MXDBR$")>;
 916: def : InstRW<[WLat15, VecDF4, GroupAlone], (instregex "MXBR$")>;
 917: 
 918: // Multiply and add / subtract
 919: def : InstRW<[WLat6LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
 920:              (instregex "M(A|S)EB$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: def : InstRW<[WLat6, VecBF, GroupAlone], (instregex "M(A|S)EBR$")>;
 922: def : InstRW<[WLat6LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
 923:              (instregex "M(A|S)DB$")>;
 924: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "M(A|S)DBR$")>;
 925: 
 926: // Division
 927: def : InstRW<[WLat30, RegReadAdv, VecFPd, LSU, NormalGr],
 928:              (instregex "D(E|D)B$")>;
 929: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "D(E|D)BR$")>;
 930: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "DXBR$")>;
 931: 
 932: // Divide to integer
 933: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "DI(E|D)BR$")>;
 934: 
 935: //===----------------------------------------------------------------------===//
 936: // FP: Comparisons
 937: //===----------------------------------------------------------------------===//
 938: 
 939: // Compare
 940: def : InstRW<[WLat3LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
 941:              (instregex "(K|C)(E|D)B$")>;
 942: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "(K|C)(E|D)BR$")>;
 943: def : InstRW<[WLat9, VecDF2, GroupAlone], (instregex "(K|C)XBR$")>;
 944: 
 945: // Test Data Class
 946: def : InstRW<[WLat5, LSU, VecXsPm, NormalGr], (instregex "TC(E|D)B$")>;
 947: def : InstRW<[WLat10, LSU, VecDF4, GroupAlone], (instregex "TCXB$")>;
 948: 
 949: //===----------------------------------------------------------------------===//
 950: // FP: Floating-point control register instructions
 951: //===----------------------------------------------------------------------===//
 952: 
 953: def : InstRW<[WLat4, FXa, LSU, GroupAlone], (instregex "EFPC$")>;
 954: def : InstRW<[WLat1, FXb, LSU, GroupAlone], (instregex "STFPC$")>;
 955: def : InstRW<[WLat3, LSU, GroupAlone], (instregex "SFPC$")>;
 956: def : InstRW<[WLat3LSU, LSU2, GroupAlone], (instregex "LFPC$")>;
 957: def : InstRW<[WLat30, MCD], (instregex "SFASR$")>;
 958: def : InstRW<[WLat30, MCD], (instregex "LFAS$")>;
 959: def : InstRW<[WLat3, FXb, GroupAlone], (instregex "SRNM(B|T)?$")>;
 960: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: 
 962: // --------------------- Hexadecimal floating point ------------------------- //
 963: 
 964: //===----------------------------------------------------------------------===//
 965: // HFP: Move instructions
 966: //===----------------------------------------------------------------------===//
 967: 
 968: // Load and Test
 969: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "LT(E|D)R$")>;
 970: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXR$")>;
 971: 
 972: //===----------------------------------------------------------------------===//
 973: // HFP: Conversion instructions
 974: //===----------------------------------------------------------------------===//
 975: 
 976: // Load rounded
 977: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "(LEDR|LRER)$")>;
 978: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "LEXR$")>;
 979: def : InstRW<[WLat9, VecDF2, NormalGr], (instregex "(LDXR|LRDR)$")>;
 980: 
 981: // Load lengthened
 982: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LDE$")>;
 983: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LDER$")>;
 984: def : InstRW<[WLat7LSU, VecBF4, LSU, GroupAlone], (instregex "LX(E|D)$")>;
 985: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "LX(E|D)R$")>;
 986: 
 987: // Convert from fixed
 988: def : InstRW<[WLat7, FXb, VecBF, Cracked], (instregex "C(E|D)(F|G)R$")>;
 989: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)R$")>;
 990: 
 991: // Convert to fixed
 992: def : InstRW<[WLat9, WLat9, FXb, VecBF, Cracked], (instregex "C(F|G)(E|D)R$")>;
 993: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked], (instregex "C(F|G)XR$")>;
 994: 
 995: // Convert BFP to HFP / HFP to BFP.
 996: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "THD(E)?R$")>;
 997: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "TB(E)?DR$")>;
 998: 
 999: //===----------------------------------------------------------------------===//
1000: // HFP: Unary arithmetic
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: //===----------------------------------------------------------------------===//
1002: 
1003: // Load Complement / Negative / Positive
1004: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "L(C|N|P)(E|D)R$")>;
1005: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "L(C|N|P)XR$")>;
1006: 
1007: // Halve
1008: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "H(E|D)R$")>;
1009: 
1010: // Square root
1011: def : InstRW<[WLat30, VecFPd, LSU, NormalGr], (instregex "SQ(E|D)$")>;
1012: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "SQ(E|D)R$")>;
1013: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "SQXR$")>;
1014: 
1015: // Load FP integer
1016: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "FI(E|D)R$")>;
1017: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXR$")>;
1018: 
1019: //===----------------------------------------------------------------------===//
1020: // HFP: Binary arithmetic
1021: //===----------------------------------------------------------------------===//
1022: 
1023: // Addition
1024: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
1025:              (instregex "A(E|D|U|W)$")>;
1026: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "A(E|D|U|W)R$")>;
1027: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXR$")>;
1028: 
1029: // Subtraction
1030: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
1031:              (instregex "S(E|D|U|W)$")>;
1032: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "S(E|D|U|W)R$")>;
1033: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXR$")>;
1034: 
1035: // Multiply
1036: def : InstRW<[WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
1037:              (instregex "M(D|DE|E|EE)$")>;
1038: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "M(D|DE|E|EE)R$")>;
1039: def : InstRW<[WLat7LSU, RegReadAdv, VecBF4, LSU, GroupAlone],
1040:              (instregex "MXD$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "MXDR$")>;
1042: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "MXR$")>;
1043: def : InstRW<[WLat7LSU, RegReadAdv, VecBF4, LSU, GroupAlone], (instregex "MY$")>;
1044: def : InstRW<[WLat6LSU, RegReadAdv, VecBF2, LSU, GroupAlone],
1045:              (instregex "MY(H|L)$")>;
1046: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "MYR$")>;
1047: def : InstRW<[WLat6, VecBF, GroupAlone], (instregex "MY(H|L)R$")>;
1048: 
1049: // Multiply and add / subtract
1050: def : InstRW<[WLat6LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
1051:              (instregex "M(A|S)(E|D)$")>;
1052: def : InstRW<[WLat6, VecBF, GroupAlone], (instregex "M(A|S)(E|D)R$")>;
1053: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF4, LSU, GroupAlone],
1054:              (instregex "MAY$")>;
1055: def : InstRW<[WLat6LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
1056:              (instregex "MAY(H|L)$")>;
1057: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "MAYR$")>;
1058: def : InstRW<[WLat6, VecBF, GroupAlone], (instregex "MAY(H|L)R$")>;
1059: 
1060: // Division
1061: def : InstRW<[WLat30, RegReadAdv, VecFPd, LSU, NormalGr], (instregex "D(E|D)$")>;
1062: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "D(E|D)R$")>;
1063: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "DXR$")>;
1064: 
1065: //===----------------------------------------------------------------------===//
1066: // HFP: Comparisons
1067: //===----------------------------------------------------------------------===//
1068: 
1069: // Compare
1070: def : InstRW<[WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
1071:              (instregex "C(E|D)$")>;
1072: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "C(E|D)R$")>;
1073: def : InstRW<[WLat10, VecDF2, GroupAlone], (instregex "CXR$")>;
1074: 
1075: 
1076: // ------------------------ Decimal floating point -------------------------- //
1077: 
1078: //===----------------------------------------------------------------------===//
1079: // DFP: Move instructions
1080: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: 
1082: // Load and Test
1083: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "LTDTR$")>;
1084: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXTR$")>;
1085: 
1086: //===----------------------------------------------------------------------===//
1087: // DFP: Conversion instructions
1088: //===----------------------------------------------------------------------===//
1089: 
1090: // Load rounded
1091: def : InstRW<[WLat15, VecDF, NormalGr], (instregex "LEDTR$")>;
1092: def : InstRW<[WLat15, VecDF2, NormalGr], (instregex "LDXTR$")>;
1093: 
1094: // Load lengthened
1095: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "LDETR$")>;
1096: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "LXDTR$")>;
1097: 
1098: // Convert from fixed / logical
1099: def : InstRW<[WLat15, FXb, VecDF, Cracked], (instregex "CDFTR(A)?$")>;
1100: def : InstRW<[WLat30, FXb, VecDF, Cracked], (instregex "CDGTR(A)?$")>;
1101: def : InstRW<[WLat15, FXb, VecDF4, GroupAlone2], (instregex "CXFTR(A)?$")>;
1102: def : InstRW<[WLat30, FXb, VecDF4, GroupAlone2], (instregex "CXGTR(A)?$")>;
1103: def : InstRW<[WLat15, FXb, VecDF, Cracked], (instregex "CDLFTR$")>;
1104: def : InstRW<[WLat30, FXb, VecDF, Cracked], (instregex "CDLGTR$")>;
1105: def : InstRW<[WLat15, FXb, VecDF4, GroupAlone2], (instregex "CXLFTR$")>;
1106: def : InstRW<[WLat30, FXb, VecDF4, GroupAlone2], (instregex "CXLGTR$")>;
1107: 
1108: // Convert to fixed / logical
1109: def : InstRW<[WLat30, WLat30, FXb, VecDF, Cracked],
1110:              (instregex "C(F|G)DTR(A)?$")>;
1111: def : InstRW<[WLat30, WLat30, FXb, VecDF2, Cracked],
1112:              (instregex "C(F|G)XTR(A)?$")>;
1113: def : InstRW<[WLat30, WLat30, FXb, VecDF, Cracked], (instregex "CL(F|G)DTR$")>;
1114: def : InstRW<[WLat30, WLat30, FXb, VecDF2, Cracked], (instregex "CL(F|G)XTR$")>;
1115: 
1116: // Convert from / to signed / unsigned packed
1117: def : InstRW<[WLat9, FXb, VecDF, Cracked], (instregex "CD(S|U)TR$")>;
1118: def : InstRW<[WLat12, FXb2, VecDF4, GroupAlone2], (instregex "CX(S|U)TR$")>;
1119: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "C(S|U)DTR$")>;
1120: def : InstRW<[WLat15, FXb2, VecDF4, GroupAlone2], (instregex "C(S|U)XTR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: 
1122: // Convert from / to zoned
1123: def : InstRW<[WLat8LSU, LSU, VecDF, Cracked], (instregex "CDZT$")>;
1124: def : InstRW<[WLat16LSU, LSU2, VecDF4, GroupAlone3], (instregex "CXZT$")>;
1125: def : InstRW<[WLat1, FXb, LSU, VecDF, Cracked], (instregex "CZDT$")>;
1126: def : InstRW<[WLat1, FXb, LSU, VecDF2, GroupAlone], (instregex "CZXT$")>;
1127: 
1128: // Convert from / to packed
1129: def : InstRW<[WLat8LSU, LSU, VecDF, Cracked], (instregex "CDPT$")>;
1130: def : InstRW<[WLat16LSU, LSU2, VecDF4, GroupAlone3], (instregex "CXPT$")>;
1131: def : InstRW<[WLat1, FXb, LSU, VecDF, Cracked], (instregex "CPDT$")>;
1132: def : InstRW<[WLat1, FXb, LSU, VecDF2, GroupAlone], (instregex "CPXT$")>;
1133: 
1134: // Perform floating-point operation
1135: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "PFPO$")>;
1136: 
1137: //===----------------------------------------------------------------------===//
1138: // DFP: Unary arithmetic
1139: //===----------------------------------------------------------------------===//
1140: 
1141: // Load FP integer
1142: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "FIDTR$")>;
1143: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXTR$")>;
1144: 
1145: // Extract biased exponent
1146: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "EEDTR$")>;
1147: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "EEXTR$")>;
1148: 
1149: // Extract significance
1150: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "ESDTR$")>;
1151: def : InstRW<[WLat12, FXb, VecDF2, Cracked], (instregex "ESXTR$")>;
1152: 
1153: //===----------------------------------------------------------------------===//
1154: // DFP: Binary arithmetic
1155: //===----------------------------------------------------------------------===//
1156: 
1157: // Addition
1158: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "ADTR(A)?$")>;
1159: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXTR(A)?$")>;
1160: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161: // Subtraction
1162: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "SDTR(A)?$")>;
1163: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXTR(A)?$")>;
1164: 
1165: // Multiply
1166: def : InstRW<[WLat30, VecDF, NormalGr], (instregex "MDTR(A)?$")>;
1167: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "MXTR(A)?$")>;
1168: 
1169: // Division
1170: def : InstRW<[WLat30, VecDF, NormalGr], (instregex "DDTR(A)?$")>;
1171: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "DXTR(A)?$")>;
1172: 
1173: // Quantize
1174: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "QADTR$")>;
1175: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "QAXTR$")>;
1176: 
1177: // Reround
1178: def : InstRW<[WLat9, WLat9, FXb, VecDF, Cracked], (instregex "RRDTR$")>;
1179: def : InstRW<[WLat11, WLat11, FXb, VecDF4, GroupAlone2], (instregex "RRXTR$")>;
1180: 
1181: // Shift significand left/right
1182: def : InstRW<[WLat11LSU, LSU, VecDF, GroupAlone], (instregex "S(L|R)DT$")>;
1183: def : InstRW<[WLat11LSU, LSU, VecDF4, GroupAlone], (instregex "S(L|R)XT$")>;
1184: 
1185: // Insert biased exponent
1186: def : InstRW<[WLat9, FXb, VecDF, Cracked], (instregex "IEDTR$")>;
1187: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "IEXTR$")>;
1188: 
1189: //===----------------------------------------------------------------------===//
1190: // DFP: Comparisons
1191: //===----------------------------------------------------------------------===//
1192: 
1193: // Compare
1194: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "(K|C)DTR$")>;
1195: def : InstRW<[WLat9, VecDF2, GroupAlone], (instregex "(K|C)XTR$")>;
1196: 
1197: // Compare biased exponent
1198: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "CEDTR$")>;
1199: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "CEXTR$")>;
1200: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201: // Test Data Class/Group
1202: def : InstRW<[WLat15, LSU, VecDF, NormalGr], (instregex "TD(C|G)(E|D)T$")>;
1203: def : InstRW<[WLat15, LSU, VecDF2, GroupAlone], (instregex "TD(C|G)XT$")>;
1204: 
1205: 
1206: // --------------------------------- Vector --------------------------------- //
1207: 
1208: //===----------------------------------------------------------------------===//
1209: // Vector: Move instructions
1210: //===----------------------------------------------------------------------===//
1211: 
1212: def : InstRW<[WLat1, FXb, NormalGr], (instregex "VLR(16|32|64)?$")>;
1213: def : InstRW<[WLat3, FXb, NormalGr], (instregex "VLGV(B|F|G|H)?$")>;
1214: def : InstRW<[WLat1, FXb, NormalGr], (instregex "VLVG(B|F|G|H)?$")>;
1215: def : InstRW<[WLat3, FXb, NormalGr], (instregex "VLVGP(32)?$")>;
1216: 
1217: //===----------------------------------------------------------------------===//
1218: // Vector: Immediate instructions
1219: //===----------------------------------------------------------------------===//
1220: 
1221: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VZERO$")>;
1222: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VONE$")>;
1223: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VGBM$")>;
1224: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VGM(B|F|G|H)?$")>;
1225: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VREPI(B|F|G|H)?$")>;
1226: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLEI(B|F|G|H)$")>;
1227: 
1228: //===----------------------------------------------------------------------===//
1229: // Vector: Loads
1230: //===----------------------------------------------------------------------===//
1231: 
1232: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(Align)?$")>;
1233: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(L|BB)$")>;
1234: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(16|32|64)$")>;
1235: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLLEZ(B|F|G|H|LF)?$")>;
1236: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLREP(B|F|G|H)?$")>;
1237: def : InstRW<[WLat2LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
1238:              (instregex "VLE(B|F|G|H)$")>;
1239: def : InstRW<[WLat5LSU, RegReadAdv, FXb, LSU, VecXsPm, Cracked],
1240:              (instregex "VGE(F|G)$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241: def : InstRW<[WLat4LSU, WLat4LSU, LSU5, GroupAlone],
1242:              (instregex "VLM(Align)?$")>;
1243: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLRL(R)?$")>;
1244: 
1245: //===----------------------------------------------------------------------===//
1246: // Vector: Stores
1247: //===----------------------------------------------------------------------===//
1248: 
1249: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VST(Align|L|16|32|64)?$")>;
1250: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTE(F|G)$")>;
1251: def : InstRW<[WLat1, FXb, LSU, VecXsPm, Cracked], (instregex "VSTE(B|H)$")>;
1252: def : InstRW<[WLat1, LSU2, FXb3, GroupAlone2], (instregex "VSTM(Align)?$")>;
1253: def : InstRW<[WLat1, FXb2, LSU, Cracked], (instregex "VSCE(F|G)$")>;
1254: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTRL(R)?$")>;
1255: 
1256: //===----------------------------------------------------------------------===//
1257: // Vector: Byte swaps
1258: //===----------------------------------------------------------------------===//
1259: 
1260: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLBR(H|F|G|Q)?$")>;
1261: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLER(H|F|G)?$")>;
1262: def : InstRW<[WLat2LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
1263:              (instregex "VLEBR(H|F|G)$")>;
1264: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLLEBRZ(H|F|G|E)?$")>;
1265: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLBRREP(H|F|G)?$")>;
1266: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTBR(H|F|G|Q)?$")>;
1267: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTER(H|F|G)?$")>;
1268: def : InstRW<[WLat1, FXb, LSU, VecXsPm, Cracked], (instregex "VSTEBRH$")>;
1269: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTEBR(F|G)$")>;
1270: 
1271: //===----------------------------------------------------------------------===//
1272: // Vector: Selects and permutes
1273: //===----------------------------------------------------------------------===//
1274: 
1275: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMRH(B|F|G|H)?$")>;
1276: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMRL(B|F|G|H)?$")>;
1277: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPERM$")>;
1278: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPDI$")>;
1279: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VBPERM$")>;
1280: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VREP(B|F|G|H)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSEL$")>;
1282: 
1283: //===----------------------------------------------------------------------===//
1284: // Vector: Widening and narrowing
1285: //===----------------------------------------------------------------------===//
1286: 
1287: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPK(F|G|H)?$")>;
1288: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPKS(F|G|H)?$")>;
1289: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VPKS(F|G|H)S$")>;
1290: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPKLS(F|G|H)?$")>;
1291: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VPKLS(F|G|H)S$")>;
1292: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSEG(B|F|H)?$")>;
1293: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPH(B|F|H)?$")>;
1294: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPL(B|F)?$")>;
1295: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPLH(B|F|H|W)?$")>;
1296: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPLL(B|F|H)?$")>;
1297: 
1298: //===----------------------------------------------------------------------===//
1299: // Vector: Integer arithmetic
1300: //===----------------------------------------------------------------------===//
1301: 
1302: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VA(B|F|G|H|Q|C|CQ)?$")>;
1303: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VACC(B|F|G|H|Q|C|CQ)?$")>;
1304: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VAVG(B|F|G|H)?$")>;
1305: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VAVGL(B|F|G|H)?$")>;
1306: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VN(C|O|N|X)?$")>;
1307: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VO(C)?$")>;
1308: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VCKSM$")>;
1309: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCLZ(B|F|G|H)?$")>;
1310: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCTZ(B|F|G|H)?$")>;
1311: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VX$")>;
1312: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFM?$")>;
1313: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFMA(B|F|G|H)?$")>;
1314: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFM(B|F|G|H)$")>;
1315: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLC(B|F|G|H)?$")>;
1316: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLP(B|F|G|H)?$")>;
1317: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMX(B|F|G|H)?$")>;
1318: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMXL(B|F|G|H)?$")>;
1319: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMN(B|F|G|H)?$")>;
1320: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMNL(B|F|G|H)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1321-1360 / 第 1321-1360 行
```tablegen
1321: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAL(B|F)?$")>;
1322: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALE(B|F|H)?$")>;
1323: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALH(B|F|H|W)?$")>;
1324: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALO(B|F|H)?$")>;
1325: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAO(B|F|H)?$")>;
1326: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAE(B|F|H)?$")>;
1327: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAH(B|F|H)?$")>;
1328: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VME(B|F|H)?$")>;
1329: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMH(B|F|H)?$")>;
1330: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VML(B|F)?$")>;
1331: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLE(B|F|H)?$")>;
1332: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLH(B|F|H|W)?$")>;
1333: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLO(B|F|H)?$")>;
1334: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMO(B|F|H)?$")>;
1335: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VMSL(G)?$")>;
1336: 
1337: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPOPCT(B|F|G|H)?$")>;
1338: 
1339: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERLL(B|F|G|H)?$")>;
1340: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERLLV(B|F|G|H)?$")>;
1341: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERIM(B|F|G|H)?$")>;
1342: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESL(B|F|G|H)?$")>;
1343: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESLV(B|F|G|H)?$")>;
1344: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRA(B|F|G|H)?$")>;
1345: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRAV(B|F|G|H)?$")>;
1346: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRL(B|F|G|H)?$")>;
1347: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRLV(B|F|G|H)?$")>;
1348: 
1349: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSL(DB)?$")>;
1350: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSLB$")>;
1351: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSR(A|L)$")>;
1352: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSR(A|L)B$")>;
1353: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSLD$")>;
1354: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSRD$")>;
1355: 
1356: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSB(I|IQ|CBI|CBIQ)?$")>;
1357: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSCBI(B|F|G|H|Q)?$")>;
1358: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VS(F|G|H|Q)?$")>;
1359: 
1360: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUM(B|H)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1361-1400 / 第 1361-1400 行
```tablegen
1361: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUMG(F|H)?$")>;
1362: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUMQ(F|G)?$")>;
1363: 
1364: //===----------------------------------------------------------------------===//
1365: // Vector: Integer comparison
1366: //===----------------------------------------------------------------------===//
1367: 
1368: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "VEC(B|F|G|H)?$")>;
1369: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "VECL(B|F|G|H)?$")>;
1370: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCEQ(B|F|G|H)?$")>;
1371: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCEQ(B|F|G|H)S$")>;
1372: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCH(B|F|G|H)?$")>;
1373: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCH(B|F|G|H)S$")>;
1374: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCHL(B|F|G|H)?$")>;
1375: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCHL(B|F|G|H)S$")>;
1376: def : InstRW<[WLat4, VecStr, NormalGr], (instregex "VTM$")>;
1377: 
1378: //===----------------------------------------------------------------------===//
1379: // Vector: Floating-point arithmetic
1380: //===----------------------------------------------------------------------===//
1381: 
1382: // Conversion and rounding
1383: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VCFP(S|L)$")>;
1384: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VCD(L)?G$")>;
1385: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VCD(L)?GB$")>;
1386: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WCD(L)?GB$")>;
1387: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VCE(L)?FB$")>;
1388: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WCE(L)?FB$")>;
1389: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VC(S|L)FP$")>;
1390: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VC(L)?GD$")>;
1391: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VC(L)?GDB$")>;
1392: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WC(L)?GDB$")>;
1393: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VC(L)?FEB$")>;
1394: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WC(L)?FEB$")>;
1395: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VL(DE|ED)$")>;
1396: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VL(DE|ED)B$")>;
1397: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WL(DE|ED)B$")>;
1398: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFL(L|R)$")>;
1399: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFL(LS|RD)$")>;
1400: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFL(LS|RD)$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1401-1440 / 第 1401-1440 行
```tablegen
1401: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFLLD$")>;
1402: def : InstRW<[WLat10, VecDF2, NormalGr], (instregex "WFLRX$")>;
1403: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFI(DB)?$")>;
1404: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFIDB$")>;
1405: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFISB$")>;
1406: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFISB$")>;
1407: def : InstRW<[WLat10, VecDF2, NormalGr], (instregex "WFIXB$")>;
1408: 
1409: // Sign operations
1410: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VFPSO$")>;
1411: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FPSODB$")>;
1412: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FPSOSB$")>;
1413: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFPSOXB$")>;
1414: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FL(C|N|P)DB$")>;
1415: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FL(C|N|P)SB$")>;
1416: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFL(C|N|P)XB$")>;
1417: 
1418: // Minimum / maximum
1419: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(MAX|MIN)$")>;
1420: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(MAX|MIN)DB$")>;
1421: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WF(MAX|MIN)DB$")>;
1422: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(MAX|MIN)SB$")>;
1423: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WF(MAX|MIN)SB$")>;
1424: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "WF(MAX|MIN)XB$")>;
1425: 
1426: // Test data class
1427: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFTCI$")>;
1428: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "(V|W)FTCIDB$")>;
1429: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "(V|W)FTCISB$")>;
1430: def : InstRW<[WLat3, WLat3, VecDFX, NormalGr], (instregex "WFTCIXB$")>;
1431: 
1432: // Add / subtract
1433: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(A|S)$")>;
1434: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(A|S)DB$")>;
1435: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WF(A|S)DB$")>;
1436: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(A|S)SB$")>;
1437: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WF(A|S)SB$")>;
1438: def : InstRW<[WLat10, VecDF2, NormalGr], (instregex "WF(A|S)XB$")>;
1439: 
1440: // Multiply / multiply-and-add/subtract
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1441-1480 / 第 1441-1480 行
```tablegen
1441: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFM(DB)?$")>;
1442: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFM(D|S)B$")>;
1443: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFMSB$")>;
1444: def : InstRW<[WLat20, VecDF2, NormalGr], (instregex "WFMXB$")>;
1445: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(N)?M(A|S)$")>;
1446: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(N)?M(A|S)DB$")>;
1447: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WF(N)?M(A|S)DB$")>;
1448: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(N)?M(A|S)SB$")>;
1449: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WF(N)?M(A|S)SB$")>;
1450: def : InstRW<[WLat30, VecDF2, NormalGr], (instregex "WF(N)?M(A|S)XB$")>;
1451: 
1452: // Divide / square root
1453: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFD$")>;
1454: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FDDB$")>;
1455: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FDSB$")>;
1456: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "WFDXB$")>;
1457: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFSQ$")>;
1458: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FSQDB$")>;
1459: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FSQSB$")>;
1460: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "WFSQXB$")>;
1461: 
1462: //===----------------------------------------------------------------------===//
1463: // Vector: Floating-point comparison
1464: //===----------------------------------------------------------------------===//
1465: 
1466: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(C|K)(E|H|HE)$")>;
1467: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(C|K)(E|H|HE)DB$")>;
1468: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)DB$")>;
1469: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFK(E|H|HE)DB$")>;
1470: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(C|K)(E|H|HE)SB$")>;
1471: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)SB$")>;
1472: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFK(E|H|HE)SB$")>;
1473: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "WFC(E|H|HE)XB$")>;
1474: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "WFK(E|H|HE)XB$")>;
1475: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFC(E|H|HE)DBS$")>;
1476: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFK(E|H|HE)DBS$")>;
1477: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr],
1478:              (instregex "WF(C|K)(E|H|HE)DBS$")>;
1479: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr],
1480:              (instregex "VF(C|K)(E|H|HE)SBS$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1481-1520 / 第 1481-1520 行
```tablegen
1481: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)SBS$")>;
1482: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "WFK(E|H|HE)SBS$")>;
1483: def : InstRW<[WLat3, WLat3, VecDFX, NormalGr], (instregex "WFC(E|H|HE)XBS$")>;
1484: def : InstRW<[WLat3, WLat3, VecDFX, NormalGr], (instregex "WFK(E|H|HE)XBS$")>;
1485: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)$")>;
1486: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)DB$")>;
1487: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)SB$")>;
1488: def : InstRW<[WLat3, VecDFX, NormalGr], (instregex "WF(C|K)XB$")>;
1489: 
1490: //===----------------------------------------------------------------------===//
1491: // Vector: Floating-point insertion and extraction
1492: //===----------------------------------------------------------------------===//
1493: 
1494: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LEFR(_16)?$")>;
1495: def : InstRW<[WLat3, FXb, NormalGr], (instregex "LFER(_16)?$")>;
1496: 
1497: //===----------------------------------------------------------------------===//
1498: // Vector: String instructions
1499: //===----------------------------------------------------------------------===//
1500: 
1501: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAE(B)?$")>;
1502: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAE(F|H)$")>;
1503: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VFAE(B|F|H)S$")>;
1504: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAEZ(B|F|H)$")>;
1505: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VFAEZ(B|F|H)S$")>;
1506: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFEE(B|F|H|ZB|ZF|ZH)?$")>;
1507: def : InstRW<[WLat4, WLat4, VecStr, NormalGr],
1508:              (instregex "VFEE(B|F|H|ZB|ZF|ZH)S$")>;
1509: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFENE(B|F|H|ZB|ZF|ZH)?$")>;
1510: def : InstRW<[WLat4, WLat4, VecStr, NormalGr],
1511:              (instregex "VFENE(B|F|H|ZB|ZF|ZH)S$")>;
1512: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VISTR(B|F|H)?$")>;
1513: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VISTR(B|F|H)S$")>;
1514: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VSTRC(B|F|H)?$")>;
1515: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRC(B|F|H)S$")>;
1516: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VSTRCZ(B|F|H)$")>;
1517: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRCZ(B|F|H)S$")>;
1518: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRS(B|F|H)?$")>;
1519: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRSZ(B|F|H)$")>;
1520: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1521-1560 / 第 1521-1560 行
```tablegen
1521: //===----------------------------------------------------------------------===//
1522: // Vector: Packed-decimal instructions
1523: //===----------------------------------------------------------------------===//
1524: 
1525: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "VLIP$")>;
1526: def : InstRW<[WLat6, VecDFX, LSU, GroupAlone2], (instregex "VPKZ$")>;
1527: def : InstRW<[WLat1, VecDFX, FXb, LSU2, GroupAlone2], (instregex "VUPKZ$")>;
1528: def : InstRW<[WLat20, WLat20, VecDF2, FXb, GroupAlone],
1529:              (instregex "VCVB(G)?(Opt)?$")>;
1530: def : InstRW<[WLat15, WLat15, VecDF2, FXb, GroupAlone],
1531:              (instregex "VCVD(G)?$")>;
1532: def : InstRW<[WLat4, WLat4, VecDFX, NormalGr], (instregex "V(A|S)P$")>;
1533: def : InstRW<[WLat30, WLat30, VecDF2, GroupAlone], (instregex "VM(S)?P$")>;
1534: def : InstRW<[WLat30, WLat30, VecDF2, GroupAlone], (instregex "V(D|R)P$")>;
1535: def : InstRW<[WLat30, WLat30, VecDF2, GroupAlone], (instregex "VSDP$")>;
1536: def : InstRW<[WLat10, WLat10, VecDF2, NormalGr], (instregex "VSRP$")>;
1537: def : InstRW<[WLat4, WLat4, VecDFX, NormalGr], (instregex "VPSOP$")>;
1538: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "V(T|C)P$")>;
1539: 
1540: 
1541: // -------------------------------- System ---------------------------------- //
1542: 
1543: //===----------------------------------------------------------------------===//
1544: // System: Program-Status Word Instructions
1545: //===----------------------------------------------------------------------===//
1546: 
1547: def : InstRW<[WLat30, WLat30, MCD], (instregex "EPSW$")>;
1548: def : InstRW<[WLat20, GroupAlone3], (instregex "LPSW(E)?$")>;
1549: def : InstRW<[WLat3, FXa, GroupAlone], (instregex "IPK$")>;
1550: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SPKA$")>;
1551: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SSM$")>;
1552: def : InstRW<[WLat1, FXb, LSU, GroupAlone], (instregex "ST(N|O)SM$")>;
1553: def : InstRW<[WLat3, FXa, NormalGr], (instregex "IAC$")>;
1554: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SAC(F)?$")>;
1555: 
1556: //===----------------------------------------------------------------------===//
1557: // System: Control Register Instructions
1558: //===----------------------------------------------------------------------===//
1559: 
1560: def : InstRW<[WLat4LSU, WLat4LSU, LSU2, GroupAlone], (instregex "LCTL(G)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1561-1600 / 第 1561-1600 行
```tablegen
1561: def : InstRW<[WLat1, LSU5, FXb, GroupAlone2], (instregex "STCT(L|G)$")>;
1562: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "E(P|S)A(I)?R$")>;
1563: def : InstRW<[WLat30, MCD], (instregex "SSA(I)?R$")>;
1564: def : InstRW<[WLat30, MCD], (instregex "ESEA$")>;
1565: 
1566: //===----------------------------------------------------------------------===//
1567: // System: Prefix-Register Instructions
1568: //===----------------------------------------------------------------------===//
1569: 
1570: def : InstRW<[WLat30, MCD], (instregex "S(T)?PX$")>;
1571: 
1572: //===----------------------------------------------------------------------===//
1573: // System: Storage-Key and Real Memory Instructions
1574: //===----------------------------------------------------------------------===//
1575: 
1576: def : InstRW<[WLat30, MCD], (instregex "ISKE$")>;
1577: def : InstRW<[WLat30, MCD], (instregex "IVSK$")>;
1578: def : InstRW<[WLat30, MCD], (instregex "SSKE(Opt)?$")>;
1579: def : InstRW<[WLat30, MCD], (instregex "RRB(E|M)$")>;
1580: def : InstRW<[WLat30, MCD], (instregex "IRBM$")>;
1581: def : InstRW<[WLat30, MCD], (instregex "PFMF$")>;
1582: def : InstRW<[WLat30, WLat30, MCD], (instregex "TB$")>;
1583: def : InstRW<[WLat30, MCD], (instregex "PGIN$")>;
1584: def : InstRW<[WLat30, MCD], (instregex "PGOUT$")>;
1585: 
1586: //===----------------------------------------------------------------------===//
1587: // System: Dynamic-Address-Translation Instructions
1588: //===----------------------------------------------------------------------===//
1589: 
1590: def : InstRW<[WLat30, MCD], (instregex "IPTE(Opt)?(Opt)?$")>;
1591: def : InstRW<[WLat30, MCD], (instregex "IDTE(Opt)?$")>;
1592: def : InstRW<[WLat30, MCD], (instregex "CRDTE(Opt)?$")>;
1593: def : InstRW<[WLat30, MCD], (instregex "PTLB$")>;
1594: def : InstRW<[WLat30, WLat30, MCD], (instregex "CSP(G)?$")>;
1595: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "LPTEA$")>;
1596: def : InstRW<[WLat30, WLat30, MCD], (instregex "LRA(Y|G)?$")>;
1597: def : InstRW<[WLat30, MCD], (instregex "STRAG$")>;
1598: def : InstRW<[WLat30, MCD], (instregex "LURA(G)?$")>;
1599: def : InstRW<[WLat30, MCD], (instregex "STUR(A|G)$")>;
1600: def : InstRW<[WLat30, MCD], (instregex "TPROT$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1601-1640 / 第 1601-1640 行
```tablegen
1601: 
1602: //===----------------------------------------------------------------------===//
1603: // System: Memory-move Instructions
1604: //===----------------------------------------------------------------------===//
1605: 
1606: def : InstRW<[WLat4LSU, FXa2, FXb, LSU5, GroupAlone2], (instregex "MVC(K|P|S)$")>;
1607: def : InstRW<[WLat1, FXa, LSU5, GroupAlone2], (instregex "MVC(S|D)K$")>;
1608: def : InstRW<[WLat30, MCD], (instregex "MVCOS$")>;
1609: def : InstRW<[WLat30, MCD], (instregex "MVPG$")>;
1610: 
1611: //===----------------------------------------------------------------------===//
1612: // System: Address-Space Instructions
1613: //===----------------------------------------------------------------------===//
1614: 
1615: def : InstRW<[WLat30, MCD], (instregex "LASP$")>;
1616: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "PALB$")>;
1617: def : InstRW<[WLat30, MCD], (instregex "PC$")>;
1618: def : InstRW<[WLat30, MCD], (instregex "PR$")>;
1619: def : InstRW<[WLat30, MCD], (instregex "PT(I)?$")>;
1620: def : InstRW<[WLat30, MCD], (instregex "RP$")>;
1621: def : InstRW<[WLat30, MCD], (instregex "BS(G|A)$")>;
1622: def : InstRW<[WLat30, MCD], (instregex "TAR$")>;
1623: 
1624: //===----------------------------------------------------------------------===//
1625: // System: Linkage-Stack Instructions
1626: //===----------------------------------------------------------------------===//
1627: 
1628: def : InstRW<[WLat30, MCD], (instregex "BAKR$")>;
1629: def : InstRW<[WLat30, MCD], (instregex "EREG(G)?$")>;
1630: def : InstRW<[WLat30, WLat30, MCD], (instregex "(E|M)STA$")>;
1631: 
1632: //===----------------------------------------------------------------------===//
1633: // System: Time-Related Instructions
1634: //===----------------------------------------------------------------------===//
1635: 
1636: def : InstRW<[WLat30, MCD], (instregex "PTFF$")>;
1637: def : InstRW<[WLat30, MCD], (instregex "SCK(PF|C)?$")>;
1638: def : InstRW<[WLat1, LSU2, GroupAlone], (instregex "SPT$")>;
1639: def : InstRW<[WLat15, LSU3, FXa2, FXb, GroupAlone2], (instregex "STCK(F)?$")>;
1640: def : InstRW<[WLat20, LSU4, FXa2, FXb2, GroupAlone3], (instregex "STCKE$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1641-1680 / 第 1641-1680 行
```tablegen
1641: def : InstRW<[WLat30, MCD], (instregex "STCKC$")>;
1642: def : InstRW<[WLat1, LSU2, FXb, Cracked], (instregex "STPT$")>;
1643: 
1644: //===----------------------------------------------------------------------===//
1645: // System: CPU-Related Instructions
1646: //===----------------------------------------------------------------------===//
1647: 
1648: def : InstRW<[WLat30, MCD], (instregex "STAP$")>;
1649: def : InstRW<[WLat30, MCD], (instregex "STIDP$")>;
1650: def : InstRW<[WLat30, WLat30, MCD], (instregex "STSI$")>;
1651: def : InstRW<[WLat30, WLat30, MCD], (instregex "STFL(E)?$")>;
1652: def : InstRW<[WLat30, MCD], (instregex "ECAG$")>;
1653: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECTG$")>;
1654: def : InstRW<[WLat30, MCD], (instregex "PTF$")>;
1655: def : InstRW<[WLat30, MCD], (instregex "PCKMO$")>;
1656: 
1657: //===----------------------------------------------------------------------===//
1658: // System: Miscellaneous Instructions
1659: //===----------------------------------------------------------------------===//
1660: 
1661: def : InstRW<[WLat30, MCD], (instregex "SVC$")>;
1662: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "MC$")>;
1663: def : InstRW<[WLat30, MCD], (instregex "DIAG$")>;
1664: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TRAC(E|G)$")>;
1665: def : InstRW<[WLat30, MCD], (instregex "TRAP(2|4)$")>;
1666: def : InstRW<[WLat30, MCD], (instregex "SIG(P|A)$")>;
1667: def : InstRW<[WLat30, MCD], (instregex "SIE$")>;
1668: 
1669: //===----------------------------------------------------------------------===//
1670: // System: CPU-Measurement Facility Instructions
1671: //===----------------------------------------------------------------------===//
1672: 
1673: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LPP$")>;
1674: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECPGA$")>;
1675: def : InstRW<[WLat30, WLat30, MCD], (instregex "E(C|P)CTR$")>;
1676: def : InstRW<[WLat30, MCD], (instregex "LCCTL$")>;
1677: def : InstRW<[WLat30, MCD], (instregex "L(P|S)CTL$")>;
1678: def : InstRW<[WLat30, MCD], (instregex "Q(S|CTR)I$")>;
1679: def : InstRW<[WLat30, MCD], (instregex "S(C|P)CTR$")>;
1680: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1681-1700 / 第 1681-1700 行
```tablegen
1681: //===----------------------------------------------------------------------===//
1682: // System: I/O Instructions
1683: //===----------------------------------------------------------------------===//
1684: 
1685: def : InstRW<[WLat30, MCD], (instregex "(C|H|R|X)SCH$")>;
1686: def : InstRW<[WLat30, MCD], (instregex "(M|S|ST|T)SCH$")>;
1687: def : InstRW<[WLat30, MCD], (instregex "RCHP$")>;
1688: def : InstRW<[WLat30, MCD], (instregex "SCHM$")>;
1689: def : InstRW<[WLat30, MCD], (instregex "STC(PS|RW)$")>;
1690: def : InstRW<[WLat30, MCD], (instregex "TPE?I$")>;
1691: def : InstRW<[WLat30, MCD], (instregex "SAL$")>;
1692: 
1693: //===----------------------------------------------------------------------===//
1694: // NOPs
1695: //===----------------------------------------------------------------------===//
1696: 
1697: def : InstRW<[WLat1, FXb, NormalGr], (instregex "NOP(R)?(Opt)?$")>;
1698: def : InstRW<[WLat1, VBU, NormalGr], (instregex "J(G)?NOP$")>;
1699: }
1700: 
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
