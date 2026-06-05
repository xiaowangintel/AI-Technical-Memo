# SystemZScheduleZ13.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZScheduleZ13.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //-- SystemZScheduleZ13.td - SystemZ Scheduling Definitions ----*- tblgen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the machine model for Z13 to support instruction
  10: // scheduling and other instruction cost heuristics.
  11: //
  12: // Pseudos expanded right after isel do not need to be modelled here.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: def Z13Model : SchedMachineModel {
  17: 
  18:     let UnsupportedFeatures = Arch11UnsupportedFeatures.List;
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
  30: let SchedModel = Z13Model in  {
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
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `Z13Model`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `Z13Model` 等 TableGen 记录。

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
  76: def Z13_FXaUnit     : ProcResource<2>;
  77: def Z13_FXbUnit     : ProcResource<2>;
  78: def Z13_LSUnit      : ProcResource<2>;
  79: def Z13_VecUnit     : ProcResource<2>;
  80: def Z13_VecFPdUnit  : ProcResource<2> { let BufferSize = 1; /* blocking */ }
```
- **EN**: This block declares or refines TableGen records such as `Z13_FXaUnit`, `Z13_FXbUnit`, `Z13_LSUnit`, `Z13_VecUnit`, `Z13_VecFPdUnit`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `Z13_FXaUnit`, `Z13_FXbUnit`, `Z13_LSUnit`, `Z13_VecUnit`, `Z13_VecFPdUnit` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 81-120 / 第 81-120 行
```tablegen
  81: def Z13_VBUnit      : ProcResource<2>;
  82: def Z13_MCD         : ProcResource<1>;
  83: 
  84: // Subtarget specific definitions of scheduling resources.
  85: let NumMicroOps = 0 in {
  86:   def : WriteRes<FXa, [Z13_FXaUnit]>;
  87:   def : WriteRes<FXb, [Z13_FXbUnit]>;
  88:   def : WriteRes<LSU, [Z13_LSUnit]>;
  89:   def : WriteRes<VecBF,  [Z13_VecUnit]>;
  90:   def : WriteRes<VecDF,  [Z13_VecUnit]>;
  91:   def : WriteRes<VecDFX, [Z13_VecUnit]>;
  92:   def : WriteRes<VecMul,  [Z13_VecUnit]>;
  93:   def : WriteRes<VecStr,  [Z13_VecUnit]>;
  94:   def : WriteRes<VecXsPm, [Z13_VecUnit]>;
  95:   foreach Num = 2-5 in { let ReleaseAtCycles = [Num] in {
  96:     def : WriteRes<!cast<SchedWrite>("FXa"#Num), [Z13_FXaUnit]>;
  97:     def : WriteRes<!cast<SchedWrite>("FXb"#Num), [Z13_FXbUnit]>;
  98:     def : WriteRes<!cast<SchedWrite>("LSU"#Num), [Z13_LSUnit]>;
  99:     def : WriteRes<!cast<SchedWrite>("VecBF"#Num), [Z13_VecUnit]>;
 100:     def : WriteRes<!cast<SchedWrite>("VecDF"#Num), [Z13_VecUnit]>;
 101:     def : WriteRes<!cast<SchedWrite>("VecDFX"#Num), [Z13_VecUnit]>;
 102:     def : WriteRes<!cast<SchedWrite>("VecMul"#Num), [Z13_VecUnit]>;
 103:     def : WriteRes<!cast<SchedWrite>("VecStr"#Num), [Z13_VecUnit]>;
 104:     def : WriteRes<!cast<SchedWrite>("VecXsPm"#Num), [Z13_VecUnit]>;
 105:   }}
 106: 
 107:   def : WriteRes<VecFPd,  [Z13_VecFPdUnit]> { let ReleaseAtCycles = [30]; }
 108: 
 109:   def : WriteRes<VBU,     [Z13_VBUnit]>; // Virtual Branching Unit
 110: }
 111: 
 112: def : WriteRes<MCD, [Z13_MCD]> { let NumMicroOps = 3;
 113:                                  let BeginGroup  = 1;
 114:                                  let EndGroup    = 1; }
 115: 
 116: // -------------------------- INSTRUCTIONS ---------------------------------- //
 117: 
 118: // InstRW constructs have been used in order to preserve the
 119: // readability of the InstrInfo files.
 120: 
```
- **EN**: This block declares or refines TableGen records such as `Z13_VBUnit`, `Z13_MCD`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `Z13_VBUnit`, `Z13_MCD` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

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
 140: def : InstRW<[WLat1, FXa, EndGroup], (instregex "BRCT(G)?$")>;
 141: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BRCTH$")>;
 142: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BCT(G)?(R)?$")>;
 143: def : InstRW<[WLat1, FXa2, FXb2, GroupAlone2],
 144:              (instregex "B(R)?X(H|L).*$")>;
 145: 
 146: // Compare and branch
 147: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(L)?(G)?(I|R)J(Asm.*)?$")>;
 148: def : InstRW<[WLat1, FXb2, GroupAlone],
 149:              (instregex "C(L)?(G)?(I|R)B(Call|Return|Asm.*)?$")>;
 150: 
 151: //===----------------------------------------------------------------------===//
 152: // Trap instructions
 153: //===----------------------------------------------------------------------===//
 154: 
 155: // Trap
 156: def : InstRW<[WLat1, VBU, NormalGr], (instregex "(Cond)?Trap$")>;
 157: 
 158: // Compare and trap
 159: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(G)?(I|R)T(Asm.*)?$")>;
 160: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CL(G)?RT(Asm.*)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 161-200 / 第 161-200 行
```tablegen
 161: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CL(F|G)IT(Asm.*)?$")>;
 162: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "CL(G)?T(Asm.*)?$")>;
 163: 
 164: //===----------------------------------------------------------------------===//
 165: // Call and return instructions
 166: //===----------------------------------------------------------------------===//
 167: 
 168: // Call
 169: def : InstRW<[WLat1, VBU, FXa2, GroupAlone], (instregex "(Call)?BRAS$")>;
 170: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "(Call)?BRASL(_XPLINK64)?$")>;
 171: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "(Call)?BAS(R)?(_XPLINK64|_STACKEXT)?$")>;
 172: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "TLS_(G|L)DCALL$")>;
 173: 
 174: // Return
 175: def : InstRW<[WLat1, FXb, EndGroup], (instregex "Return(_XPLINK)?$")>;
 176: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CondReturn(_XPLINK)?$")>;
 177: 
 178: //===----------------------------------------------------------------------===//
 179: // Move instructions
 180: //===----------------------------------------------------------------------===//
 181: 
 182: // Moves
 183: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "MV(G|H)?HI$")>;
 184: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "MVI(Y)?$")>;
 185: 
 186: // Move character
 187: def : InstRW<[WLat1, FXb, LSU3, GroupAlone], (instregex "MVC$")>;
 188: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "MVCL(E|U)?$")>;
 189: 
 190: // Pseudo -> reg move
 191: def : InstRW<[WLat1, FXa, NormalGr], (instregex "COPY(_TO_REGCLASS)?$")>;
 192: def : InstRW<[WLat1, FXa, NormalGr], (instregex "EXTRACT_SUBREG$")>;
 193: def : InstRW<[WLat1, FXa, NormalGr], (instregex "INSERT_SUBREG$")>;
 194: def : InstRW<[WLat1, FXa, NormalGr], (instregex "REG_SEQUENCE$")>;
 195: 
 196: // Loads
 197: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L(Y|FH|RL|Mux)?$")>;
 198: def : InstRW<[LSULatency, LSULatency, LSU, NormalGr], (instregex "LCBB$")>;
 199: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LG(RL)?$")>;
 200: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "L128$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 201-240 / 第 201-240 行
```tablegen
 201: 
 202: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLIH(F|H|L)$")>;
 203: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLIL(F|H|L)$")>;
 204: 
 205: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LG(F|H)I$")>;
 206: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LHI(Mux)?$")>;
 207: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LR$")>;
 208: 
 209: // Load and zero rightmost byte
 210: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LZR(F|G)$")>;
 211: 
 212: // Load and trap
 213: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "L(FH|G)?AT$")>;
 214: 
 215: // Load and test
 216: def : InstRW<[WLat1LSU, WLat1LSU, LSU, FXa, NormalGr], (instregex "LT(G)?$")>;
 217: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LT(G)?R$")>;
 218: 
 219: // Stores
 220: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STG(RL)?$")>;
 221: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST128$")>;
 222: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST(Y|FH|RL|Mux)?$")>;
 223: 
 224: // String moves.
 225: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "MVST$")>;
 226: 
 227: //===----------------------------------------------------------------------===//
 228: // Conditional move instructions
 229: //===----------------------------------------------------------------------===//
 230: 
 231: def : InstRW<[WLat2, FXa, NormalGr], (instregex "LOCRMux$")>;
 232: def : InstRW<[WLat2, FXa, NormalGr], (instregex "LOC(G|FH)?R(Asm.*)?$")>;
 233: def : InstRW<[WLat2, FXa, NormalGr], (instregex "LOC(G|H)?HI(Mux|(Asm.*))?$")>;
 234: def : InstRW<[WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 235:              (instregex "LOC(G|FH|Mux)?(Asm.*)?$")>;
 236: def : InstRW<[WLat1, FXb, LSU, NormalGr],
 237:              (instregex "STOC(G|FH|Mux)?(Asm.*)?$")>;
 238: 
 239: //===----------------------------------------------------------------------===//
 240: // Sign extensions
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 241-280 / 第 241-280 行
```tablegen
 241: //===----------------------------------------------------------------------===//
 242: 
 243: def : InstRW<[WLat1, FXa, NormalGr], (instregex "L(B|H|G)R$")>;
 244: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LG(B|H|F)R$")>;
 245: 
 246: def : InstRW<[WLat1LSU, WLat1LSU, FXa, LSU, NormalGr], (instregex "LTGF$")>;
 247: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LTGFR$")>;
 248: 
 249: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LB(H|Mux)?$")>;
 250: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LH(Y)?$")>;
 251: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LH(H|Mux|RL)$")>;
 252: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LG(B|H|F)$")>;
 253: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LG(H|F)RL$")>;
 254: 
 255: //===----------------------------------------------------------------------===//
 256: // Zero extensions
 257: //===----------------------------------------------------------------------===//
 258: 
 259: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLCR(Mux)?$")>;
 260: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLHR(Mux)?$")>;
 261: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LLG(C|H|F|T)R$")>;
 262: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLC(Mux)?$")>;
 263: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLH(Mux)?$")>;
 264: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LL(C|H)H$")>;
 265: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLHRL$")>;
 266: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLG(C|H|F|T|HRL|FRL)$")>;
 267: 
 268: // Load and zero rightmost byte
 269: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLZRGF$")>;
 270: 
 271: // Load and trap
 272: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "LLG(F|T)?AT$")>;
 273: 
 274: //===----------------------------------------------------------------------===//
 275: // Truncations
 276: //===----------------------------------------------------------------------===//
 277: 
 278: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STC(H|Y|Mux)?$")>;
 279: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STH(H|Y|RL|Mux)?$")>;
 280: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STCM(H|Y)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 281-320 / 第 281-320 行
```tablegen
 281: 
 282: //===----------------------------------------------------------------------===//
 283: // Multi-register moves
 284: //===----------------------------------------------------------------------===//
 285: 
 286: // Load multiple (estimated average of 5 ops)
 287: def : InstRW<[WLat10, WLat10, LSU5, GroupAlone], (instregex "LM(H|Y|G)?$")>;
 288: 
 289: // Load multiple disjoint
 290: def : InstRW<[WLat30, WLat30, MCD], (instregex "LMD$")>;
 291: 
 292: // Store multiple
 293: def : InstRW<[WLat1, LSU2, FXb3, GroupAlone], (instregex "STM(G|H|Y)?$")>;
 294: 
 295: //===----------------------------------------------------------------------===//
 296: // Byte swaps
 297: //===----------------------------------------------------------------------===//
 298: 
 299: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LRV(G)?R$")>;
 300: def : InstRW<[WLat1LSU, FXa, LSU, NormalGr], (instregex "LRV(G|H)?$")>;
 301: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STRV(G|H)?$")>;
 302: def : InstRW<[WLat30, MCD], (instregex "MVCIN$")>;
 303: 
 304: //===----------------------------------------------------------------------===//
 305: // Load address instructions
 306: //===----------------------------------------------------------------------===//
 307: 
 308: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LA(Y|RL)?$")>;
 309: 
 310: // Load the Global Offset Table address ( -> larl )
 311: def : InstRW<[WLat1, FXa, NormalGr], (instregex "GOT$")>;
 312: 
 313: //===----------------------------------------------------------------------===//
 314: // Absolute and Negation
 315: //===----------------------------------------------------------------------===//
 316: 
 317: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "LP(G)?R$")>;
 318: def : InstRW<[WLat3, WLat3, FXa2, Cracked], (instregex "L(N|P)GFR$")>;
 319: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "LN(R|GR)$")>;
 320: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LC(R|GR)$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 321-360 / 第 321-360 行
```tablegen
 321: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "LCGFR$")>;
 322: 
 323: //===----------------------------------------------------------------------===//
 324: // Insertion
 325: //===----------------------------------------------------------------------===//
 326: 
 327: def : InstRW<[WLat1LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "IC(Y)?$")>;
 328: def : InstRW<[WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 329:              (instregex "IC32(Y)?$")>;
 330: def : InstRW<[WLat1LSU, RegReadAdv, WLat1LSU, FXa, LSU, NormalGr],
 331:              (instregex "ICM(H|Y)?$")>;
 332: def : InstRW<[WLat1, FXa, NormalGr], (instregex "II(F|H|L)Mux$")>;
 333: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHF(64)?$")>;
 334: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHH(64)?$")>;
 335: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHL(64)?$")>;
 336: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILF(64)?$")>;
 337: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILH(64)?$")>;
 338: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILL(64)?$")>;
 339: 
 340: //===----------------------------------------------------------------------===//
 341: // Addition
 342: //===----------------------------------------------------------------------===//
 343: 
 344: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 345:              (instregex "A(Y)?$")>;
 346: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 347:              (instregex "AH(Y)?$")>;
 348: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AIH$")>;
 349: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AFI(Mux)?$")>;
 350: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 351:              (instregex "AG$")>;
 352: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGFI$")>;
 353: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGHI(K)?$")>;
 354: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGR(K)?$")>;
 355: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AHI(K)?$")>;
 356: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AHIMux(K)?$")>;
 357: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 358:              (instregex "AL(Y)?$")>;
 359: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AL(FI|HSIK)$")>;
 360: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 361-400 / 第 361-400 行
```tablegen
 361:              (instregex "ALG(F)?$")>;
 362: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGHSIK$")>;
 363: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGF(I|R)$")>;
 364: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGR(K)?$")>;
 365: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALR(K)?$")>;
 366: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AR(K)?$")>;
 367: def : InstRW<[WLat1, FXa, NormalGr], (instregex "A(L)?HHHR$")>;
 368: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "A(L)?HHLR$")>;
 369: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALSIH(N)?$")>;
 370: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "A(L)?(G)?SI$")>;
 371: 
 372: // Logical addition with carry
 373: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, GroupAlone],
 374:              (instregex "ALC(G)?$")>;
 375: def : InstRW<[WLat2, WLat2, FXa, GroupAlone], (instregex "ALC(G)?R$")>;
 376: 
 377: // Add with sign extension (32 -> 64)
 378: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 379:              (instregex "AGF$")>;
 380: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "AGFR$")>;
 381: 
 382: //===----------------------------------------------------------------------===//
 383: // Subtraction
 384: //===----------------------------------------------------------------------===//
 385: 
 386: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 387:              (instregex "S(G|Y)?$")>;
 388: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 389:              (instregex "SH(Y)?$")>;
 390: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SGR(K)?$")>;
 391: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLFI$")>;
 392: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 393:              (instregex "SL(G|GF|Y)?$")>;
 394: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLGF(I|R)$")>;
 395: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLGR(K)?$")>;
 396: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLR(K)?$")>;
 397: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SR(K)?$")>;
 398: def : InstRW<[WLat1, FXa, NormalGr], (instregex "S(L)?HHHR$")>;
 399: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "S(L)?HHLR$")>;
 400: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 401-440 / 第 401-440 行
```tablegen
 401: // Subtraction with borrow
 402: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, GroupAlone],
 403:              (instregex "SLB(G)?$")>;
 404: def : InstRW<[WLat2, WLat2, FXa, GroupAlone], (instregex "SLB(G)?R$")>;
 405: 
 406: // Subtraction with sign extension (32 -> 64)
 407: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 408:              (instregex "SGF$")>;
 409: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "SGFR$")>;
 410: 
 411: //===----------------------------------------------------------------------===//
 412: // AND
 413: //===----------------------------------------------------------------------===//
 414: 
 415: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 416:              (instregex "N(G|Y)?$")>;
 417: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NGR(K)?$")>;
 418: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NI(FMux|HMux|LMux)$")>;
 419: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "NI(Y)?$")>;
 420: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHF(64)?$")>;
 421: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHH(64)?$")>;
 422: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHL(64)?$")>;
 423: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILF(64)?$")>;
 424: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILH(64)?$")>;
 425: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILL(64)?$")>;
 426: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NR(K)?$")>;
 427: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "NC$")>;
 428: 
 429: //===----------------------------------------------------------------------===//
 430: // OR
 431: //===----------------------------------------------------------------------===//
 432: 
 433: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 434:              (instregex "O(G|Y)?$")>;
 435: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OGR(K)?$")>;
 436: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "OI(Y)?$")>;
 437: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OI(FMux|HMux|LMux)$")>;
 438: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHF(64)?$")>;
 439: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHH(64)?$")>;
 440: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHL(64)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 441-480 / 第 441-480 行
```tablegen
 441: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILF(64)?$")>;
 442: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILH(64)?$")>;
 443: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILL(64)?$")>;
 444: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OR(K)?$")>;
 445: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "OC$")>;
 446: 
 447: //===----------------------------------------------------------------------===//
 448: // XOR
 449: //===----------------------------------------------------------------------===//
 450: 
 451: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 452:              (instregex "X(G|Y)?$")>;
 453: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "XI(Y)?$")>;
 454: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XIFMux$")>;
 455: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XGR(K)?$")>;
 456: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XIHF(64)?$")>;
 457: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XILF(64)?$")>;
 458: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XR(K)?$")>;
 459: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "XC$")>;
 460: 
 461: //===----------------------------------------------------------------------===//
 462: // Multiplication
 463: //===----------------------------------------------------------------------===//
 464: 
 465: def : InstRW<[WLat6LSU, RegReadAdv, FXa, LSU, NormalGr],
 466:              (instregex "MS(GF|Y)?$")>;
 467: def : InstRW<[WLat6, FXa, NormalGr], (instregex "MS(R|FI)$")>;
 468: def : InstRW<[WLat8LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "MSG$")>;
 469: def : InstRW<[WLat8, FXa, NormalGr], (instregex "MSGR$")>;
 470: def : InstRW<[WLat6, FXa, NormalGr], (instregex "MSGF(I|R)$")>;
 471: def : InstRW<[WLat11LSU, RegReadAdv, FXa2, LSU, GroupAlone],
 472:              (instregex "MLG$")>;
 473: def : InstRW<[WLat9, FXa2, GroupAlone], (instregex "MLGR$")>;
 474: def : InstRW<[WLat5, FXa, NormalGr], (instregex "MGHI$")>;
 475: def : InstRW<[WLat5, FXa, NormalGr], (instregex "MHI$")>;
 476: def : InstRW<[WLat5LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "MH(Y)?$")>;
 477: def : InstRW<[WLat7, FXa2, GroupAlone], (instregex "M(L)?R$")>;
 478: def : InstRW<[WLat7LSU, RegReadAdv, FXa2, LSU, GroupAlone],
 479:              (instregex "M(FY|L)?$")>;
 480: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: //===----------------------------------------------------------------------===//
 482: // Division and remainder
 483: //===----------------------------------------------------------------------===//
 484: 
 485: def : InstRW<[WLat20, FXa4, GroupAlone], (instregex "DR$")>;
 486: def : InstRW<[WLat30, RegReadAdv, FXa4, LSU, GroupAlone2], (instregex "D$")>;
 487: def : InstRW<[WLat30, FXa2, GroupAlone], (instregex "DSG(F)?R$")>;
 488: def : InstRW<[WLat30, RegReadAdv, FXa2, LSU, GroupAlone2],
 489:              (instregex "DSG(F)?$")>;
 490: def : InstRW<[WLat20, FXa4, GroupAlone], (instregex "DLR$")>;
 491: def : InstRW<[WLat30, FXa4, GroupAlone], (instregex "DLGR$")>;
 492: def : InstRW<[WLat30, RegReadAdv, FXa4, LSU, GroupAlone2], (instregex "DL(G)?$")>;
 493: 
 494: //===----------------------------------------------------------------------===//
 495: // Shifts
 496: //===----------------------------------------------------------------------===//
 497: 
 498: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLL(G|K)?$")>;
 499: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SRL(G|K)?$")>;
 500: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SRA(G|K)?$")>;
 501: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLA(G|K)?$")>;
 502: def : InstRW<[WLat5LSU, WLat5LSU, FXa4, LSU, GroupAlone2],
 503:              (instregex "S(L|R)D(A|L)$")>;
 504: 
 505: // Rotate
 506: def : InstRW<[WLat2LSU, FXa, LSU, NormalGr], (instregex "RLL(G)?$")>;
 507: 
 508: // Rotate and insert
 509: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBH(G|H|L)(Opt)?$")>;
 510: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBL(G|H|L)(Opt)?$")>;
 511: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBG(N|32)?(Z)?(Opt)?$")>;
 512: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBMux$")>;
 513: 
 514: // Rotate and Select
 515: def : InstRW<[WLat3, WLat3, FXa2, Cracked], (instregex "R(N|O|X)SBG(Opt)?$")>;
 516: 
 517: //===----------------------------------------------------------------------===//
 518: // Comparison
 519: //===----------------------------------------------------------------------===//
 520: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 521-560 / 第 521-560 行
```tablegen
 521: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr],
 522:              (instregex "C(G|Y|Mux)?$")>;
 523: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CRL$")>;
 524: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(F|H)I(Mux)?$")>;
 525: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CG(F|H)I$")>;
 526: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CG(HSI|RL)$")>;
 527: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(G)?R$")>;
 528: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CIH$")>;
 529: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CHF$")>;
 530: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CHSI$")>;
 531: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr],
 532:              (instregex "CL(Y|Mux)?$")>;
 533: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLFHSI$")>;
 534: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLFI(Mux)?$")>;
 535: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLG$")>;
 536: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLG(HRL|HSI)$")>;
 537: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLGF$")>;
 538: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLGFRL$")>;
 539: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLGF(I|R)$")>;
 540: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLGR$")>;
 541: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLGRL$")>;
 542: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLHF$")>;
 543: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLH(RL|HSI)$")>;
 544: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLIH$")>;
 545: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLI(Y)?$")>;
 546: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLR$")>;
 547: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLRL$")>;
 548: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(L)?HHR$")>;
 549: def : InstRW<[WLat2, FXb, NormalGr], (instregex "C(L)?HLR$")>;
 550: 
 551: // Compare halfword
 552: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CH(Y)?$")>;
 553: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CHRL$")>;
 554: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CGH$")>;
 555: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CGHRL$")>;
 556: def : InstRW<[WLat2LSU, FXa, FXb, LSU, Cracked], (instregex "CHHSI$")>;
 557: 
 558: // Compare with sign extension (32 -> 64)
 559: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CGF$")>;
 560: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CGFRL$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: def : InstRW<[WLat2, FXb, NormalGr], (instregex "CGFR$")>;
 562: 
 563: // Compare logical character
 564: def : InstRW<[WLat6, FXb, LSU2, Cracked], (instregex "CLC$")>;
 565: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLCL(E|U)?$")>;
 566: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLST$")>;
 567: 
 568: // Test under mask
 569: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "TM(Y)?$")>;
 570: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TM(H|L)Mux$")>;
 571: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMHH(64)?$")>;
 572: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMHL(64)?$")>;
 573: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMLH(64)?$")>;
 574: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMLL(64)?$")>;
 575: 
 576: // Compare logical characters under mask
 577: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr],
 578:              (instregex "CLM(H|Y)?$")>;
 579: 
 580: //===----------------------------------------------------------------------===//
 581: // Prefetch and execution hint
 582: //===----------------------------------------------------------------------===//
 583: 
 584: def : InstRW<[WLat1, LSU, NormalGr], (instregex "PFD(RL)?$")>;
 585: def : InstRW<[WLat1, FXb, NormalGr], (instregex "BPP$")>;
 586: def : InstRW<[FXb, EndGroup], (instregex "BPRP$")>;
 587: def : InstRW<[WLat1, FXb, NormalGr], (instregex "NIAI$")>;
 588: 
 589: //===----------------------------------------------------------------------===//
 590: // Atomic operations
 591: //===----------------------------------------------------------------------===//
 592: 
 593: def : InstRW<[WLat1, FXb, EndGroup], (instregex "Serialize$")>;
 594: 
 595: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAA(G)?$")>;
 596: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAAL(G)?$")>;
 597: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAN(G)?$")>;
 598: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAO(G)?$")>;
 599: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAX(G)?$")>;
 600: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 601-640 / 第 601-640 行
```tablegen
 601: // Test and set
 602: def : InstRW<[WLat2LSU, FXb, LSU, EndGroup], (instregex "TS$")>;
 603: 
 604: // Compare and swap
 605: def : InstRW<[WLat3LSU, WLat3LSU, FXa, FXb, LSU, GroupAlone],
 606:              (instregex "CS(G|Y)?$")>;
 607: 
 608: // Compare double and swap
 609: def : InstRW<[WLat6LSU, WLat6LSU, FXa3, FXb2, LSU, GroupAlone2],
 610:              (instregex "CDS(Y)?$")>;
 611: def : InstRW<[WLat15, WLat15, FXa2, FXb4, LSU3, GroupAlone3],
 612:              (instregex "CDSG$")>;
 613: 
 614: // Compare and swap and store
 615: def : InstRW<[WLat30, MCD], (instregex "CSST$")>;
 616: 
 617: // Perform locked operation
 618: def : InstRW<[WLat30, MCD], (instregex "PLO$")>;
 619: 
 620: // Load/store pair from/to quadword
 621: def : InstRW<[WLat4LSU, LSU2, GroupAlone], (instregex "LPQ$")>;
 622: def : InstRW<[WLat1, FXb2, LSU, GroupAlone], (instregex "STPQ$")>;
 623: 
 624: // Load pair disjoint
 625: def : InstRW<[WLat1LSU, WLat1LSU, LSU2, GroupAlone], (instregex "LPD(G)?$")>;
 626: 
 627: //===----------------------------------------------------------------------===//
 628: // Translate and convert
 629: //===----------------------------------------------------------------------===//
 630: 
 631: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "TR$")>;
 632: def : InstRW<[WLat30, WLat30, WLat30, FXa3, LSU2, GroupAlone2],
 633:              (instregex "TRT$")>;
 634: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRTR$")>;
 635: def : InstRW<[WLat30, WLat30, MCD], (instregex "TRE$")>;
 636: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRT(R)?E(Opt)?$")>;
 637: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TR(T|O)(T|O)(Opt)?$")>;
 638: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 639:              (instregex "CU(12|14|21|24|41|42)(Opt)?$")>;
 640: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "(CUUTF|CUTFU)(Opt)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 641-680 / 第 641-680 行
```tablegen
 641: 
 642: //===----------------------------------------------------------------------===//
 643: // Message-security assist
 644: //===----------------------------------------------------------------------===//
 645: 
 646: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD],
 647:              (instregex "KM(C|F|O|CTR)?$")>;
 648: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 649:              (instregex "(KIMD|KLMD|KMAC|PCC|PPNO)$")>;
 650: 
 651: //===----------------------------------------------------------------------===//
 652: // Decimal arithmetic
 653: //===----------------------------------------------------------------------===//
 654: 
 655: def : InstRW<[WLat30, RegReadAdv, FXb, VecDF2, LSU2, GroupAlone2],
 656:              (instregex "CVBG$")>;
 657: def : InstRW<[WLat30, RegReadAdv, FXb, VecDF, LSU, GroupAlone2],
 658:              (instregex "CVB(Y)?$")>;
 659: def : InstRW<[WLat1, FXb3, VecDF4, LSU, GroupAlone3], (instregex "CVDG$")>;
 660: def : InstRW<[WLat1, FXb2, VecDF, LSU, GroupAlone2], (instregex "CVD(Y)?$")>;
 661: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "MV(N|O|Z)$")>;
 662: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "(PACK|PKA|PKU)$")>;
 663: def : InstRW<[WLat12, LSU5, GroupAlone], (instregex "UNPK(A|U)$")>;
 664: def : InstRW<[WLat1, FXb, LSU2, Cracked], (instregex "UNPK$")>;
 665: 
 666: def : InstRW<[WLat5LSU, FXb, VecDFX, LSU3, GroupAlone2],
 667:              (instregex "(A|S|ZA)P$")>;
 668: def : InstRW<[WLat1, FXb, VecDFX4, LSU3, GroupAlone2], (instregex "(M|D)P$")>;
 669: def : InstRW<[WLat15, FXb, VecDFX2, LSU2, GroupAlone3], (instregex "SRP$")>;
 670: def : InstRW<[WLat8, VecDFX, LSU, LSU, GroupAlone], (instregex "CP$")>;
 671: def : InstRW<[WLat3LSU, VecDFX, LSU, Cracked], (instregex "TP$")>;
 672: def : InstRW<[WLat30, MCD], (instregex "ED(MK)?$")>;
 673: 
 674: //===----------------------------------------------------------------------===//
 675: // Access registers
 676: //===----------------------------------------------------------------------===//
 677: 
 678: // Extract/set/copy access register
 679: def : InstRW<[WLat3, LSU, NormalGr], (instregex "(EAR|SAR|CPYA)$")>;
 680: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 681-720 / 第 681-720 行
```tablegen
 681: // Load address extended
 682: def : InstRW<[WLat5, LSU, FXa, Cracked], (instregex "LAE(Y)?$")>;
 683: 
 684: // Load/store access multiple (not modeled precisely)
 685: def : InstRW<[WLat20, WLat20, LSU5, GroupAlone], (instregex "LAM(Y)?$")>;
 686: def : InstRW<[WLat1, LSU5, FXb, GroupAlone2], (instregex "STAM(Y)?$")>;
 687: 
 688: //===----------------------------------------------------------------------===//
 689: // Program mask and addressing mode
 690: //===----------------------------------------------------------------------===//
 691: 
 692: // Insert Program Mask
 693: def : InstRW<[WLat3, FXa, EndGroup], (instregex "IPM$")>;
 694: 
 695: // Set Program Mask
 696: def : InstRW<[WLat3, LSU, EndGroup], (instregex "SPM$")>;
 697: 
 698: // Branch and link
 699: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "BAL(R)?$")>;
 700: 
 701: // Test addressing mode
 702: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TAM$")>;
 703: 
 704: // Set addressing mode
 705: def : InstRW<[WLat1, FXb, EndGroup], (instregex "SAM(24|31|64)$")>;
 706: 
 707: // Branch (and save) and set mode.
 708: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BSM$")>;
 709: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "BASSM$")>;
 710: 
 711: //===----------------------------------------------------------------------===//
 712: // Transactional execution
 713: //===----------------------------------------------------------------------===//
 714: 
 715: // Transaction begin
 716: def : InstRW<[WLat9, LSU2, FXb5, GroupAlone2], (instregex "TBEGIN(C)?$")>;
 717: 
 718: // Transaction end
 719: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "TEND$")>;
 720: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: // Transaction abort
 722: def : InstRW<[WLat30, MCD], (instregex "TABORT$")>;
 723: 
 724: // Extract Transaction Nesting Depth
 725: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ETND$")>;
 726: 
 727: // Nontransactional store
 728: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "NTSTG$")>;
 729: 
 730: //===----------------------------------------------------------------------===//
 731: // Processor assist
 732: //===----------------------------------------------------------------------===//
 733: 
 734: def : InstRW<[WLat30, MCD], (instregex "PPA$")>;
 735: 
 736: //===----------------------------------------------------------------------===//
 737: // Miscellaneous Instructions.
 738: //===----------------------------------------------------------------------===//
 739: 
 740: // Find leftmost one
 741: def : InstRW<[WLat7, WLat7, FXa2, GroupAlone], (instregex "FLOGR$")>;
 742: 
 743: // Population count
 744: def : InstRW<[WLat3, WLat3, FXa, NormalGr], (instregex "POPCNT$")>;
 745: 
 746: // String instructions
 747: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "SRST(U)?$")>;
 748: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CUSE$")>;
 749: 
 750: // Various complex instructions
 751: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CFC$")>;
 752: def : InstRW<[WLat30, WLat30, WLat30, WLat30, WLat30, WLat30, MCD],
 753:              (instregex "UPT$")>;
 754: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CKSM$")>;
 755: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CMPSC$")>;
 756: 
 757: // Execute
 758: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "EX(RL)?$")>;
 759: 
 760: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 761-800 / 第 761-800 行
```tablegen
 761: // .insn directive instructions
 762: //===----------------------------------------------------------------------===//
 763: 
 764: // An "empty" sched-class will be assigned instead of the "invalid sched-class".
 765: // getNumDecoderSlots() will then return 1 instead of 0.
 766: def : InstRW<[], (instregex "Insn.*")>;
 767: 
 768: 
 769: // ----------------------------- Floating point ----------------------------- //
 770: 
 771: //===----------------------------------------------------------------------===//
 772: // FP: Move instructions
 773: //===----------------------------------------------------------------------===//
 774: 
 775: // Load zero
 776: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LZ(DR|ER|ER_16)$")>;
 777: def : InstRW<[WLat2, FXb2, Cracked], (instregex "LZXR$")>;
 778: 
 779: // Load
 780: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "LER(16)?$")>;
 781: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LD(R|R16|R32|GR)$")>;
 782: def : InstRW<[WLat3, FXb, NormalGr], (instregex "LGDR$")>;
 783: def : InstRW<[WLat2, FXb2, GroupAlone], (instregex "LXR$")>;
 784: 
 785: // Load and Test
 786: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "LT(E|D)BR$")>;
 787: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXBR$")>;
 788: 
 789: // Copy sign
 790: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "CPSDR(d|s|h)(d|s|h)$")>;
 791: 
 792: //===----------------------------------------------------------------------===//
 793: // FP: Load instructions
 794: //===----------------------------------------------------------------------===//
 795: 
 796: def : InstRW<[WLat2LSU, VecXsPm, LSU, NormalGr], (instregex "L(E16|E)(Y)?$")>;
 797: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LD(Y|E32)?$")>;
 798: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LX$")>;
 799: 
 800: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: // FP: Store instructions
 802: //===----------------------------------------------------------------------===//
 803: 
 804: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST(E16|E|D)(Y)?$")>;
 805: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STX$")>;
 806: 
 807: //===----------------------------------------------------------------------===//
 808: // FP: Conversion instructions
 809: //===----------------------------------------------------------------------===//
 810: 
 811: // Load rounded
 812: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "LEDBR(A)?$")>;
 813: def : InstRW<[WLat9, VecDF2, NormalGr], (instregex "L(E|D)XBR(A)?$")>;
 814: 
 815: // Load lengthened
 816: def : InstRW<[WLat7LSU, VecBF, LSU, NormalGr], (instregex "LDEB$")>;
 817: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "LDEBR$")>;
 818: def : InstRW<[WLat8LSU, VecBF4, LSU, GroupAlone], (instregex "LX(E|D)B$")>;
 819: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "LX(E|D)BR$")>;
 820: 
 821: // Convert from fixed / logical
 822: def : InstRW<[WLat8, FXb, VecBF, Cracked], (instregex "C(E|D)(F|G)BR(A)?$")>;
 823: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)BR(A)?$")>;
 824: def : InstRW<[WLat8, FXb, VecBF, Cracked], (instregex "C(E|D)L(F|G)BR$")>;
 825: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CXL(F|G)BR$")>;
 826: 
 827: // Convert to fixed / logical
 828: def : InstRW<[WLat10, WLat10, FXb, VecBF, Cracked],
 829:              (instregex "C(F|G)(E|D)BR(A)?$")>;
 830: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked],
 831:              (instregex "C(F|G)XBR(A)?$")>;
 832: def : InstRW<[WLat10, WLat10, FXb, VecBF, GroupAlone], (instregex "CLFEBR$")>;
 833: def : InstRW<[WLat10, WLat10, FXb, VecBF, Cracked], (instregex "CLFDBR$")>;
 834: def : InstRW<[WLat10, WLat10, FXb, VecBF, Cracked], (instregex "CLG(E|D)BR$")>;
 835: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked], (instregex "CL(F|G)XBR$")>;
 836: 
 837: //===----------------------------------------------------------------------===//
 838: // FP: Unary arithmetic
 839: //===----------------------------------------------------------------------===//
 840: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: // Load Complement / Negative / Positive
 842: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "L(C|N|P)(E|D)BR$")>;
 843: def : InstRW<[WLat1, FXb, NormalGr], (instregex "L(C|N|P)DFR(_32|_16)?$")>;
 844: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "L(C|N|P)XBR$")>;
 845: 
 846: // Square root
 847: def : InstRW<[WLat30, VecFPd, LSU, NormalGr], (instregex "SQ(E|D)B$")>;
 848: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "SQ(E|D)BR$")>;
 849: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "SQXBR$")>;
 850: 
 851: // Load FP integer
 852: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "FI(E|D)BR(A)?$")>;
 853: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXBR(A)?$")>;
 854: 
 855: //===----------------------------------------------------------------------===//
 856: // FP: Binary arithmetic
 857: //===----------------------------------------------------------------------===//
 858: 
 859: // Addition
 860: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
 861:              (instregex "A(E|D)B$")>;
 862: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "A(E|D)BR$")>;
 863: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXBR$")>;
 864: 
 865: // Subtraction
 866: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
 867:              (instregex "S(E|D)B$")>;
 868: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "S(E|D)BR$")>;
 869: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXBR$")>;
 870: 
 871: // Multiply
 872: def : InstRW<[WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
 873:              (instregex "M(D|DE|EE)B$")>;
 874: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "M(D|DE|EE)BR$")>;
 875: def : InstRW<[WLat8LSU, RegReadAdv, VecBF4, LSU, GroupAlone],
 876:              (instregex "MXDB$")>;
 877: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "MXDBR$")>;
 878: def : InstRW<[WLat20, VecDF4, GroupAlone], (instregex "MXBR$")>;
 879: 
 880: // Multiply and add / subtract
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 881-920 / 第 881-920 行
```tablegen
 881: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
 882:              (instregex "M(A|S)EB$")>;
 883: def : InstRW<[WLat7, VecBF, GroupAlone], (instregex "M(A|S)EBR$")>;
 884: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
 885:              (instregex "M(A|S)DB$")>;
 886: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "M(A|S)DBR$")>;
 887: 
 888: // Division
 889: def : InstRW<[WLat30, RegReadAdv, VecFPd, LSU, NormalGr],
 890:              (instregex "D(E|D)B$")>;
 891: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "D(E|D)BR$")>;
 892: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "DXBR$")>;
 893: 
 894: // Divide to integer
 895: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "DI(E|D)BR$")>;
 896: 
 897: //===----------------------------------------------------------------------===//
 898: // FP: Comparisons
 899: //===----------------------------------------------------------------------===//
 900: 
 901: // Compare
 902: def : InstRW<[WLat3LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
 903:              (instregex "(K|C)(E|D)B$")>;
 904: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "(K|C)(E|D)BR$")>;
 905: def : InstRW<[WLat9, VecDF2, GroupAlone], (instregex "(K|C)XBR$")>;
 906: 
 907: // Test Data Class
 908: def : InstRW<[WLat5, LSU, VecXsPm, NormalGr], (instregex "TC(E|D)B$")>;
 909: def : InstRW<[WLat10, LSU, VecDF4, GroupAlone], (instregex "TCXB$")>;
 910: 
 911: //===----------------------------------------------------------------------===//
 912: // FP: Floating-point control register instructions
 913: //===----------------------------------------------------------------------===//
 914: 
 915: def : InstRW<[WLat4, FXa, LSU, GroupAlone], (instregex "EFPC$")>;
 916: def : InstRW<[WLat1, FXb, LSU, GroupAlone], (instregex "STFPC$")>;
 917: def : InstRW<[WLat3, LSU, GroupAlone], (instregex "SFPC$")>;
 918: def : InstRW<[WLat3LSU, LSU2, GroupAlone], (instregex "LFPC$")>;
 919: def : InstRW<[WLat30, MCD], (instregex "SFASR$")>;
 920: def : InstRW<[WLat30, MCD], (instregex "LFAS$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: def : InstRW<[WLat3, FXb, GroupAlone], (instregex "SRNM(B|T)?$")>;
 922: 
 923: 
 924: // --------------------- Hexadecimal floating point ------------------------- //
 925: 
 926: //===----------------------------------------------------------------------===//
 927: // HFP: Move instructions
 928: //===----------------------------------------------------------------------===//
 929: 
 930: // Load and Test
 931: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "LT(E|D)R$")>;
 932: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXR$")>;
 933: 
 934: //===----------------------------------------------------------------------===//
 935: // HFP: Conversion instructions
 936: //===----------------------------------------------------------------------===//
 937: 
 938: // Load rounded
 939: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "(LEDR|LRER)$")>;
 940: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "LEXR$")>;
 941: def : InstRW<[WLat9, VecDF2, NormalGr], (instregex "(LDXR|LRDR)$")>;
 942: 
 943: // Load lengthened
 944: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LDE$")>;
 945: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LDER$")>;
 946: def : InstRW<[WLat8LSU, VecBF4, LSU, GroupAlone], (instregex "LX(E|D)$")>;
 947: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "LX(E|D)R$")>;
 948: 
 949: // Convert from fixed
 950: def : InstRW<[WLat8, FXb, VecBF, Cracked], (instregex "C(E|D)(F|G)R$")>;
 951: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)R$")>;
 952: 
 953: // Convert to fixed
 954: def : InstRW<[WLat10, WLat10, FXb, VecBF, Cracked], (instregex "C(F|G)(E|D)R$")>;
 955: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked], (instregex "C(F|G)XR$")>;
 956: 
 957: // Convert BFP to HFP / HFP to BFP.
 958: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "THD(E)?R$")>;
 959: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "TB(E)?DR$")>;
 960: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: //===----------------------------------------------------------------------===//
 962: // HFP: Unary arithmetic
 963: //===----------------------------------------------------------------------===//
 964: 
 965: // Load Complement / Negative / Positive
 966: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "L(C|N|P)(E|D)R$")>;
 967: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "L(C|N|P)XR$")>;
 968: 
 969: // Halve
 970: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "H(E|D)R$")>;
 971: 
 972: // Square root
 973: def : InstRW<[WLat30, VecFPd, LSU, NormalGr], (instregex "SQ(E|D)$")>;
 974: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "SQ(E|D)R$")>;
 975: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "SQXR$")>;
 976: 
 977: // Load FP integer
 978: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "FI(E|D)R$")>;
 979: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXR$")>;
 980: 
 981: //===----------------------------------------------------------------------===//
 982: // HFP: Binary arithmetic
 983: //===----------------------------------------------------------------------===//
 984: 
 985: // Addition
 986: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
 987:              (instregex "A(E|D|U|W)$")>;
 988: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "A(E|D|U|W)R$")>;
 989: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXR$")>;
 990: 
 991: // Subtraction
 992: def : InstRW<[WLat7LSU, WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
 993:              (instregex "S(E|D|U|W)$")>;
 994: def : InstRW<[WLat7, WLat7, VecBF, NormalGr], (instregex "S(E|D|U|W)R$")>;
 995: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXR$")>;
 996: 
 997: // Multiply
 998: def : InstRW<[WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
 999:              (instregex "M(D|DE|E|EE)$")>;
1000: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "M(D|DE|E|EE)R$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: def : InstRW<[WLat8LSU, RegReadAdv, VecBF4, LSU, GroupAlone],
1002:              (instregex "MXD$")>;
1003: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "MXDR$")>;
1004: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "MXR$")>;
1005: def : InstRW<[WLat8LSU, RegReadAdv, VecBF4, LSU, GroupAlone],
1006:              (instregex "MY$")>;
1007: def : InstRW<[WLat7LSU, RegReadAdv, VecBF2, LSU, GroupAlone],
1008:              (instregex "MY(H|L)$")>;
1009: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "MYR$")>;
1010: def : InstRW<[WLat7, VecBF, GroupAlone], (instregex "MY(H|L)R$")>;
1011: 
1012: // Multiply and add / subtract
1013: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
1014:              (instregex "M(A|S)(E|D)$")>;
1015: def : InstRW<[WLat7, VecBF, GroupAlone], (instregex "M(A|S)(E|D)R$")>;
1016: def : InstRW<[WLat8LSU, RegReadAdv, RegReadAdv, VecBF4, LSU, GroupAlone],
1017:              (instregex "MAY$")>;
1018: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
1019:              (instregex "MAY(H|L)$")>;
1020: def : InstRW<[WLat8, VecBF4, GroupAlone], (instregex "MAYR$")>;
1021: def : InstRW<[WLat7, VecBF, GroupAlone], (instregex "MAY(H|L)R$")>;
1022: 
1023: // Division
1024: def : InstRW<[WLat30, RegReadAdv, VecFPd, LSU, NormalGr],
1025:              (instregex "D(E|D)$")>;
1026: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "D(E|D)R$")>;
1027: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "DXR$")>;
1028: 
1029: //===----------------------------------------------------------------------===//
1030: // HFP: Comparisons
1031: //===----------------------------------------------------------------------===//
1032: 
1033: // Compare
1034: def : InstRW<[WLat7LSU, RegReadAdv, VecBF, LSU, NormalGr],
1035:              (instregex "C(E|D)$")>;
1036: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "C(E|D)R$")>;
1037: def : InstRW<[WLat10, VecDF2, GroupAlone], (instregex "CXR$")>;
1038: 
1039: 
1040: // ------------------------ Decimal floating point -------------------------- //
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: 
1042: //===----------------------------------------------------------------------===//
1043: // DFP: Move instructions
1044: //===----------------------------------------------------------------------===//
1045: 
1046: // Load and Test
1047: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "LTDTR$")>;
1048: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXTR$")>;
1049: 
1050: //===----------------------------------------------------------------------===//
1051: // DFP: Conversion instructions
1052: //===----------------------------------------------------------------------===//
1053: 
1054: // Load rounded
1055: def : InstRW<[WLat15, VecDF, NormalGr], (instregex "LEDTR$")>;
1056: def : InstRW<[WLat15, VecDF2, NormalGr], (instregex "LDXTR$")>;
1057: 
1058: // Load lengthened
1059: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "LDETR$")>;
1060: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "LXDTR$")>;
1061: 
1062: // Convert from fixed / logical
1063: def : InstRW<[WLat30, FXb, VecDF, Cracked], (instregex "CD(F|G)TR(A)?$")>;
1064: def : InstRW<[WLat30, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)TR(A)?$")>;
1065: def : InstRW<[WLat30, FXb, VecDF, Cracked], (instregex "CDL(F|G)TR$")>;
1066: def : InstRW<[WLat30, FXb, VecDF4, GroupAlone2], (instregex "CXL(F|G)TR$")>;
1067: 
1068: // Convert to fixed / logical
1069: def : InstRW<[WLat30, WLat30, FXb, VecDF, Cracked],
1070:              (instregex "C(F|G)DTR(A)?$")>;
1071: def : InstRW<[WLat30, WLat30, FXb, VecDF2, Cracked],
1072:              (instregex "C(F|G)XTR(A)?$")>;
1073: def : InstRW<[WLat30, WLat30, FXb, VecDF, Cracked], (instregex "CL(F|G)DTR$")>;
1074: def : InstRW<[WLat30, WLat30, FXb, VecDF2, Cracked], (instregex "CL(F|G)XTR$")>;
1075: 
1076: // Convert from / to signed / unsigned packed
1077: def : InstRW<[WLat9, FXb, VecDF, Cracked], (instregex "CD(S|U)TR$")>;
1078: def : InstRW<[WLat12, FXb2, VecDF4, GroupAlone2], (instregex "CX(S|U)TR$")>;
1079: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "C(S|U)DTR$")>;
1080: def : InstRW<[WLat15, FXb2, VecDF4, GroupAlone2], (instregex "C(S|U)XTR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: 
1082: // Convert from / to zoned
1083: def : InstRW<[WLat8LSU, LSU, VecDF, Cracked], (instregex "CDZT$")>;
1084: def : InstRW<[WLat16LSU, LSU2, VecDF4, GroupAlone3], (instregex "CXZT$")>;
1085: def : InstRW<[WLat1, FXb, LSU, VecDF, Cracked], (instregex "CZDT$")>;
1086: def : InstRW<[WLat1, FXb, LSU, VecDF2, GroupAlone], (instregex "CZXT$")>;
1087: 
1088: // Convert from / to packed
1089: def : InstRW<[WLat8LSU, LSU, VecDF, Cracked], (instregex "CDPT$")>;
1090: def : InstRW<[WLat16LSU, LSU2, VecDF4, GroupAlone3], (instregex "CXPT$")>;
1091: def : InstRW<[WLat1, FXb, LSU, VecDF, Cracked], (instregex "CPDT$")>;
1092: def : InstRW<[WLat1, FXb, LSU, VecDF2, GroupAlone], (instregex "CPXT$")>;
1093: 
1094: // Perform floating-point operation
1095: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "PFPO$")>;
1096: 
1097: //===----------------------------------------------------------------------===//
1098: // DFP: Unary arithmetic
1099: //===----------------------------------------------------------------------===//
1100: 
1101: // Load FP integer
1102: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "FIDTR$")>;
1103: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXTR$")>;
1104: 
1105: // Extract biased exponent
1106: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "EEDTR$")>;
1107: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "EEXTR$")>;
1108: 
1109: // Extract significance
1110: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "ESDTR$")>;
1111: def : InstRW<[WLat12, FXb, VecDF2, Cracked], (instregex "ESXTR$")>;
1112: 
1113: //===----------------------------------------------------------------------===//
1114: // DFP: Binary arithmetic
1115: //===----------------------------------------------------------------------===//
1116: 
1117: // Addition
1118: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "ADTR(A)?$")>;
1119: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXTR(A)?$")>;
1120: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: // Subtraction
1122: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "SDTR(A)?$")>;
1123: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXTR(A)?$")>;
1124: 
1125: // Multiply
1126: def : InstRW<[WLat30, VecDF, NormalGr], (instregex "MDTR(A)?$")>;
1127: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "MXTR(A)?$")>;
1128: 
1129: // Division
1130: def : InstRW<[WLat30, VecDF, NormalGr], (instregex "DDTR(A)?$")>;
1131: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "DXTR(A)?$")>;
1132: 
1133: // Quantize
1134: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "QADTR$")>;
1135: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "QAXTR$")>;
1136: 
1137: // Reround
1138: def : InstRW<[WLat9, WLat9, FXb, VecDF, Cracked], (instregex "RRDTR$")>;
1139: def : InstRW<[WLat11, WLat11, FXb, VecDF4, GroupAlone2], (instregex "RRXTR$")>;
1140: 
1141: // Shift significand left/right
1142: def : InstRW<[WLat11LSU, LSU, VecDF, GroupAlone], (instregex "S(L|R)DT$")>;
1143: def : InstRW<[WLat11LSU, LSU, VecDF4, GroupAlone], (instregex "S(L|R)XT$")>;
1144: 
1145: // Insert biased exponent
1146: def : InstRW<[WLat9, FXb, VecDF, Cracked], (instregex "IEDTR$")>;
1147: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "IEXTR$")>;
1148: 
1149: //===----------------------------------------------------------------------===//
1150: // DFP: Comparisons
1151: //===----------------------------------------------------------------------===//
1152: 
1153: // Compare
1154: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "(K|C)DTR$")>;
1155: def : InstRW<[WLat9, VecDF2, GroupAlone], (instregex "(K|C)XTR$")>;
1156: 
1157: // Compare biased exponent
1158: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "CEDTR$")>;
1159: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "CEXTR$")>;
1160: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161: // Test Data Class/Group
1162: def : InstRW<[WLat15, LSU, VecDF, NormalGr], (instregex "TD(C|G)(E|D)T$")>;
1163: def : InstRW<[WLat15, LSU, VecDF2, GroupAlone], (instregex "TD(C|G)XT$")>;
1164: 
1165: 
1166: // --------------------------------- Vector --------------------------------- //
1167: 
1168: //===----------------------------------------------------------------------===//
1169: // Vector: Move instructions
1170: //===----------------------------------------------------------------------===//
1171: 
1172: def : InstRW<[WLat1, FXb, NormalGr], (instregex "VLR(16|32|64)?$")>;
1173: def : InstRW<[WLat4, FXb, NormalGr], (instregex "VLGV(B|F|G|H)?$")>;
1174: def : InstRW<[WLat1, FXb, NormalGr], (instregex "VLVG(B|F|G|H)?$")>;
1175: def : InstRW<[WLat3, FXb, NormalGr], (instregex "VLVGP(32)?$")>;
1176: 
1177: //===----------------------------------------------------------------------===//
1178: // Vector: Immediate instructions
1179: //===----------------------------------------------------------------------===//
1180: 
1181: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VZERO$")>;
1182: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VONE$")>;
1183: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VGBM$")>;
1184: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VGM(B|F|G|H)?$")>;
1185: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VREPI(B|F|G|H)?$")>;
1186: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLEI(B|F|G|H)$")>;
1187: 
1188: //===----------------------------------------------------------------------===//
1189: // Vector: Loads
1190: //===----------------------------------------------------------------------===//
1191: 
1192: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(Align)?$")>;
1193: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(L|BB)$")>;
1194: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(16|32|64)$")>;
1195: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLLEZ(B|F|G|H)?$")>;
1196: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLREP(B|F|G|H)?$")>;
1197: def : InstRW<[WLat2LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
1198:              (instregex "VLE(B|F|G|H)$")>;
1199: def : InstRW<[WLat6LSU, RegReadAdv, FXb, LSU, VecXsPm, Cracked],
1200:              (instregex "VGE(F|G)$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201: def : InstRW<[WLat4LSU, WLat4LSU, LSU5, GroupAlone],
1202:              (instregex "VLM(Align)?$")>;
1203: 
1204: //===----------------------------------------------------------------------===//
1205: // Vector: Stores
1206: //===----------------------------------------------------------------------===//
1207: 
1208: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VST(Align|L|16|32|64)?$")>;
1209: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTE(F|G)$")>;
1210: def : InstRW<[WLat1, FXb, LSU, VecXsPm, Cracked], (instregex "VSTE(B|H)$")>;
1211: def : InstRW<[WLat1, LSU2, FXb3, GroupAlone2], (instregex "VSTM(Align)?$")>;
1212: def : InstRW<[WLat1, FXb2, LSU, Cracked], (instregex "VSCE(F|G)$")>;
1213: 
1214: //===----------------------------------------------------------------------===//
1215: // Vector: Selects and permutes
1216: //===----------------------------------------------------------------------===//
1217: 
1218: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMRH(B|F|G|H)?$")>;
1219: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMRL(B|F|G|H)?$")>;
1220: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPERM$")>;
1221: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPDI$")>;
1222: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VREP(B|F|G|H)?$")>;
1223: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSEL$")>;
1224: 
1225: //===----------------------------------------------------------------------===//
1226: // Vector: Widening and narrowing
1227: //===----------------------------------------------------------------------===//
1228: 
1229: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPK(F|G|H)?$")>;
1230: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPKS(F|G|H)?$")>;
1231: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VPKS(F|G|H)S$")>;
1232: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPKLS(F|G|H)?$")>;
1233: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VPKLS(F|G|H)S$")>;
1234: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSEG(B|F|H)?$")>;
1235: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPH(B|F|H)?$")>;
1236: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPL(B|F)?$")>;
1237: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPLH(B|F|H|W)?$")>;
1238: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPLL(B|F|H)?$")>;
1239: 
1240: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241: // Vector: Integer arithmetic
1242: //===----------------------------------------------------------------------===//
1243: 
1244: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VA(B|F|G|H|Q|C|CQ)?$")>;
1245: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VACC(B|F|G|H|Q|C|CQ)?$")>;
1246: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VAVG(B|F|G|H)?$")>;
1247: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VAVGL(B|F|G|H)?$")>;
1248: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VN(C|O)?$")>;
1249: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VO$")>;
1250: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VCKSM$")>;
1251: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCLZ(B|F|G|H)?$")>;
1252: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCTZ(B|F|G|H)?$")>;
1253: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VX$")>;
1254: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFM?$")>;
1255: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFMA(B|F|G|H)?$")>;
1256: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFM(B|F|G|H)$")>;
1257: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLC(B|F|G|H)?$")>;
1258: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLP(B|F|G|H)?$")>;
1259: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMX(B|F|G|H)?$")>;
1260: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMXL(B|F|G|H)?$")>;
1261: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMN(B|F|G|H)?$")>;
1262: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMNL(B|F|G|H)?$")>;
1263: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAL(B|F)?$")>;
1264: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALE(B|F|H)?$")>;
1265: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALH(B|F|H|W)?$")>;
1266: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALO(B|F|H)?$")>;
1267: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAO(B|F|H)?$")>;
1268: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAE(B|F|H)?$")>;
1269: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAH(B|F|H)?$")>;
1270: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VME(B|F|H)?$")>;
1271: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMH(B|F|H)?$")>;
1272: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VML(B|F)?$")>;
1273: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLE(B|F|H)?$")>;
1274: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLH(B|F|H|W)?$")>;
1275: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLO(B|F|H)?$")>;
1276: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMO(B|F|H)?$")>;
1277: 
1278: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPOPCT$")>;
1279: 
1280: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERLL(B|F|G|H)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERLLV(B|F|G|H)?$")>;
1282: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERIM(B|F|G|H)?$")>;
1283: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESL(B|F|G|H)?$")>;
1284: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESLV(B|F|G|H)?$")>;
1285: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRA(B|F|G|H)?$")>;
1286: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRAV(B|F|G|H)?$")>;
1287: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRL(B|F|G|H)?$")>;
1288: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRLV(B|F|G|H)?$")>;
1289: 
1290: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSL(DB)?$")>;
1291: def : InstRW<[WLat3, VecXsPm2, NormalGr], (instregex "VSLB$")>;
1292: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSR(A|L)$")>;
1293: def : InstRW<[WLat3, VecXsPm2, NormalGr], (instregex "VSR(A|L)B$")>;
1294: 
1295: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSB(I|IQ|CBI|CBIQ)?$")>;
1296: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSCBI(B|F|G|H|Q)?$")>;
1297: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VS(F|G|H|Q)?$")>;
1298: 
1299: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUM(B|H)?$")>;
1300: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUMG(F|H)?$")>;
1301: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUMQ(F|G)?$")>;
1302: 
1303: //===----------------------------------------------------------------------===//
1304: // Vector: Integer comparison
1305: //===----------------------------------------------------------------------===//
1306: 
1307: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "VEC(B|F|G|H)?$")>;
1308: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "VECL(B|F|G|H)?$")>;
1309: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCEQ(B|F|G|H)?$")>;
1310: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCEQ(B|F|G|H)S$")>;
1311: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCH(B|F|G|H)?$")>;
1312: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCH(B|F|G|H)S$")>;
1313: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCHL(B|F|G|H)?$")>;
1314: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCHL(B|F|G|H)S$")>;
1315: def : InstRW<[WLat4, VecStr, NormalGr], (instregex "VTM$")>;
1316: 
1317: //===----------------------------------------------------------------------===//
1318: // Vector: Floating-point arithmetic
1319: //===----------------------------------------------------------------------===//
1320: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1321-1360 / 第 1321-1360 行
```tablegen
1321: // Conversion and rounding
1322: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VCD(L)?G$")>;
1323: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VCD(L)?GB$")>;
1324: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WCD(L)?GB$")>;
1325: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VC(L)?GD$")>;
1326: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VC(L)?GDB$")>;
1327: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WC(L)?GDB$")>;
1328: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VL(DE|ED)$")>;
1329: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VL(DE|ED)B$")>;
1330: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WL(DE|ED)B$")>;
1331: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFI$")>;
1332: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFIDB$")>;
1333: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WFIDB$")>;
1334: 
1335: // Sign operations
1336: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VFPSO$")>;
1337: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FPSODB$")>;
1338: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FL(C|N|P)DB$")>;
1339: 
1340: // Test data class
1341: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFTCI$")>;
1342: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "(V|W)FTCIDB$")>;
1343: 
1344: // Add / subtract
1345: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VF(A|S)$")>;
1346: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VF(A|S)DB$")>;
1347: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WF(A|S)DB$")>;
1348: 
1349: // Multiply / multiply-and-add/subtract
1350: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFM$")>;
1351: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFMDB$")>;
1352: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WFMDB$")>;
1353: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFM(A|S)$")>;
1354: def : InstRW<[WLat8, VecBF2, NormalGr], (instregex "VFM(A|S)DB$")>;
1355: def : InstRW<[WLat7, VecBF, NormalGr], (instregex "WFM(A|S)DB$")>;
1356: 
1357: // Divide / square root
1358: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFD$")>;
1359: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FDDB$")>;
1360: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFSQ$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1361-1400 / 第 1361-1400 行
```tablegen
1361: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FSQDB$")>;
1362: 
1363: //===----------------------------------------------------------------------===//
1364: // Vector: Floating-point comparison
1365: //===----------------------------------------------------------------------===//
1366: 
1367: def : InstRW<[WLat2, WLat2, VecXsPm, NormalGr], (instregex "VFC(E|H|HE)$")>;
1368: def : InstRW<[WLat2, WLat2, VecXsPm, NormalGr], (instregex "VFC(E|H|HE)DB$")>;
1369: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)DB$")>;
1370: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFC(E|H|HE)DBS$")>;
1371: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)DBS$")>;
1372: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)$")>;
1373: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)DB$")>;
1374: 
1375: //===----------------------------------------------------------------------===//
1376: // Vector: Floating-point insertion and extraction
1377: //===----------------------------------------------------------------------===//
1378: 
1379: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LEFR(_16)?$")>;
1380: def : InstRW<[WLat4, FXb, NormalGr], (instregex "LFER(_16)?$")>;
1381: 
1382: //===----------------------------------------------------------------------===//
1383: // Vector: String instructions
1384: //===----------------------------------------------------------------------===//
1385: 
1386: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAE(B)?$")>;
1387: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAE(F|H)$")>;
1388: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VFAE(B|F|H)S$")>;
1389: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAEZ(B|F|H)$")>;
1390: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VFAEZ(B|F|H)S$")>;
1391: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFEE(B|F|H|ZB|ZF|ZH)?$")>;
1392: def : InstRW<[WLat4, WLat4, VecStr, NormalGr],
1393:              (instregex "VFEE(B|F|H|ZB|ZF|ZH)S$")>;
1394: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFENE(B|F|H|ZB|ZF|ZH)?$")>;
1395: def : InstRW<[WLat4, WLat4, VecStr, NormalGr],
1396:              (instregex "VFENE(B|F|H|ZB|ZF|ZH)S$")>;
1397: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VISTR(B|F|H)?$")>;
1398: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VISTR(B|F|H)S$")>;
1399: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VSTRC(B|F|H)?$")>;
1400: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRC(B|F|H)S$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1401-1440 / 第 1401-1440 行
```tablegen
1401: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VSTRCZ(B|F|H)$")>;
1402: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRCZ(B|F|H)S$")>;
1403: 
1404: 
1405: // -------------------------------- System ---------------------------------- //
1406: 
1407: //===----------------------------------------------------------------------===//
1408: // System: Program-Status Word Instructions
1409: //===----------------------------------------------------------------------===//
1410: 
1411: def : InstRW<[WLat30, WLat30, MCD], (instregex "EPSW$")>;
1412: def : InstRW<[WLat30, MCD], (instregex "LPSW(E)?$")>;
1413: def : InstRW<[WLat3, FXa, GroupAlone], (instregex "IPK$")>;
1414: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SPKA$")>;
1415: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SSM$")>;
1416: def : InstRW<[WLat1, FXb, LSU, GroupAlone], (instregex "ST(N|O)SM$")>;
1417: def : InstRW<[WLat3, FXa, NormalGr], (instregex "IAC$")>;
1418: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SAC(F)?$")>;
1419: 
1420: //===----------------------------------------------------------------------===//
1421: // System: Control Register Instructions
1422: //===----------------------------------------------------------------------===//
1423: 
1424: def : InstRW<[WLat4LSU, WLat4LSU, LSU2, GroupAlone], (instregex "LCTL(G)?$")>;
1425: def : InstRW<[WLat1, LSU5, FXb, GroupAlone2], (instregex "STCT(L|G)$")>;
1426: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "E(P|S)A(I)?R$")>;
1427: def : InstRW<[WLat30, MCD], (instregex "SSA(I)?R$")>;
1428: def : InstRW<[WLat30, MCD], (instregex "ESEA$")>;
1429: 
1430: //===----------------------------------------------------------------------===//
1431: // System: Prefix-Register Instructions
1432: //===----------------------------------------------------------------------===//
1433: 
1434: def : InstRW<[WLat30, MCD], (instregex "S(T)?PX$")>;
1435: 
1436: //===----------------------------------------------------------------------===//
1437: // System: Storage-Key and Real Memory Instructions
1438: //===----------------------------------------------------------------------===//
1439: 
1440: def : InstRW<[WLat30, MCD], (instregex "ISKE$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1441-1480 / 第 1441-1480 行
```tablegen
1441: def : InstRW<[WLat30, MCD], (instregex "IVSK$")>;
1442: def : InstRW<[WLat30, MCD], (instregex "SSKE(Opt)?$")>;
1443: def : InstRW<[WLat30, MCD], (instregex "RRB(E|M)$")>;
1444: def : InstRW<[WLat30, MCD], (instregex "PFMF$")>;
1445: def : InstRW<[WLat30, WLat30, MCD], (instregex "TB$")>;
1446: def : InstRW<[WLat30, MCD], (instregex "PGIN$")>;
1447: def : InstRW<[WLat30, MCD], (instregex "PGOUT$")>;
1448: 
1449: //===----------------------------------------------------------------------===//
1450: // System: Dynamic-Address-Translation Instructions
1451: //===----------------------------------------------------------------------===//
1452: 
1453: def : InstRW<[WLat30, MCD], (instregex "IPTE(Opt)?(Opt)?$")>;
1454: def : InstRW<[WLat30, MCD], (instregex "IDTE(Opt)?$")>;
1455: def : InstRW<[WLat30, MCD], (instregex "CRDTE(Opt)?$")>;
1456: def : InstRW<[WLat30, MCD], (instregex "PTLB$")>;
1457: def : InstRW<[WLat30, WLat30, MCD], (instregex "CSP(G)?$")>;
1458: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "LPTEA$")>;
1459: def : InstRW<[WLat30, WLat30, MCD], (instregex "LRA(Y|G)?$")>;
1460: def : InstRW<[WLat30, MCD], (instregex "STRAG$")>;
1461: def : InstRW<[WLat30, MCD], (instregex "LURA(G)?$")>;
1462: def : InstRW<[WLat30, MCD], (instregex "STUR(A|G)$")>;
1463: def : InstRW<[WLat30, MCD], (instregex "TPROT$")>;
1464: 
1465: //===----------------------------------------------------------------------===//
1466: // System: Memory-move Instructions
1467: //===----------------------------------------------------------------------===//
1468: 
1469: def : InstRW<[WLat4LSU, FXa2, FXb, LSU5, GroupAlone2], (instregex "MVC(K|P|S)$")>;
1470: def : InstRW<[WLat1, FXa, LSU5, GroupAlone2], (instregex "MVC(S|D)K$")>;
1471: def : InstRW<[WLat30, MCD], (instregex "MVCOS$")>;
1472: def : InstRW<[WLat30, MCD], (instregex "MVPG$")>;
1473: 
1474: //===----------------------------------------------------------------------===//
1475: // System: Address-Space Instructions
1476: //===----------------------------------------------------------------------===//
1477: 
1478: def : InstRW<[WLat30, MCD], (instregex "LASP$")>;
1479: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "PALB$")>;
1480: def : InstRW<[WLat30, MCD], (instregex "PC$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1481-1520 / 第 1481-1520 行
```tablegen
1481: def : InstRW<[WLat30, MCD], (instregex "PR$")>;
1482: def : InstRW<[WLat30, MCD], (instregex "PT(I)?$")>;
1483: def : InstRW<[WLat30, MCD], (instregex "RP$")>;
1484: def : InstRW<[WLat30, MCD], (instregex "BS(G|A)$")>;
1485: def : InstRW<[WLat30, MCD], (instregex "TAR$")>;
1486: 
1487: //===----------------------------------------------------------------------===//
1488: // System: Linkage-Stack Instructions
1489: //===----------------------------------------------------------------------===//
1490: 
1491: def : InstRW<[WLat30, MCD], (instregex "BAKR$")>;
1492: def : InstRW<[WLat30, MCD], (instregex "EREG(G)?$")>;
1493: def : InstRW<[WLat30, WLat30, MCD], (instregex "(E|M)STA$")>;
1494: 
1495: //===----------------------------------------------------------------------===//
1496: // System: Time-Related Instructions
1497: //===----------------------------------------------------------------------===//
1498: 
1499: def : InstRW<[WLat30, MCD], (instregex "PTFF$")>;
1500: def : InstRW<[WLat30, MCD], (instregex "SCK(PF|C)?$")>;
1501: def : InstRW<[WLat1, LSU2, GroupAlone], (instregex "SPT$")>;
1502: def : InstRW<[WLat15, LSU3, FXa2, FXb, GroupAlone2], (instregex "STCK(F)?$")>;
1503: def : InstRW<[WLat20, LSU4, FXa2, FXb2, GroupAlone3], (instregex "STCKE$")>;
1504: def : InstRW<[WLat30, MCD], (instregex "STCKC$")>;
1505: def : InstRW<[WLat1, LSU2, FXb, Cracked], (instregex "STPT$")>;
1506: 
1507: //===----------------------------------------------------------------------===//
1508: // System: CPU-Related Instructions
1509: //===----------------------------------------------------------------------===//
1510: 
1511: def : InstRW<[WLat30, MCD], (instregex "STAP$")>;
1512: def : InstRW<[WLat30, MCD], (instregex "STIDP$")>;
1513: def : InstRW<[WLat30, WLat30, MCD], (instregex "STSI$")>;
1514: def : InstRW<[WLat30, WLat30, MCD], (instregex "STFL(E)?$")>;
1515: def : InstRW<[WLat30, MCD], (instregex "ECAG$")>;
1516: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECTG$")>;
1517: def : InstRW<[WLat30, MCD], (instregex "PTF$")>;
1518: def : InstRW<[WLat30, MCD], (instregex "PCKMO$")>;
1519: 
1520: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1521-1560 / 第 1521-1560 行
```tablegen
1521: // System: Miscellaneous Instructions
1522: //===----------------------------------------------------------------------===//
1523: 
1524: def : InstRW<[WLat30, MCD], (instregex "SVC$")>;
1525: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "MC$")>;
1526: def : InstRW<[WLat30, MCD], (instregex "DIAG$")>;
1527: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TRAC(E|G)$")>;
1528: def : InstRW<[WLat30, MCD], (instregex "TRAP(2|4)$")>;
1529: def : InstRW<[WLat30, MCD], (instregex "SIG(P|A)$")>;
1530: def : InstRW<[WLat30, MCD], (instregex "SIE$")>;
1531: 
1532: //===----------------------------------------------------------------------===//
1533: // System: CPU-Measurement Facility Instructions
1534: //===----------------------------------------------------------------------===//
1535: 
1536: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LPP$")>;
1537: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECPGA$")>;
1538: def : InstRW<[WLat30, WLat30, MCD], (instregex "E(C|P)CTR$")>;
1539: def : InstRW<[WLat30, MCD], (instregex "LCCTL$")>;
1540: def : InstRW<[WLat30, MCD], (instregex "L(P|S)CTL$")>;
1541: def : InstRW<[WLat30, MCD], (instregex "Q(S|CTR)I$")>;
1542: def : InstRW<[WLat30, MCD], (instregex "S(C|P)CTR$")>;
1543: 
1544: //===----------------------------------------------------------------------===//
1545: // System: I/O Instructions
1546: //===----------------------------------------------------------------------===//
1547: 
1548: def : InstRW<[WLat30, MCD], (instregex "(C|H|R|X)SCH$")>;
1549: def : InstRW<[WLat30, MCD], (instregex "(M|S|ST|T)SCH$")>;
1550: def : InstRW<[WLat30, MCD], (instregex "RCHP$")>;
1551: def : InstRW<[WLat30, MCD], (instregex "SCHM$")>;
1552: def : InstRW<[WLat30, MCD], (instregex "STC(PS|RW)$")>;
1553: def : InstRW<[WLat30, MCD], (instregex "TPI$")>;
1554: def : InstRW<[WLat30, MCD], (instregex "SAL$")>;
1555: 
1556: //===----------------------------------------------------------------------===//
1557: // NOPs
1558: //===----------------------------------------------------------------------===//
1559: 
1560: def : InstRW<[WLat1, FXb, NormalGr], (instregex "NOP(R)?(Opt)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1561-1563 / 第 1561-1563 行
```tablegen
1561: def : InstRW<[WLat1, VBU, NormalGr], (instregex "J(G)?NOP$")>;
1562: }
1563: 
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
