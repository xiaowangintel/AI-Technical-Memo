# SystemZScheduleZ17.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZScheduleZ17.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //--- SystemZScheduleZ17.td - SystemZ Scheduling Definitions ---*- tblgen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the machine model for Z17 to support instruction
  10: // scheduling and other instruction cost heuristics.
  11: //
  12: // Pseudos expanded right after isel do not need to be modelled here.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: def Z17Model : SchedMachineModel {
  17: 
  18:     let UnsupportedFeatures = Arch15UnsupportedFeatures.List;
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
  30: let SchedModel = Z17Model in  {
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
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `Z17Model`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `Z17Model` 等 TableGen 记录。

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
  76: def Z17_FXaUnit     : ProcResource<2>;
  77: def Z17_FXbUnit     : ProcResource<2>;
  78: def Z17_LSUnit      : ProcResource<2>;
  79: def Z17_VecUnit     : ProcResource<2>;
  80: def Z17_VecFPdUnit  : ProcResource<2> { let BufferSize = 1; /* blocking */ }
```
- **EN**: This block declares or refines TableGen records such as `Z17_FXaUnit`, `Z17_FXbUnit`, `Z17_LSUnit`, `Z17_VecUnit`, `Z17_VecFPdUnit`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `Z17_FXaUnit`, `Z17_FXbUnit`, `Z17_LSUnit`, `Z17_VecUnit`, `Z17_VecFPdUnit` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 81-120 / 第 81-120 行
```tablegen
  81: def Z17_VBUnit      : ProcResource<2>;
  82: def Z17_MCD         : ProcResource<1>;
  83: 
  84: // Subtarget specific definitions of scheduling resources.
  85: let NumMicroOps = 0 in {
  86:   def : WriteRes<FXa, [Z17_FXaUnit]>;
  87:   def : WriteRes<FXb, [Z17_FXbUnit]>;
  88:   def : WriteRes<LSU, [Z17_LSUnit]>;
  89:   def : WriteRes<VecBF,  [Z17_VecUnit]>;
  90:   def : WriteRes<VecDF,  [Z17_VecUnit]>;
  91:   def : WriteRes<VecDFX, [Z17_VecUnit]>;
  92:   def : WriteRes<VecMul,  [Z17_VecUnit]>;
  93:   def : WriteRes<VecStr,  [Z17_VecUnit]>;
  94:   def : WriteRes<VecXsPm, [Z17_VecUnit]>;
  95:   foreach Num = 2-5 in { let ReleaseAtCycles = [Num] in {
  96:     def : WriteRes<!cast<SchedWrite>("FXa"#Num), [Z17_FXaUnit]>;
  97:     def : WriteRes<!cast<SchedWrite>("FXb"#Num), [Z17_FXbUnit]>;
  98:     def : WriteRes<!cast<SchedWrite>("LSU"#Num), [Z17_LSUnit]>;
  99:     def : WriteRes<!cast<SchedWrite>("VecBF"#Num), [Z17_VecUnit]>;
 100:     def : WriteRes<!cast<SchedWrite>("VecDF"#Num), [Z17_VecUnit]>;
 101:     def : WriteRes<!cast<SchedWrite>("VecDFX"#Num), [Z17_VecUnit]>;
 102:     def : WriteRes<!cast<SchedWrite>("VecMul"#Num), [Z17_VecUnit]>;
 103:     def : WriteRes<!cast<SchedWrite>("VecStr"#Num), [Z17_VecUnit]>;
 104:     def : WriteRes<!cast<SchedWrite>("VecXsPm"#Num), [Z17_VecUnit]>;
 105:   }}
 106: 
 107:   def : WriteRes<VecFPd,   [Z17_VecFPdUnit]> { let ReleaseAtCycles = [30]; }
 108:   def : WriteRes<VecFPd20, [Z17_VecFPdUnit]> { let ReleaseAtCycles = [20]; }
 109: 
 110:   def : WriteRes<VBU,     [Z17_VBUnit]>; // Virtual Branching Unit
 111: }
 112: 
 113: def : WriteRes<MCD, [Z17_MCD]> { let NumMicroOps = 3;
 114:                                  let BeginGroup  = 1;
 115:                                  let EndGroup    = 1; }
 116: 
 117: // -------------------------- INSTRUCTIONS ---------------------------------- //
 118: 
 119: // InstRW constructs have been used in order to preserve the
 120: // readability of the InstrInfo files.
```
- **EN**: This block declares or refines TableGen records such as `Z17_VBUnit`, `Z17_MCD`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `Z17_VBUnit`, `Z17_MCD` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 121-160 / 第 121-160 行
```tablegen
 121: 
 122: // For each instruction, as matched by a regexp, provide a list of
 123: // resources that it needs. These will be combined into a SchedClass.
 124: 
 125: //===----------------------------------------------------------------------===//
 126: // Stack allocation
 127: //===----------------------------------------------------------------------===//
 128: 
 129: // Pseudo -> LA / LAY
 130: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ADJDYNALLOC$")>;
 131: 
 132: //===----------------------------------------------------------------------===//
 133: // Branch instructions
 134: //===----------------------------------------------------------------------===//
 135: 
 136: // Branch
 137: def : InstRW<[WLat1, VBU, NormalGr], (instregex "(Call)?BRC(L)?(Asm.*)?$")>;
 138: def : InstRW<[WLat1, VBU, NormalGr], (instregex "(Call)?J(G)?(Asm.*)?$")>;
 139: def : InstRW<[WLat1, FXb, NormalGr], (instregex "(Call)?BC(R)?(Asm.*)?$")>;
 140: def : InstRW<[WLat1, FXb, NormalGr], (instregex "(Call)?B(R)?(Asm.*)?$")>;
 141: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "BI(C)?(Asm.*)?$")>;
 142: def : InstRW<[WLat1, FXa, EndGroup], (instregex "BRCT(G)?$")>;
 143: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BRCTH$")>;
 144: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BCT(G)?(R)?$")>;
 145: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "B(R)?X(H|L).*$")>;
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
 318: // Load (logical) indexed address.
 319: def : InstRW<[WLat2, FXa2, NormalGr], (instregex "(L)?LXA(B|H|F|G|Q)$")>;
 320: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 321-360 / 第 321-360 行
```tablegen
 321: //===----------------------------------------------------------------------===//
 322: // Absolute and Negation
 323: //===----------------------------------------------------------------------===//
 324: 
 325: def : InstRW<[WLat1, WLat1, FXa, NormalGr], (instregex "LP(G)?R$")>;
 326: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "L(N|P)GFR$")>;
 327: def : InstRW<[WLat1, WLat1, FXa, NormalGr], (instregex "LN(R|GR)$")>;
 328: def : InstRW<[WLat1, FXa, NormalGr], (instregex "LC(R|GR)$")>;
 329: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "LCGFR$")>;
 330: 
 331: //===----------------------------------------------------------------------===//
 332: // Insertion
 333: //===----------------------------------------------------------------------===//
 334: 
 335: def : InstRW<[WLat1LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "IC(Y)?$")>;
 336: def : InstRW<[WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 337:              (instregex "IC32(Y)?$")>;
 338: def : InstRW<[WLat1LSU, RegReadAdv, WLat1LSU, FXa, LSU, NormalGr],
 339:              (instregex "ICM(H|Y)?$")>;
 340: def : InstRW<[WLat1, FXa, NormalGr], (instregex "II(F|H|L)Mux$")>;
 341: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHF(64)?$")>;
 342: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHH(64)?$")>;
 343: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IIHL(64)?$")>;
 344: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILF(64)?$")>;
 345: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILH(64)?$")>;
 346: def : InstRW<[WLat1, FXa, NormalGr], (instregex "IILL(64)?$")>;
 347: 
 348: //===----------------------------------------------------------------------===//
 349: // Addition
 350: //===----------------------------------------------------------------------===//
 351: 
 352: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 353:              (instregex "A(Y)?$")>;
 354: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 355:              (instregex "AH(Y)?$")>;
 356: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AIH$")>;
 357: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AFI(Mux)?$")>;
 358: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 359:              (instregex "AG$")>;
 360: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGFI$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 361-400 / 第 361-400 行
```tablegen
 361: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGHI(K)?$")>;
 362: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AGR(K)?$")>;
 363: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AHI(K)?$")>;
 364: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AHIMux(K)?$")>;
 365: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 366:              (instregex "AL(Y)?$")>;
 367: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AL(FI|HSIK)$")>;
 368: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 369:              (instregex "ALG(F)?$")>;
 370: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGHSIK$")>;
 371: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGF(I|R)$")>;
 372: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALGR(K)?$")>;
 373: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALR(K)?$")>;
 374: def : InstRW<[WLat1, FXa, NormalGr], (instregex "AR(K)?$")>;
 375: def : InstRW<[WLat1, FXa, NormalGr], (instregex "A(L)?HHHR$")>;
 376: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "A(L)?HHLR$")>;
 377: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ALSIH(N)?$")>;
 378: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "A(L)?(G)?SI$")>;
 379: 
 380: // Logical addition with carry
 381: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, GroupAlone],
 382:              (instregex "ALC(G)?$")>;
 383: def : InstRW<[WLat2, WLat2, FXa, GroupAlone], (instregex "ALC(G)?R$")>;
 384: 
 385: // Add with sign extension (16/32 -> 64)
 386: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 387:              (instregex "AG(F|H)$")>;
 388: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "AGFR$")>;
 389: 
 390: //===----------------------------------------------------------------------===//
 391: // Subtraction
 392: //===----------------------------------------------------------------------===//
 393: 
 394: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 395:              (instregex "S(G|Y)?$")>;
 396: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 397:              (instregex "SH(Y)?$")>;
 398: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SGR(K)?$")>;
 399: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLFI$")>;
 400: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 401-440 / 第 401-440 行
```tablegen
 401:              (instregex "SL(G|GF|Y)?$")>;
 402: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLGF(I|R)$")>;
 403: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLGR(K)?$")>;
 404: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLR(K)?$")>;
 405: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SR(K)?$")>;
 406: def : InstRW<[WLat1, FXa, NormalGr], (instregex "S(L)?HHHR$")>;
 407: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "S(L)?HHLR$")>;
 408: 
 409: // Subtraction with borrow
 410: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, GroupAlone],
 411:              (instregex "SLB(G)?$")>;
 412: def : InstRW<[WLat2, WLat2, FXa, GroupAlone], (instregex "SLB(G)?R$")>;
 413: 
 414: // Subtraction with sign extension (16/32 -> 64)
 415: def : InstRW<[WLat2LSU, WLat2LSU, RegReadAdv, FXa, LSU, NormalGr],
 416:              (instregex "SG(F|H)$")>;
 417: def : InstRW<[WLat2, WLat2, FXa, NormalGr], (instregex "SGFR$")>;
 418: 
 419: //===----------------------------------------------------------------------===//
 420: // AND
 421: //===----------------------------------------------------------------------===//
 422: 
 423: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 424:              (instregex "N(G|Y)?$")>;
 425: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NGR(K)?$")>;
 426: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NI(FMux|HMux|LMux)$")>;
 427: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "NI(Y)?$")>;
 428: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHF(64)?$")>;
 429: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHH(64)?$")>;
 430: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NIHL(64)?$")>;
 431: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILF(64)?$")>;
 432: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILH(64)?$")>;
 433: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NILL(64)?$")>;
 434: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NR(K)?$")>;
 435: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "NC$")>;
 436: 
 437: //===----------------------------------------------------------------------===//
 438: // OR
 439: //===----------------------------------------------------------------------===//
 440: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 441-480 / 第 441-480 行
```tablegen
 441: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 442:              (instregex "O(G|Y)?$")>;
 443: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OGR(K)?$")>;
 444: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "OI(Y)?$")>;
 445: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OI(FMux|HMux|LMux)$")>;
 446: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHF(64)?$")>;
 447: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHH(64)?$")>;
 448: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OIHL(64)?$")>;
 449: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILF(64)?$")>;
 450: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILH(64)?$")>;
 451: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OILL(64)?$")>;
 452: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OR(K)?$")>;
 453: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "OC$")>;
 454: 
 455: //===----------------------------------------------------------------------===//
 456: // XOR
 457: //===----------------------------------------------------------------------===//
 458: 
 459: def : InstRW<[WLat1LSU, WLat1LSU, RegReadAdv, FXa, LSU, NormalGr],
 460:              (instregex "X(G|Y)?$")>;
 461: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "XI(Y)?$")>;
 462: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XIFMux$")>;
 463: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XGR(K)?$")>;
 464: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XIHF(64)?$")>;
 465: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XILF(64)?$")>;
 466: def : InstRW<[WLat1, FXa, NormalGr], (instregex "XR(K)?$")>;
 467: def : InstRW<[WLat3LSU, LSU2, FXb, Cracked], (instregex "XC$")>;
 468: 
 469: //===----------------------------------------------------------------------===//
 470: // Combined logical operations
 471: //===----------------------------------------------------------------------===//
 472: 
 473: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NC(G)?RK$")>;
 474: def : InstRW<[WLat1, FXa, NormalGr], (instregex "OC(G)?RK$")>;
 475: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NN(G)?RK$")>;
 476: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NO(G)?RK$")>;
 477: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NOT(G)?R$")>;
 478: def : InstRW<[WLat1, FXa, NormalGr], (instregex "NX(G)?RK$")>;
 479: 
 480: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: // Multiplication
 482: //===----------------------------------------------------------------------===//
 483: 
 484: def : InstRW<[WLat4LSU, RegReadAdv, FXa, LSU, NormalGr],
 485:              (instregex "MS(GF|Y)?$")>;
 486: def : InstRW<[WLat4, FXa, NormalGr], (instregex "MS(R|FI)$")>;
 487: def : InstRW<[WLat4LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "MSG$")>;
 488: def : InstRW<[WLat4, FXa, NormalGr], (instregex "MSGR$")>;
 489: def : InstRW<[WLat4, FXa, NormalGr], (instregex "MSGF(I|R)$")>;
 490: def : InstRW<[WLat5LSU, RegReadAdv, FXa2, LSU, GroupAlone], (instregex "MLG$")>;
 491: def : InstRW<[WLat5, FXa2, GroupAlone], (instregex "MLGR$")>;
 492: def : InstRW<[WLat4, FXa, NormalGr], (instregex "MGHI$")>;
 493: def : InstRW<[WLat4, FXa, NormalGr], (instregex "MHI$")>;
 494: def : InstRW<[WLat4LSU, RegReadAdv, FXa, LSU, NormalGr], (instregex "MH(Y)?$")>;
 495: def : InstRW<[WLat5, FXa2, GroupAlone], (instregex "M(L)?R$")>;
 496: def : InstRW<[WLat5LSU, RegReadAdv, FXa2, LSU, GroupAlone],
 497:              (instregex "M(FY|L)?$")>;
 498: def : InstRW<[WLat8, RegReadAdv, FXa, LSU, NormalGr], (instregex "MGH$")>;
 499: def : InstRW<[WLat9, RegReadAdv, FXa2, LSU, GroupAlone], (instregex "MG$")>;
 500: def : InstRW<[WLat5, FXa2, GroupAlone], (instregex "MGRK$")>;
 501: def : InstRW<[WLat4LSU, WLat4LSU, RegReadAdv, FXa, LSU, NormalGr],
 502:              (instregex "MSC$")>;
 503: def : InstRW<[WLat4LSU, WLat4LSU, RegReadAdv, FXa, LSU, NormalGr],
 504:              (instregex "MSGC$")>;
 505: def : InstRW<[WLat4, WLat4, FXa, NormalGr], (instregex "MSRKC$")>;
 506: def : InstRW<[WLat4, WLat4, FXa, NormalGr], (instregex "MSGRKC$")>;
 507: 
 508: //===----------------------------------------------------------------------===//
 509: // Division and remainder
 510: //===----------------------------------------------------------------------===//
 511: 
 512: def : InstRW<[WLat20, FXa4, GroupAlone], (instregex "DR$")>;
 513: def : InstRW<[WLat30, RegReadAdv, FXa4, LSU, GroupAlone2], (instregex "D$")>;
 514: def : InstRW<[WLat30, FXa2, GroupAlone], (instregex "DSG(F)?R$")>;
 515: def : InstRW<[WLat30, RegReadAdv, FXa2, LSU, GroupAlone2],
 516:              (instregex "DSG(F)?$")>;
 517: def : InstRW<[WLat20, FXa4, GroupAlone], (instregex "DLR$")>;
 518: def : InstRW<[WLat30, FXa4, GroupAlone], (instregex "DLGR$")>;
 519: def : InstRW<[WLat30, RegReadAdv, FXa4, LSU, GroupAlone2],
 520:              (instregex "DL(G)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 521-560 / 第 521-560 行
```tablegen
 521: 
 522: //===----------------------------------------------------------------------===//
 523: // Shifts
 524: //===----------------------------------------------------------------------===//
 525: 
 526: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLL(G|K)?$")>;
 527: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SRL(G|K)?$")>;
 528: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SRA(G|K)?$")>;
 529: def : InstRW<[WLat1, FXa, NormalGr], (instregex "SLA(G|K)?$")>;
 530: def : InstRW<[WLat5LSU, WLat5LSU, FXa4, LSU, GroupAlone2],
 531:              (instregex "S(L|R)D(A|L)$")>;
 532: 
 533: // Rotate
 534: def : InstRW<[WLat2LSU, FXa, LSU, NormalGr], (instregex "RLL(G)?$")>;
 535: 
 536: // Rotate and insert
 537: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBH(G|H|L)(Opt)?$")>;
 538: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBL(G|H|L)(Opt)?$")>;
 539: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBG(N|32)?(Z)?(Opt)?$")>;
 540: def : InstRW<[WLat1, FXa, NormalGr], (instregex "RISBMux$")>;
 541: 
 542: // Rotate and Select
 543: def : InstRW<[WLat2, WLat2, FXa2, Cracked], (instregex "R(N|O|X)SBG(Opt)?$")>;
 544: 
 545: //===----------------------------------------------------------------------===//
 546: // Comparison
 547: //===----------------------------------------------------------------------===//
 548: 
 549: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr],
 550:              (instregex "C(G|Y|Mux)?$")>;
 551: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CRL$")>;
 552: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(F|H)I(Mux)?$")>;
 553: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CG(F|H)I$")>;
 554: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CG(HSI|RL)$")>;
 555: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(G)?R$")>;
 556: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CIH$")>;
 557: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CHF$")>;
 558: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CHSI$")>;
 559: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr],
 560:              (instregex "CL(Y|Mux)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLFHSI$")>;
 562: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLFI(Mux)?$")>;
 563: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLG$")>;
 564: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLG(HRL|HSI)$")>;
 565: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLGF$")>;
 566: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLGFRL$")>;
 567: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLGF(I|R)$")>;
 568: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLGR$")>;
 569: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLGRL$")>;
 570: def : InstRW<[WLat1LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CLHF$")>;
 571: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLH(RL|HSI)$")>;
 572: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLIH$")>;
 573: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLI(Y)?$")>;
 574: def : InstRW<[WLat1, FXb, NormalGr], (instregex "CLR$")>;
 575: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "CLRL$")>;
 576: def : InstRW<[WLat1, FXb, NormalGr], (instregex "C(L)?HHR$")>;
 577: def : InstRW<[WLat2, FXb, NormalGr], (instregex "C(L)?HLR$")>;
 578: 
 579: // Compare halfword
 580: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CH(Y)?$")>;
 581: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CHRL$")>;
 582: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CGH$")>;
 583: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CGHRL$")>;
 584: def : InstRW<[WLat2LSU, FXa, FXb, LSU, Cracked], (instregex "CHHSI$")>;
 585: 
 586: // Compare with sign extension (32 -> 64)
 587: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr], (instregex "CGF$")>;
 588: def : InstRW<[WLat2LSU, FXb, LSU, NormalGr], (instregex "CGFRL$")>;
 589: def : InstRW<[WLat2, FXb, NormalGr], (instregex "CGFR$")>;
 590: 
 591: // Compare logical character
 592: def : InstRW<[WLat6, FXb, LSU2, Cracked], (instregex "CLC$")>;
 593: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLCL(E|U)?$")>;
 594: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CLST$")>;
 595: 
 596: // Test under mask
 597: def : InstRW<[WLat1LSU, FXb, LSU, NormalGr], (instregex "TM(Y)?$")>;
 598: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TM(H|L)Mux$")>;
 599: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMHH(64)?$")>;
 600: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMHL(64)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 601-640 / 第 601-640 行
```tablegen
 601: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMLH(64)?$")>;
 602: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TMLL(64)?$")>;
 603: 
 604: // Compare logical characters under mask
 605: def : InstRW<[WLat2LSU, RegReadAdv, FXb, LSU, NormalGr],
 606:              (instregex "CLM(H|Y)?$")>;
 607: 
 608: //===----------------------------------------------------------------------===//
 609: // Prefetch and execution hint
 610: //===----------------------------------------------------------------------===//
 611: 
 612: def : InstRW<[WLat1, LSU, NormalGr], (instregex "PFD(RL)?$")>;
 613: def : InstRW<[WLat1, FXb, NormalGr], (instregex "BPP$")>;
 614: def : InstRW<[FXb, EndGroup], (instregex "BPRP$")>;
 615: def : InstRW<[WLat1, FXb, NormalGr], (instregex "NIAI$")>;
 616: 
 617: //===----------------------------------------------------------------------===//
 618: // Atomic operations
 619: //===----------------------------------------------------------------------===//
 620: 
 621: def : InstRW<[WLat1, FXb, EndGroup], (instregex "Serialize$")>;
 622: 
 623: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAA(G)?$")>;
 624: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAAL(G)?$")>;
 625: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAN(G)?$")>;
 626: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAO(G)?$")>;
 627: def : InstRW<[WLat2LSU, WLat2LSU, FXb, LSU, NormalGr], (instregex "LAX(G)?$")>;
 628: 
 629: // Test and set
 630: def : InstRW<[WLat2LSU, FXb, LSU, EndGroup], (instregex "TS$")>;
 631: 
 632: // Compare and swap
 633: def : InstRW<[WLat3LSU, WLat3LSU, FXa, FXb, LSU, GroupAlone],
 634:              (instregex "CS(G|Y)?$")>;
 635: 
 636: // Compare double and swap
 637: def : InstRW<[WLat6LSU, WLat6LSU, FXa3, FXb2, LSU, GroupAlone2],
 638:              (instregex "CDS(Y)?$")>;
 639: def : InstRW<[WLat15, WLat15, FXa2, FXb4, LSU3,
 640:               GroupAlone3], (instregex "CDSG$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 641-680 / 第 641-680 行
```tablegen
 641: 
 642: // Compare and swap and store
 643: def : InstRW<[WLat30, MCD], (instregex "CSST$")>;
 644: 
 645: // Perform locked operation
 646: def : InstRW<[WLat30, MCD], (instregex "PLO$")>;
 647: 
 648: // Load/store pair from/to quadword
 649: def : InstRW<[WLat4LSU, LSU2, GroupAlone], (instregex "LPQ$")>;
 650: def : InstRW<[WLat1, FXb2, LSU, GroupAlone], (instregex "STPQ$")>;
 651: 
 652: // Load pair disjoint
 653: def : InstRW<[WLat1LSU, WLat1LSU, LSU2, GroupAlone], (instregex "LPD(G)?$")>;
 654: 
 655: // Compare and load
 656: def : InstRW<[WLat30, MCD], (instregex "CAL(G|GF)?$")>;
 657: 
 658: // Perform functions with concurrent results
 659: def : InstRW<[WLat30, MCD], (instregex "PFCR$")>;
 660: 
 661: //===----------------------------------------------------------------------===//
 662: // Translate and convert
 663: //===----------------------------------------------------------------------===//
 664: 
 665: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "TR$")>;
 666: def : InstRW<[WLat30, WLat30, WLat30, FXa3, LSU2, GroupAlone2],
 667:              (instregex "TRT$")>;
 668: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRTR$")>;
 669: def : InstRW<[WLat30, WLat30, MCD], (instregex "TRE$")>;
 670: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TRT(R)?E(Opt)?$")>;
 671: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "TR(T|O)(T|O)(Opt)?$")>;
 672: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 673:              (instregex "CU(12|14|21|24|41|42)(Opt)?$")>;
 674: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "(CUUTF|CUTFU)(Opt)?$")>;
 675: 
 676: //===----------------------------------------------------------------------===//
 677: // Message-security assist
 678: //===----------------------------------------------------------------------===//
 679: 
 680: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD],
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 681-720 / 第 681-720 行
```tablegen
 681:              (instregex "KM(C|F|O|CTR|A)?$")>;
 682: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 683:              (instregex "(KIMD|KLMD|KMAC|KDSA)(Opt)?$")>;
 684: def : InstRW<[WLat30, WLat30, WLat30, MCD],
 685:              (instregex "(PCC|PPNO|PRNO)$")>;
 686: 
 687: //===----------------------------------------------------------------------===//
 688: // Guarded storage
 689: //===----------------------------------------------------------------------===//
 690: 
 691: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LGG$")>;
 692: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LLGFSG$")>;
 693: def : InstRW<[WLat30, MCD], (instregex "(L|ST)GSC$")>;
 694: 
 695: //===----------------------------------------------------------------------===//
 696: // Decimal arithmetic
 697: //===----------------------------------------------------------------------===//
 698: 
 699: def : InstRW<[WLat20, RegReadAdv, FXb, VecDF2, LSU2, GroupAlone2],
 700:              (instregex "CVBG$")>;
 701: def : InstRW<[WLat20, RegReadAdv, FXb, VecDF, LSU, GroupAlone2],
 702:              (instregex "CVB(Y)?$")>;
 703: def : InstRW<[WLat1, FXb3, VecDF4, LSU, GroupAlone3], (instregex "CVDG$")>;
 704: def : InstRW<[WLat1, FXb2, VecDF, LSU, GroupAlone2], (instregex "CVD(Y)?$")>;
 705: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "MV(N|O|Z)$")>;
 706: def : InstRW<[WLat1, LSU5, GroupAlone], (instregex "(PACK|PKA|PKU)$")>;
 707: def : InstRW<[WLat12, LSU5, GroupAlone], (instregex "UNPK(A|U)$")>;
 708: def : InstRW<[WLat1, FXb, LSU2, Cracked], (instregex "UNPK$")>;
 709: 
 710: def : InstRW<[WLat5LSU, FXb, VecDFX, LSU3, GroupAlone2],
 711:              (instregex "(A|S|ZA)P$")>;
 712: def : InstRW<[WLat1, FXb, VecDFX2, LSU3, GroupAlone2], (instregex "MP$")>;
 713: def : InstRW<[WLat1, FXb, VecDFX4, LSU3, GroupAlone2], (instregex "DP$")>;
 714: def : InstRW<[WLat15, FXb, VecDFX2, LSU2, GroupAlone3], (instregex "SRP$")>;
 715: def : InstRW<[WLat8, VecDFX, LSU, LSU, GroupAlone], (instregex "CP$")>;
 716: def : InstRW<[WLat3LSU, VecDFX, LSU, Cracked], (instregex "TP$")>;
 717: def : InstRW<[WLat30, MCD], (instregex "ED(MK)?$")>;
 718: 
 719: //===----------------------------------------------------------------------===//
 720: // Access registers
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: //===----------------------------------------------------------------------===//
 722: 
 723: // Extract/set/copy access register
 724: def : InstRW<[WLat3, LSU, NormalGr], (instregex "(EAR|SAR|CPYA)$")>;
 725: 
 726: // Load address extended
 727: def : InstRW<[WLat5, LSU, FXa, Cracked], (instregex "LAE(Y)?$")>;
 728: 
 729: // Load/store access multiple (not modeled precisely)
 730: def : InstRW<[WLat20, WLat20, LSU5, GroupAlone], (instregex "LAM(Y)?$")>;
 731: def : InstRW<[WLat1, LSU5, FXb, GroupAlone2], (instregex "STAM(Y)?$")>;
 732: 
 733: //===----------------------------------------------------------------------===//
 734: // Program mask and addressing mode
 735: //===----------------------------------------------------------------------===//
 736: 
 737: // Insert Program Mask
 738: def : InstRW<[WLat3, FXa, EndGroup], (instregex "IPM$")>;
 739: 
 740: // Set Program Mask
 741: def : InstRW<[WLat3, LSU, EndGroup], (instregex "SPM$")>;
 742: 
 743: // Branch and link
 744: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "BAL(R)?$")>;
 745: 
 746: // Test addressing mode
 747: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TAM$")>;
 748: 
 749: // Set addressing mode
 750: def : InstRW<[WLat1, FXb, EndGroup], (instregex "SAM(24|31|64)$")>;
 751: 
 752: // Branch (and save) and set mode.
 753: def : InstRW<[WLat1, FXa, FXb, GroupAlone], (instregex "BSM$")>;
 754: def : InstRW<[WLat1, FXa2, FXb, GroupAlone], (instregex "BASSM$")>;
 755: 
 756: //===----------------------------------------------------------------------===//
 757: // Transactional execution
 758: //===----------------------------------------------------------------------===//
 759: 
 760: // Transaction begin
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 761-800 / 第 761-800 行
```tablegen
 761: def : InstRW<[WLat9, LSU2, FXb5, GroupAlone2], (instregex "TBEGIN(C)?$")>;
 762: 
 763: // Transaction end
 764: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "TEND$")>;
 765: 
 766: // Transaction abort
 767: def : InstRW<[WLat30, MCD], (instregex "TABORT$")>;
 768: 
 769: // Extract Transaction Nesting Depth
 770: def : InstRW<[WLat1, FXa, NormalGr], (instregex "ETND$")>;
 771: 
 772: // Nontransactional store
 773: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "NTSTG$")>;
 774: 
 775: //===----------------------------------------------------------------------===//
 776: // Processor assist
 777: //===----------------------------------------------------------------------===//
 778: 
 779: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "PPA$")>;
 780: 
 781: //===----------------------------------------------------------------------===//
 782: // Miscellaneous Instructions.
 783: //===----------------------------------------------------------------------===//
 784: 
 785: // Count leading/trailing zeros.
 786: def : InstRW<[WLat3, FXa, NormalGr], (instregex "C(L|T)ZG$")>;
 787: 
 788: // Find leftmost one
 789: def : InstRW<[WLat5, WLat5, FXa2, GroupAlone], (instregex "FLOGR$")>;
 790: 
 791: // Population count
 792: def : InstRW<[WLat3, WLat3, FXa, NormalGr], (instregex "POPCNT(Opt)?$")>;
 793: 
 794: // Bit deposit and bit extract.
 795: def : InstRW<[WLat4, FXa, NormalGr], (instregex "(BDEPG|BEXTG)$")>;
 796: 
 797: // String instructions
 798: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "SRST(U)?$")>;
 799: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CUSE$")>;
 800: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: // Various complex instructions
 802: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CFC$")>;
 803: def : InstRW<[WLat30, WLat30, WLat30, WLat30, WLat30, WLat30, MCD],
 804:              (instregex "UPT$")>;
 805: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "CKSM$")>;
 806: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "CMPSC$")>;
 807: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "SORTL$")>;
 808: def : InstRW<[WLat30, WLat30, WLat30, WLat30, MCD], (instregex "DFLTCC$")>;
 809: def : InstRW<[WLat30, WLat30, MCD], (instregex "NNPA$")>;
 810: 
 811: // Execute
 812: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "EX(RL)?$")>;
 813: 
 814: //===----------------------------------------------------------------------===//
 815: // .insn directive instructions
 816: //===----------------------------------------------------------------------===//
 817: 
 818: // An "empty" sched-class will be assigned instead of the "invalid sched-class".
 819: // getNumDecoderSlots() will then return 1 instead of 0.
 820: def : InstRW<[], (instregex "Insn.*")>;
 821: 
 822: 
 823: // ----------------------------- Floating point ----------------------------- //
 824: 
 825: //===----------------------------------------------------------------------===//
 826: // FP: Move instructions
 827: //===----------------------------------------------------------------------===//
 828: 
 829: // Load zero
 830: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LZ(DR|ER|ER_16)$")>;
 831: def : InstRW<[WLat2, FXb2, Cracked], (instregex "LZXR$")>;
 832: 
 833: // Load
 834: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "LER(16)?$")>;
 835: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LD(R|R16|R32|GR)$")>;
 836: def : InstRW<[WLat3, FXb, NormalGr], (instregex "LGDR$")>;
 837: def : InstRW<[WLat2, FXb2, GroupAlone], (instregex "LXR$")>;
 838: 
 839: // Load and Test
 840: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "LT(E|D)BR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXBR$")>;
 842: 
 843: // Copy sign
 844: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "CPSDR(d|s|h)(d|s|h)$")>;
 845: 
 846: //===----------------------------------------------------------------------===//
 847: // FP: Load instructions
 848: //===----------------------------------------------------------------------===//
 849: 
 850: def : InstRW<[WLat2LSU, VecXsPm, LSU, NormalGr], (instregex "L(E16|E)(Y)?$")>;
 851: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LD(Y|E32)?$")>;
 852: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LX$")>;
 853: 
 854: //===----------------------------------------------------------------------===//
 855: // FP: Store instructions
 856: //===----------------------------------------------------------------------===//
 857: 
 858: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "ST(E16|E|D)(Y)?$")>;
 859: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "STX$")>;
 860: 
 861: //===----------------------------------------------------------------------===//
 862: // FP: Conversion instructions
 863: //===----------------------------------------------------------------------===//
 864: 
 865: // Load rounded
 866: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "LEDBR(A)?$")>;
 867: def : InstRW<[WLat9, VecDF2, NormalGr], (instregex "L(E|D)XBR(A)?$")>;
 868: 
 869: // Load lengthened
 870: def : InstRW<[WLat6LSU, VecBF, LSU, NormalGr], (instregex "LDEB$")>;
 871: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "LDEBR$")>;
 872: def : InstRW<[WLat7LSU, VecBF4, LSU, GroupAlone], (instregex "LX(E|D)B$")>;
 873: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "LX(E|D)BR$")>;
 874: 
 875: // Convert from fixed / logical
 876: def : InstRW<[WLat7, FXb, VecBF, Cracked], (instregex "C(E|D)(F|G)BR(A)?$")>;
 877: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)BR(A)?$")>;
 878: def : InstRW<[WLat7, FXb, VecBF, Cracked], (instregex "C(E|D)L(F|G)BR$")>;
 879: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CXL(F|G)BR$")>;
 880: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 881-920 / 第 881-920 行
```tablegen
 881: // Convert to fixed / logical
 882: def : InstRW<[WLat9, WLat9, FXb, VecBF, Cracked],
 883:              (instregex "C(F|G)(E|D)BR(A)?$")>;
 884: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked],
 885:              (instregex "C(F|G)XBR(A)?$")>;
 886: def : InstRW<[WLat9, WLat9, FXb, VecBF, GroupAlone], (instregex "CLFEBR$")>;
 887: def : InstRW<[WLat9, WLat9, FXb, VecBF, Cracked], (instregex "CLFDBR$")>;
 888: def : InstRW<[WLat9, WLat9, FXb, VecBF, Cracked], (instregex "CLG(E|D)BR$")>;
 889: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked], (instregex "CL(F|G)XBR$")>;
 890: 
 891: //===----------------------------------------------------------------------===//
 892: // FP: Unary arithmetic
 893: //===----------------------------------------------------------------------===//
 894: 
 895: // Load Complement / Negative / Positive
 896: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "L(C|N|P)(E|D)BR$")>;
 897: def : InstRW<[WLat1, FXb, NormalGr], (instregex "L(C|N|P)DFR(_32|_16)?$")>;
 898: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "L(C|N|P)XBR$")>;
 899: 
 900: // Square root
 901: def : InstRW<[WLat30, VecFPd, LSU, NormalGr], (instregex "SQ(E|D)B$")>;
 902: def : InstRW<[WLat20, VecFPd20, NormalGr], (instregex "SQEBR$")>;
 903: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "SQDBR$")>;
 904: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "SQXBR$")>;
 905: 
 906: // Load FP integer
 907: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "FI(E|D)BR(A)?$")>;
 908: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXBR(A)?$")>;
 909: 
 910: //===----------------------------------------------------------------------===//
 911: // FP: Binary arithmetic
 912: //===----------------------------------------------------------------------===//
 913: 
 914: // Addition
 915: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
 916:              (instregex "A(E|D)B$")>;
 917: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "A(E|D)BR$")>;
 918: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXBR$")>;
 919: 
 920: // Subtraction
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
 922:              (instregex "S(E|D)B$")>;
 923: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "S(E|D)BR$")>;
 924: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXBR$")>;
 925: 
 926: // Multiply
 927: def : InstRW<[WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
 928:              (instregex "M(D|DE|EE)B$")>;
 929: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "M(D|DE|EE)BR$")>;
 930: def : InstRW<[WLat7LSU, RegReadAdv, VecBF4, LSU, GroupAlone],
 931:              (instregex "MXDB$")>;
 932: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "MXDBR$")>;
 933: def : InstRW<[WLat20, VecDF4, GroupAlone], (instregex "MXBR$")>;
 934: 
 935: // Multiply and add / subtract
 936: def : InstRW<[WLat6LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
 937:              (instregex "M(A|S)EB$")>;
 938: def : InstRW<[WLat6, VecBF, GroupAlone], (instregex "M(A|S)EBR$")>;
 939: def : InstRW<[WLat6LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
 940:              (instregex "M(A|S)DB$")>;
 941: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "M(A|S)DBR$")>;
 942: 
 943: // Division
 944: def : InstRW<[WLat20, RegReadAdv, VecFPd20, LSU, NormalGr], (instregex "DEB$")>;
 945: def : InstRW<[WLat30, RegReadAdv, VecFPd, LSU, NormalGr], (instregex "DDB$")>;
 946: def : InstRW<[WLat20, VecFPd20, NormalGr], (instregex "DEBR$")>;
 947: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "DDBR$")>;
 948: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "DXBR$")>;
 949: 
 950: // Divide to integer
 951: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "DI(E|D)BR$")>;
 952: 
 953: //===----------------------------------------------------------------------===//
 954: // FP: Comparisons
 955: //===----------------------------------------------------------------------===//
 956: 
 957: // Compare
 958: def : InstRW<[WLat3LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
 959:              (instregex "(K|C)(E|D)B$")>;
 960: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "(K|C)(E|D)BR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: def : InstRW<[WLat9, VecDF2, GroupAlone], (instregex "(K|C)XBR$")>;
 962: 
 963: // Test Data Class
 964: def : InstRW<[WLat5, LSU, VecXsPm, NormalGr], (instregex "TC(E|D)B$")>;
 965: def : InstRW<[WLat10, LSU, VecDF4, GroupAlone], (instregex "TCXB$")>;
 966: 
 967: //===----------------------------------------------------------------------===//
 968: // FP: Floating-point control register instructions
 969: //===----------------------------------------------------------------------===//
 970: 
 971: def : InstRW<[WLat4, FXa, LSU, GroupAlone], (instregex "EFPC$")>;
 972: def : InstRW<[WLat1, FXb, LSU, GroupAlone], (instregex "STFPC$")>;
 973: def : InstRW<[WLat3, LSU, GroupAlone], (instregex "SFPC$")>;
 974: def : InstRW<[WLat3LSU, LSU2, GroupAlone], (instregex "LFPC$")>;
 975: def : InstRW<[WLat30, MCD], (instregex "SFASR$")>;
 976: def : InstRW<[WLat30, MCD], (instregex "LFAS$")>;
 977: def : InstRW<[WLat3, FXb, GroupAlone], (instregex "SRNM(B|T)?$")>;
 978: 
 979: 
 980: // --------------------- Hexadecimal floating point ------------------------- //
 981: 
 982: //===----------------------------------------------------------------------===//
 983: // HFP: Move instructions
 984: //===----------------------------------------------------------------------===//
 985: 
 986: // Load and Test
 987: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "LT(E|D)R$")>;
 988: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXR$")>;
 989: 
 990: //===----------------------------------------------------------------------===//
 991: // HFP: Conversion instructions
 992: //===----------------------------------------------------------------------===//
 993: 
 994: // Load rounded
 995: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "(LEDR|LRER)$")>;
 996: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "LEXR$")>;
 997: def : InstRW<[WLat9, VecDF2, NormalGr], (instregex "(LDXR|LRDR)$")>;
 998: 
 999: // Load lengthened
1000: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "LDE$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LDER$")>;
1002: def : InstRW<[WLat7LSU, VecBF4, LSU, GroupAlone], (instregex "LX(E|D)$")>;
1003: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "LX(E|D)R$")>;
1004: 
1005: // Convert from fixed
1006: def : InstRW<[WLat7, FXb, VecBF, Cracked], (instregex "C(E|D)(F|G)R$")>;
1007: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "CX(F|G)R$")>;
1008: 
1009: // Convert to fixed
1010: def : InstRW<[WLat9, WLat9, FXb, VecBF, Cracked], (instregex "C(F|G)(E|D)R$")>;
1011: def : InstRW<[WLat12, WLat12, FXb, VecDF2, Cracked], (instregex "C(F|G)XR$")>;
1012: 
1013: // Convert BFP to HFP / HFP to BFP.
1014: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "THD(E)?R$")>;
1015: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "TB(E)?DR$")>;
1016: 
1017: //===----------------------------------------------------------------------===//
1018: // HFP: Unary arithmetic
1019: //===----------------------------------------------------------------------===//
1020: 
1021: // Load Complement / Negative / Positive
1022: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "L(C|N|P)(E|D)R$")>;
1023: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "L(C|N|P)XR$")>;
1024: 
1025: // Halve
1026: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "H(E|D)R$")>;
1027: 
1028: // Square root
1029: def : InstRW<[WLat30, VecFPd, LSU, NormalGr], (instregex "SQ(E|D)$")>;
1030: def : InstRW<[WLat20, VecFPd20, NormalGr], (instregex "SQER$")>;
1031: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "SQDR$")>;
1032: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "SQXR$")>;
1033: 
1034: // Load FP integer
1035: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "FI(E|D)R$")>;
1036: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXR$")>;
1037: 
1038: //===----------------------------------------------------------------------===//
1039: // HFP: Binary arithmetic
1040: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: 
1042: // Addition
1043: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
1044:              (instregex "A(E|D|U|W)$")>;
1045: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "A(E|D|U|W)R$")>;
1046: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXR$")>;
1047: 
1048: // Subtraction
1049: def : InstRW<[WLat6LSU, WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
1050:              (instregex "S(E|D|U|W)$")>;
1051: def : InstRW<[WLat6, WLat6, VecBF, NormalGr], (instregex "S(E|D|U|W)R$")>;
1052: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXR$")>;
1053: 
1054: // Multiply
1055: def : InstRW<[WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
1056:              (instregex "M(D|DE|E|EE)$")>;
1057: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "M(D|DE|E|EE)R$")>;
1058: def : InstRW<[WLat7LSU, RegReadAdv, VecBF4, LSU, GroupAlone],
1059:              (instregex "MXD$")>;
1060: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "MXDR$")>;
1061: def : InstRW<[WLat20, VecDF4, GroupAlone], (instregex "MXR$")>;
1062: def : InstRW<[WLat7LSU, RegReadAdv, VecBF4, LSU, GroupAlone], (instregex "MY$")>;
1063: def : InstRW<[WLat6LSU, RegReadAdv, VecBF2, LSU, GroupAlone],
1064:              (instregex "MY(H|L)$")>;
1065: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "MYR$")>;
1066: def : InstRW<[WLat6, VecBF, GroupAlone], (instregex "MY(H|L)R$")>;
1067: 
1068: // Multiply and add / subtract
1069: def : InstRW<[WLat6LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
1070:              (instregex "M(A|S)(E|D)$")>;
1071: def : InstRW<[WLat6, VecBF, GroupAlone], (instregex "M(A|S)(E|D)R$")>;
1072: def : InstRW<[WLat7LSU, RegReadAdv, RegReadAdv, VecBF4, LSU, GroupAlone],
1073:              (instregex "MAY$")>;
1074: def : InstRW<[WLat6LSU, RegReadAdv, RegReadAdv, VecBF2, LSU, GroupAlone],
1075:              (instregex "MAY(H|L)$")>;
1076: def : InstRW<[WLat7, VecBF4, GroupAlone], (instregex "MAYR$")>;
1077: def : InstRW<[WLat6, VecBF, GroupAlone], (instregex "MAY(H|L)R$")>;
1078: 
1079: // Division
1080: def : InstRW<[WLat20, RegReadAdv, VecFPd20, LSU, NormalGr], (instregex "DE$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: def : InstRW<[WLat30, RegReadAdv, VecFPd, LSU, NormalGr], (instregex "DD$")>;
1082: def : InstRW<[WLat20, VecFPd20, NormalGr], (instregex "DER$")>;
1083: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "DDR$")>;
1084: def : InstRW<[WLat30, VecFPd, GroupAlone], (instregex "DXR$")>;
1085: 
1086: //===----------------------------------------------------------------------===//
1087: // HFP: Comparisons
1088: //===----------------------------------------------------------------------===//
1089: 
1090: // Compare
1091: def : InstRW<[WLat6LSU, RegReadAdv, VecBF, LSU, NormalGr],
1092:              (instregex "C(E|D)$")>;
1093: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "C(E|D)R$")>;
1094: def : InstRW<[WLat10, VecDF2, GroupAlone], (instregex "CXR$")>;
1095: 
1096: 
1097: // ------------------------ Decimal floating point -------------------------- //
1098: 
1099: //===----------------------------------------------------------------------===//
1100: // DFP: Move instructions
1101: //===----------------------------------------------------------------------===//
1102: 
1103: // Load and Test
1104: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "LTDTR$")>;
1105: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "LTXTR$")>;
1106: 
1107: //===----------------------------------------------------------------------===//
1108: // DFP: Conversion instructions
1109: //===----------------------------------------------------------------------===//
1110: 
1111: // Load rounded
1112: def : InstRW<[WLat15, VecDF, NormalGr], (instregex "LEDTR$")>;
1113: def : InstRW<[WLat15, VecDF2, NormalGr], (instregex "LDXTR$")>;
1114: 
1115: // Load lengthened
1116: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "LDETR$")>;
1117: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "LXDTR$")>;
1118: 
1119: // Convert from fixed / logical
1120: def : InstRW<[WLat15, FXb, VecDF, Cracked], (instregex "CDFTR(A)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: def : InstRW<[WLat20, FXb, VecDF, Cracked], (instregex "CDGTR(A)?$")>;
1122: def : InstRW<[WLat15, FXb, VecDF4, GroupAlone2], (instregex "CXFTR(A)?$")>;
1123: def : InstRW<[WLat20, FXb, VecDF4, GroupAlone2], (instregex "CXGTR(A)?$")>;
1124: def : InstRW<[WLat15, FXb, VecDF, Cracked], (instregex "CDLFTR$")>;
1125: def : InstRW<[WLat20, FXb, VecDF, Cracked], (instregex "CDLGTR$")>;
1126: def : InstRW<[WLat15, FXb, VecDF4, GroupAlone2], (instregex "CXLFTR$")>;
1127: def : InstRW<[WLat20, FXb, VecDF4, GroupAlone2], (instregex "CXLGTR$")>;
1128: 
1129: // Convert to fixed / logical
1130: def : InstRW<[WLat20, WLat20, FXb, VecDF, Cracked],
1131:              (instregex "C(F|G)DTR(A)?$")>;
1132: def : InstRW<[WLat20, WLat20, FXb, VecDF2, Cracked],
1133:              (instregex "C(F|G)XTR(A)?$")>;
1134: def : InstRW<[WLat20, WLat20, FXb, VecDF, Cracked], (instregex "CL(F|G)DTR$")>;
1135: def : InstRW<[WLat20, WLat20, FXb, VecDF2, Cracked], (instregex "CL(F|G)XTR$")>;
1136: 
1137: // Convert from / to signed / unsigned packed
1138: def : InstRW<[WLat9, FXb, VecDF, Cracked], (instregex "CD(S|U)TR$")>;
1139: def : InstRW<[WLat12, FXb2, VecDF4, GroupAlone2], (instregex "CX(S|U)TR$")>;
1140: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "C(S|U)DTR$")>;
1141: def : InstRW<[WLat15, FXb2, VecDF4, GroupAlone2], (instregex "C(S|U)XTR$")>;
1142: 
1143: // Convert from / to zoned
1144: def : InstRW<[WLat8LSU, LSU, VecDF, Cracked], (instregex "CDZT$")>;
1145: def : InstRW<[WLat16LSU, LSU2, VecDF4, GroupAlone3], (instregex "CXZT$")>;
1146: def : InstRW<[WLat1, FXb, LSU, VecDF, Cracked], (instregex "CZDT$")>;
1147: def : InstRW<[WLat1, FXb, LSU, VecDF2, GroupAlone], (instregex "CZXT$")>;
1148: 
1149: // Convert from / to packed
1150: def : InstRW<[WLat8LSU, LSU, VecDF, Cracked], (instregex "CDPT$")>;
1151: def : InstRW<[WLat16LSU, LSU2, VecDF4, GroupAlone3], (instregex "CXPT$")>;
1152: def : InstRW<[WLat1, FXb, LSU, VecDF, Cracked], (instregex "CPDT$")>;
1153: def : InstRW<[WLat1, FXb, LSU, VecDF2, GroupAlone], (instregex "CPXT$")>;
1154: 
1155: // Perform floating-point operation
1156: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "PFPO$")>;
1157: 
1158: //===----------------------------------------------------------------------===//
1159: // DFP: Unary arithmetic
1160: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161: 
1162: // Load FP integer
1163: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "FIDTR$")>;
1164: def : InstRW<[WLat10, VecDF4, GroupAlone], (instregex "FIXTR$")>;
1165: 
1166: // Extract biased exponent
1167: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "EEDTR$")>;
1168: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "EEXTR$")>;
1169: 
1170: // Extract significance
1171: def : InstRW<[WLat11, FXb, VecDF, Cracked], (instregex "ESDTR$")>;
1172: def : InstRW<[WLat12, FXb, VecDF2, Cracked], (instregex "ESXTR$")>;
1173: 
1174: //===----------------------------------------------------------------------===//
1175: // DFP: Binary arithmetic
1176: //===----------------------------------------------------------------------===//
1177: 
1178: // Addition
1179: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "ADTR(A)?$")>;
1180: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "AXTR(A)?$")>;
1181: 
1182: // Subtraction
1183: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "SDTR(A)?$")>;
1184: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "SXTR(A)?$")>;
1185: 
1186: // Multiply
1187: def : InstRW<[WLat20, VecDF, NormalGr], (instregex "MDTR(A)?$")>;
1188: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "MXTR(A)?$")>;
1189: 
1190: // Division
1191: def : InstRW<[WLat30, VecDF, NormalGr], (instregex "DDTR(A)?$")>;
1192: def : InstRW<[WLat30, VecDF4, GroupAlone], (instregex "DXTR(A)?$")>;
1193: 
1194: // Quantize
1195: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "QADTR$")>;
1196: def : InstRW<[WLat10, WLat10, VecDF4, GroupAlone], (instregex "QAXTR$")>;
1197: 
1198: // Reround
1199: def : InstRW<[WLat9, WLat9, FXb, VecDF, Cracked], (instregex "RRDTR$")>;
1200: def : InstRW<[WLat11, WLat11, FXb, VecDF4, GroupAlone2], (instregex "RRXTR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201: 
1202: // Shift significand left/right
1203: def : InstRW<[WLat11LSU, LSU, VecDF, GroupAlone], (instregex "S(L|R)DT$")>;
1204: def : InstRW<[WLat11LSU, LSU, VecDF4, GroupAlone], (instregex "S(L|R)XT$")>;
1205: 
1206: // Insert biased exponent
1207: def : InstRW<[WLat9, FXb, VecDF, Cracked], (instregex "IEDTR$")>;
1208: def : InstRW<[WLat11, FXb, VecDF4, GroupAlone2], (instregex "IEXTR$")>;
1209: 
1210: //===----------------------------------------------------------------------===//
1211: // DFP: Comparisons
1212: //===----------------------------------------------------------------------===//
1213: 
1214: // Compare
1215: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "(K|C)DTR$")>;
1216: def : InstRW<[WLat9, VecDF2, GroupAlone], (instregex "(K|C)XTR$")>;
1217: 
1218: // Compare biased exponent
1219: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "CEDTR$")>;
1220: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "CEXTR$")>;
1221: 
1222: // Test Data Class/Group
1223: def : InstRW<[WLat15, LSU, VecDF, NormalGr], (instregex "TD(C|G)(E|D)T$")>;
1224: def : InstRW<[WLat15, LSU, VecDF2, GroupAlone], (instregex "TD(C|G)XT$")>;
1225: 
1226: 
1227: // --------------------------------- Vector --------------------------------- //
1228: 
1229: //===----------------------------------------------------------------------===//
1230: // Vector: Move instructions
1231: //===----------------------------------------------------------------------===//
1232: 
1233: def : InstRW<[WLat1, FXb, NormalGr], (instregex "VLR(16|32|64)?$")>;
1234: def : InstRW<[WLat3, FXb, NormalGr], (instregex "VLGV(B|F|G|H)?$")>;
1235: def : InstRW<[WLat1, FXb, NormalGr], (instregex "VLVG(B|F|G|H)?$")>;
1236: def : InstRW<[WLat3, FXb, NormalGr], (instregex "VLVGP(32)?$")>;
1237: 
1238: //===----------------------------------------------------------------------===//
1239: // Vector: Immediate instructions
1240: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241: 
1242: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VZERO$")>;
1243: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VONE$")>;
1244: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VGBM$")>;
1245: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VGM(B|F|G|H)?$")>;
1246: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VREPI(B|F|G|H)?$")>;
1247: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLEI(B|F|G|H)$")>;
1248: 
1249: //===----------------------------------------------------------------------===//
1250: // Vector: Loads
1251: //===----------------------------------------------------------------------===//
1252: 
1253: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(Align)?$")>;
1254: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(L|BB)$")>;
1255: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VL(16|32|64)$")>;
1256: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLLEZ(B|F|G|H|LF)?$")>;
1257: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLREP(B|F|G|H)?$")>;
1258: def : InstRW<[WLat2LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
1259:              (instregex "VLE(B|F|G|H)$")>;
1260: def : InstRW<[WLat5LSU, RegReadAdv, FXb, LSU, VecXsPm, Cracked],
1261:              (instregex "VGE(F|G)$")>;
1262: def : InstRW<[WLat4LSU, WLat4LSU, LSU5, GroupAlone],
1263:              (instregex "VLM(Align)?$")>;
1264: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLRL(R)?$")>;
1265: 
1266: //===----------------------------------------------------------------------===//
1267: // Vector: Stores
1268: //===----------------------------------------------------------------------===//
1269: 
1270: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VST(Align|L|16|32|64)?$")>;
1271: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTE(F|G)$")>;
1272: def : InstRW<[WLat1, FXb, LSU, VecXsPm, Cracked], (instregex "VSTE(B|H)$")>;
1273: def : InstRW<[WLat1, LSU2, FXb3, GroupAlone2], (instregex "VSTM(Align)?$")>;
1274: def : InstRW<[WLat1, FXb2, LSU, Cracked], (instregex "VSCE(F|G)$")>;
1275: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTRL(R)?$")>;
1276: 
1277: //===----------------------------------------------------------------------===//
1278: // Vector: Byte swaps
1279: //===----------------------------------------------------------------------===//
1280: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLBR(H|F|G|Q)?$")>;
1282: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLER(H|F|G)?$")>;
1283: def : InstRW<[WLat2LSU, RegReadAdv, VecXsPm, LSU, NormalGr],
1284:              (instregex "VLEBR(H|F|G)$")>;
1285: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLLEBRZ(H|F|G|E)?$")>;
1286: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "VLBRREP(H|F|G)?$")>;
1287: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTBR(H|F|G|Q)?$")>;
1288: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTER(H|F|G)?$")>;
1289: def : InstRW<[WLat1, FXb, LSU, VecXsPm, Cracked], (instregex "VSTEBRH$")>;
1290: def : InstRW<[WLat1, FXb, LSU, NormalGr], (instregex "VSTEBR(F|G)$")>;
1291: 
1292: //===----------------------------------------------------------------------===//
1293: // Vector: Selects and permutes
1294: //===----------------------------------------------------------------------===//
1295: 
1296: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMRH(B|F|G|H)?$")>;
1297: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMRL(B|F|G|H)?$")>;
1298: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPERM$")>;
1299: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPDI$")>;
1300: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VBPERM$")>;
1301: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VREP(B|F|G|H)?$")>;
1302: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSEL$")>;
1303: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VBLEND(B|F|G|H|Q)?$")>;
1304: 
1305: //===----------------------------------------------------------------------===//
1306: // Vector: Widening and narrowing
1307: //===----------------------------------------------------------------------===//
1308: 
1309: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPK(F|G|H)?$")>;
1310: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPKS(F|G|H)?$")>;
1311: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VPKS(F|G|H)S$")>;
1312: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPKLS(F|G|H)?$")>;
1313: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VPKLS(F|G|H)S$")>;
1314: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSEG(B|F|H)?$")>;
1315: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VGEM(B|H|F|G|Q)?$")>;
1316: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPH(B|F|H|G)?$")>;
1317: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPL(B|F|G)?$")>;
1318: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPLH(B|F|H|G|W)?$")>;
1319: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VUPLL(B|F|H|G)?$")>;
1320: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1321-1360 / 第 1321-1360 行
```tablegen
1321: //===----------------------------------------------------------------------===//
1322: // Vector: Integer arithmetic
1323: //===----------------------------------------------------------------------===//
1324: 
1325: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VA(B|F|G|H|Q|C|CQ)?$")>;
1326: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VACC(B|F|G|H|Q|C|CQ)?$")>;
1327: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VAVG(B|F|G|H|Q)?$")>;
1328: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VAVGL(B|F|G|H|Q)?$")>;
1329: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VN(C|O|N|X)?$")>;
1330: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VO(C)?$")>;
1331: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VCKSM$")>;
1332: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCLZ(B|F|G|H|Q)?$")>;
1333: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCTZ(B|F|G|H|Q)?$")>;
1334: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VD(L)?(F|G|Q)?$")>;
1335: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VEVAL$")>;
1336: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VX$")>;
1337: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFM?$")>;
1338: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFMA(B|F|G|H)?$")>;
1339: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VGFM(B|F|G|H)$")>;
1340: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLC(B|F|G|H|Q)?$")>;
1341: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VLP(B|F|G|H|Q)?$")>;
1342: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMX(B|F|G|H|Q)?$")>;
1343: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMXL(B|F|G|H|Q)?$")>;
1344: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMN(B|F|G|H|Q)?$")>;
1345: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VMNL(B|F|G|H|Q)?$")>;
1346: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAL(B|F|G|Q)?$")>;
1347: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALE(B|F|H|G)?$")>;
1348: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALH(B|F|H|G|Q|W)?$")>;
1349: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMALO(B|F|H|G)?$")>;
1350: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAO(B|F|H|G)?$")>;
1351: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAE(B|F|H|G)?$")>;
1352: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMAH(B|F|H|G|Q)?$")>;
1353: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VME(B|F|H|G)?$")>;
1354: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMH(B|F|H|G|Q)?$")>;
1355: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VML(B|F|G|Q)?$")>;
1356: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLE(B|F|H|G)?$")>;
1357: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLH(B|F|H|G|Q|W)?$")>;
1358: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMLO(B|F|H|G)?$")>;
1359: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VMO(B|F|H|G)?$")>;
1360: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VMSL(G)?$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1361-1400 / 第 1361-1400 行
```tablegen
1361: 
1362: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VPOPCT(B|F|G|H)?$")>;
1363: 
1364: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VR(L)?(F|G|Q)?$")>;
1365: 
1366: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERLL(B|F|G|H)?$")>;
1367: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERLLV(B|F|G|H)?$")>;
1368: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VERIM(B|F|G|H)?$")>;
1369: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESL(B|F|G|H)?$")>;
1370: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESLV(B|F|G|H)?$")>;
1371: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRA(B|F|G|H)?$")>;
1372: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRAV(B|F|G|H)?$")>;
1373: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRL(B|F|G|H)?$")>;
1374: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VESRLV(B|F|G|H)?$")>;
1375: 
1376: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSL(DB)?$")>;
1377: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSLB$")>;
1378: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSR(A|L)$")>;
1379: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSR(A|L)B$")>;
1380: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSLD$")>;
1381: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSRD$")>;
1382: 
1383: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSB(I|IQ|CBI|CBIQ)?$")>;
1384: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VSCBI(B|F|G|H|Q)?$")>;
1385: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VS(F|G|H|Q)?$")>;
1386: 
1387: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUM(B|H)?$")>;
1388: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUMG(F|H)?$")>;
1389: def : InstRW<[WLat4, VecMul, NormalGr], (instregex "VSUMQ(F|G)?$")>;
1390: 
1391: //===----------------------------------------------------------------------===//
1392: // Vector: Integer comparison
1393: //===----------------------------------------------------------------------===//
1394: 
1395: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "VEC(B|F|G|H|Q)?$")>;
1396: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "VECL(B|F|G|H|Q)?$")>;
1397: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCEQ(B|F|G|H|Q)?$")>;
1398: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCEQ(B|F|G|H|Q)S$")>;
1399: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCH(B|F|G|H|Q)?$")>;
1400: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCH(B|F|G|H|Q)S$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1401-1440 / 第 1401-1440 行
```tablegen
1401: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCHL(B|F|G|H|Q)?$")>;
1402: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VCHL(B|F|G|H|Q)S$")>;
1403: def : InstRW<[WLat4, VecStr, NormalGr], (instregex "VTM$")>;
1404: 
1405: //===----------------------------------------------------------------------===//
1406: // Vector: Floating-point arithmetic
1407: //===----------------------------------------------------------------------===//
1408: 
1409: // Conversion and rounding
1410: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VCFP(S|L)$")>;
1411: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VCD(L)?G$")>;
1412: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VCD(L)?GB$")>;
1413: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WCD(L)?GB$")>;
1414: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VCE(L)?FB$")>;
1415: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WCE(L)?FB$")>;
1416: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VC(S|L)FP$")>;
1417: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VC(L)?GD$")>;
1418: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VC(L)?GDB$")>;
1419: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WC(L)?GDB$")>;
1420: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VC(L)?FEB$")>;
1421: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WC(L)?FEB$")>;
1422: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VL(DE|ED)$")>;
1423: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VL(DE|ED)B$")>;
1424: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WL(DE|ED)B$")>;
1425: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFL(L|R)$")>;
1426: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFL(LS|RD)$")>;
1427: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFL(LS|RD)$")>;
1428: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFLLD$")>;
1429: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "WFLRX$")>;
1430: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFI(DB)?$")>;
1431: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFIDB$")>;
1432: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFISB$")>;
1433: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFISB$")>;
1434: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "WFIXB$")>;
1435: 
1436: // Sign operations
1437: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VFPSO$")>;
1438: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FPSODB$")>;
1439: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FPSOSB$")>;
1440: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFPSOXB$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1441-1480 / 第 1441-1480 行
```tablegen
1441: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FL(C|N|P)DB$")>;
1442: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "(V|W)FL(C|N|P)SB$")>;
1443: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFL(C|N|P)XB$")>;
1444: 
1445: // Minimum / maximum
1446: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(MAX|MIN)$")>;
1447: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(MAX|MIN)DB$")>;
1448: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WF(MAX|MIN)DB$")>;
1449: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(MAX|MIN)SB$")>;
1450: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WF(MAX|MIN)SB$")>;
1451: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "WF(MAX|MIN)XB$")>;
1452: 
1453: // Test data class
1454: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFTCI$")>;
1455: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "(V|W)FTCIDB$")>;
1456: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "(V|W)FTCISB$")>;
1457: def : InstRW<[WLat3, WLat3, VecDFX, NormalGr], (instregex "WFTCIXB$")>;
1458: 
1459: // Add / subtract
1460: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(A|S)$")>;
1461: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(A|S)DB$")>;
1462: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WF(A|S)DB$")>;
1463: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(A|S)SB$")>;
1464: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WF(A|S)SB$")>;
1465: def : InstRW<[WLat8, VecDF, NormalGr], (instregex "WF(A|S)XB$")>;
1466: 
1467: // Multiply / multiply-and-add/subtract
1468: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFM(DB)?$")>;
1469: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WFM(D|S)B$")>;
1470: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VFMSB$")>;
1471: def : InstRW<[WLat20, VecDF, NormalGr], (instregex "WFMXB$")>;
1472: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(N)?M(A|S)$")>;
1473: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(N)?M(A|S)DB$")>;
1474: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WF(N)?M(A|S)DB$")>;
1475: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "VF(N)?M(A|S)SB$")>;
1476: def : InstRW<[WLat6, VecBF, NormalGr], (instregex "WF(N)?M(A|S)SB$")>;
1477: def : InstRW<[WLat20, VecDF, NormalGr], (instregex "WF(N)?M(A|S)XB$")>;
1478: 
1479: // Divide / square root
1480: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFD$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1481-1520 / 第 1481-1520 行
```tablegen
1481: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FDDB$")>;
1482: def : InstRW<[WLat20, VecFPd20, NormalGr], (instregex "WFDSB$")>;
1483: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFDSB$")>;
1484: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "WFDXB$")>;
1485: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFSQ$")>;
1486: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "(V|W)FSQDB$")>;
1487: def : InstRW<[WLat20, VecFPd20, NormalGr], (instregex "WFSQSB$")>;
1488: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "VFSQSB$")>;
1489: def : InstRW<[WLat30, VecFPd, NormalGr], (instregex "WFSQXB$")>;
1490: 
1491: //===----------------------------------------------------------------------===//
1492: // Vector: Floating-point comparison
1493: //===----------------------------------------------------------------------===//
1494: 
1495: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(C|K)(E|H|HE)$")>;
1496: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(C|K)(E|H|HE)DB$")>;
1497: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)DB$")>;
1498: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFK(E|H|HE)DB$")>;
1499: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VF(C|K)(E|H|HE)SB$")>;
1500: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)SB$")>;
1501: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "WFK(E|H|HE)SB$")>;
1502: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "WFC(E|H|HE)XB$")>;
1503: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "WFK(E|H|HE)XB$")>;
1504: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFC(E|H|HE)DBS$")>;
1505: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "VFK(E|H|HE)DBS$")>;
1506: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr],
1507:              (instregex "WF(C|K)(E|H|HE)DBS$")>;
1508: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr],
1509:              (instregex "VF(C|K)(E|H|HE)SBS$")>;
1510: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "WFC(E|H|HE)SBS$")>;
1511: def : InstRW<[WLat3, WLat3, VecXsPm, NormalGr], (instregex "WFK(E|H|HE)SBS$")>;
1512: def : InstRW<[WLat3, WLat3, VecDFX, NormalGr], (instregex "WFC(E|H|HE)XBS$")>;
1513: def : InstRW<[WLat3, WLat3, VecDFX, NormalGr], (instregex "WFK(E|H|HE)XBS$")>;
1514: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)$")>;
1515: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)DB$")>;
1516: def : InstRW<[WLat3, VecXsPm, NormalGr], (instregex "WF(C|K)SB$")>;
1517: def : InstRW<[WLat3, VecDFX, NormalGr], (instregex "WF(C|K)XB$")>;
1518: 
1519: //===----------------------------------------------------------------------===//
1520: // Vector: Floating-point insertion and extraction
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1521-1560 / 第 1521-1560 行
```tablegen
1521: //===----------------------------------------------------------------------===//
1522: 
1523: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LEFR(_16)?$")>;
1524: def : InstRW<[WLat3, FXb, NormalGr], (instregex "LFER(_16)?$")>;
1525: 
1526: //===----------------------------------------------------------------------===//
1527: // Vector: String instructions
1528: //===----------------------------------------------------------------------===//
1529: 
1530: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAE(B)?$")>;
1531: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAE(F|H)$")>;
1532: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VFAE(B|F|H)S$")>;
1533: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFAEZ(B|F|H)$")>;
1534: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VFAEZ(B|F|H)S$")>;
1535: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFEE(B|F|H|ZB|ZF|ZH)?$")>;
1536: def : InstRW<[WLat4, WLat4, VecStr, NormalGr],
1537:              (instregex "VFEE(B|F|H|ZB|ZF|ZH)S$")>;
1538: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VFENE(B|F|H|ZB|ZF|ZH)?$")>;
1539: def : InstRW<[WLat4, WLat4, VecStr, NormalGr],
1540:              (instregex "VFENE(B|F|H|ZB|ZF|ZH)S$")>;
1541: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VISTR(B|F|H)?$")>;
1542: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VISTR(B|F|H)S$")>;
1543: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VSTRC(B|F|H)?$")>;
1544: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRC(B|F|H)S$")>;
1545: def : InstRW<[WLat3, VecStr, NormalGr], (instregex "VSTRCZ(B|F|H)$")>;
1546: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRCZ(B|F|H)S$")>;
1547: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRS(B|F|H)?$")>;
1548: def : InstRW<[WLat4, WLat4, VecStr, NormalGr], (instregex "VSTRSZ(B|F|H)$")>;
1549: 
1550: //===----------------------------------------------------------------------===//
1551: // NNP assist instructions
1552: //===----------------------------------------------------------------------===//
1553: 
1554: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCFN$")>;
1555: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VCLFN(L|H)$")>;
1556: def : InstRW<[WLat2, VecXsPm, NormalGr], (instregex "VC(R)?NF$")>;
1557: 
1558: //===----------------------------------------------------------------------===//
1559: // Vector: Packed-decimal instructions
1560: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1561-1600 / 第 1561-1600 行
```tablegen
1561: 
1562: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "VLIP$")>;
1563: def : InstRW<[WLat6, VecDFX, LSU, GroupAlone2], (instregex "VPKZ$")>;
1564: def : InstRW<[WLat1, VecDFX, FXb, LSU2, GroupAlone2], (instregex "VUPKZ$")>;
1565: def : InstRW<[WLat20, WLat20, VecDF, FXb, GroupAlone],
1566:              (instregex "VCVB(G|Q)?(Opt)?$")>;
1567: def : InstRW<[WLat15, WLat15, VecDF, FXb, GroupAlone],
1568:              (instregex "VCVD(G|Q)?$")>;
1569: def : InstRW<[WLat4, WLat4, VecDFX, NormalGr], (instregex "V(A|S)P$")>;
1570: def : InstRW<[WLat30, WLat30, VecDF, GroupAlone], (instregex "VM(S)?P$")>;
1571: def : InstRW<[WLat30, WLat30, VecDF, GroupAlone], (instregex "V(D|R)P$")>;
1572: def : InstRW<[WLat30, WLat30, VecDF, GroupAlone], (instregex "VSDP$")>;
1573: def : InstRW<[WLat8, WLat8, VecDF, NormalGr], (instregex "VSRP(R)?$")>;
1574: def : InstRW<[WLat4, WLat4, VecDFX, NormalGr], (instregex "VPSOP$")>;
1575: def : InstRW<[WLat2, VecDFX, NormalGr], (instregex "V(T|C)(P|Z)(Opt)?$")>;
1576: 
1577: def : InstRW<[WLat20, VecDF, NormalGr], (instregex "VSCH(S|D|X)?P$")>;
1578: def : InstRW<[WLat30, VecDF, NormalGr], (instregex "VSCSHP$")>;
1579: def : InstRW<[WLat30, VecDF, NormalGr], (instregex "VCSPH$")>;
1580: def : InstRW<[WLat2, WLat2, VecXsPm, NormalGr], (instregex "VCLZDP$")>;
1581: def : InstRW<[WLat2, WLat2, VecDFX, NormalGr], (instregex "VPKZR$")>;
1582: def : InstRW<[WLat2, WLat2, VecDFX, NormalGr], (instregex "VUPKZH$")>;
1583: def : InstRW<[WLat2, WLat2, VecDFX, NormalGr], (instregex "VUPKZL$")>;
1584: 
1585: // -------------------------------- System ---------------------------------- //
1586: 
1587: //===----------------------------------------------------------------------===//
1588: // System: Program-Status Word Instructions
1589: //===----------------------------------------------------------------------===//
1590: 
1591: def : InstRW<[WLat30, WLat30, MCD], (instregex "EPSW$")>;
1592: def : InstRW<[WLat20, GroupAlone3], (instregex "LPSW(E)?(Y)?$")>;
1593: def : InstRW<[WLat3, FXa, GroupAlone], (instregex "IPK$")>;
1594: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SPKA$")>;
1595: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SSM$")>;
1596: def : InstRW<[WLat1, FXb, LSU, GroupAlone], (instregex "ST(N|O)SM$")>;
1597: def : InstRW<[WLat3, FXa, NormalGr], (instregex "IAC$")>;
1598: def : InstRW<[WLat1, LSU, EndGroup], (instregex "SAC(F)?$")>;
1599: 
1600: //===----------------------------------------------------------------------===//
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1601-1640 / 第 1601-1640 行
```tablegen
1601: // System: Control Register Instructions
1602: //===----------------------------------------------------------------------===//
1603: 
1604: def : InstRW<[WLat4LSU, WLat4LSU, LSU2, GroupAlone], (instregex "LCTL(G)?$")>;
1605: def : InstRW<[WLat1, LSU5, FXb, GroupAlone2], (instregex "STCT(L|G)$")>;
1606: def : InstRW<[LSULatency, LSU, NormalGr], (instregex "E(P|S)A(I)?R$")>;
1607: def : InstRW<[WLat30, MCD], (instregex "SSA(I)?R$")>;
1608: def : InstRW<[WLat30, MCD], (instregex "ESEA$")>;
1609: 
1610: //===----------------------------------------------------------------------===//
1611: // System: Prefix-Register Instructions
1612: //===----------------------------------------------------------------------===//
1613: 
1614: def : InstRW<[WLat30, MCD], (instregex "S(T)?PX$")>;
1615: 
1616: //===----------------------------------------------------------------------===//
1617: // System: Breaking-Event-Address-Register Instructions
1618: //===----------------------------------------------------------------------===//
1619: 
1620: def : InstRW<[WLat3LSU, LSU2, GroupAlone], (instregex "LBEAR$")>;
1621: def : InstRW<[WLat1, LSU2, FXb, GroupAlone], (instregex "STBEAR$")>;
1622: 
1623: //===----------------------------------------------------------------------===//
1624: // System: Storage-Key and Real Memory Instructions
1625: //===----------------------------------------------------------------------===//
1626: 
1627: def : InstRW<[WLat30, MCD], (instregex "ISKE$")>;
1628: def : InstRW<[WLat30, MCD], (instregex "IVSK$")>;
1629: def : InstRW<[WLat30, MCD], (instregex "SSKE(Opt)?$")>;
1630: def : InstRW<[WLat30, MCD], (instregex "RRB(E|M)$")>;
1631: def : InstRW<[WLat30, MCD], (instregex "IRBM$")>;
1632: def : InstRW<[WLat30, MCD], (instregex "PFMF$")>;
1633: def : InstRW<[WLat30, WLat30, MCD], (instregex "TB$")>;
1634: def : InstRW<[WLat30, MCD], (instregex "PGIN$")>;
1635: def : InstRW<[WLat30, MCD], (instregex "PGOUT$")>;
1636: 
1637: //===----------------------------------------------------------------------===//
1638: // System: Dynamic-Address-Translation Instructions
1639: //===----------------------------------------------------------------------===//
1640: 
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1641-1680 / 第 1641-1680 行
```tablegen
1641: def : InstRW<[WLat30, MCD], (instregex "IPTE(Opt)?(Opt)?$")>;
1642: def : InstRW<[WLat30, MCD], (instregex "IDTE(Opt)?$")>;
1643: def : InstRW<[WLat30, MCD], (instregex "RDP(Opt)?$")>;
1644: def : InstRW<[WLat30, MCD], (instregex "CRDTE(Opt)?$")>;
1645: def : InstRW<[WLat30, MCD], (instregex "PTLB$")>;
1646: def : InstRW<[WLat30, WLat30, MCD], (instregex "CSP(G)?$")>;
1647: def : InstRW<[WLat30, WLat30, WLat30, MCD], (instregex "LPTEA$")>;
1648: def : InstRW<[WLat30, WLat30, MCD], (instregex "LRA(Y|G)?$")>;
1649: def : InstRW<[WLat30, MCD], (instregex "STRAG$")>;
1650: def : InstRW<[WLat30, MCD], (instregex "LURA(G)?$")>;
1651: def : InstRW<[WLat30, MCD], (instregex "STUR(A|G)$")>;
1652: def : InstRW<[WLat30, MCD], (instregex "TPROT$")>;
1653: 
1654: //===----------------------------------------------------------------------===//
1655: // System: Memory-move Instructions
1656: //===----------------------------------------------------------------------===//
1657: 
1658: def : InstRW<[WLat4LSU, FXa2, FXb, LSU5, GroupAlone2], (instregex "MVC(K|P|S)$")>;
1659: def : InstRW<[WLat1, FXa, LSU5, GroupAlone2], (instregex "MVC(S|D)K$")>;
1660: def : InstRW<[WLat30, MCD], (instregex "MVCOS$")>;
1661: def : InstRW<[WLat30, MCD], (instregex "MVPG$")>;
1662: 
1663: //===----------------------------------------------------------------------===//
1664: // System: Address-Space Instructions
1665: //===----------------------------------------------------------------------===//
1666: 
1667: def : InstRW<[WLat30, MCD], (instregex "LASP$")>;
1668: def : InstRW<[WLat1, LSU, GroupAlone], (instregex "PALB$")>;
1669: def : InstRW<[WLat30, MCD], (instregex "PC$")>;
1670: def : InstRW<[WLat30, MCD], (instregex "PR$")>;
1671: def : InstRW<[WLat30, MCD], (instregex "PT(I)?$")>;
1672: def : InstRW<[WLat30, MCD], (instregex "RP$")>;
1673: def : InstRW<[WLat30, MCD], (instregex "BS(G|A)$")>;
1674: def : InstRW<[WLat30, MCD], (instregex "TAR$")>;
1675: 
1676: //===----------------------------------------------------------------------===//
1677: // System: Linkage-Stack Instructions
1678: //===----------------------------------------------------------------------===//
1679: 
1680: def : InstRW<[WLat30, MCD], (instregex "BAKR$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1681-1720 / 第 1681-1720 行
```tablegen
1681: def : InstRW<[WLat30, MCD], (instregex "EREG(G)?$")>;
1682: def : InstRW<[WLat30, WLat30, MCD], (instregex "(E|M)STA$")>;
1683: 
1684: //===----------------------------------------------------------------------===//
1685: // System: Time-Related Instructions
1686: //===----------------------------------------------------------------------===//
1687: 
1688: def : InstRW<[WLat30, MCD], (instregex "PTFF$")>;
1689: def : InstRW<[WLat30, MCD], (instregex "SCK(PF|C)?$")>;
1690: def : InstRW<[WLat1, LSU2, GroupAlone], (instregex "SPT$")>;
1691: def : InstRW<[WLat15, LSU3, FXa2, FXb, GroupAlone2], (instregex "STCK(F)?$")>;
1692: def : InstRW<[WLat20, LSU4, FXa2, FXb2, GroupAlone3], (instregex "STCKE$")>;
1693: def : InstRW<[WLat30, MCD], (instregex "STCKC$")>;
1694: def : InstRW<[WLat1, LSU2, FXb, Cracked], (instregex "STPT$")>;
1695: 
1696: //===----------------------------------------------------------------------===//
1697: // System: CPU-Related Instructions
1698: //===----------------------------------------------------------------------===//
1699: 
1700: def : InstRW<[WLat30, MCD], (instregex "STAP$")>;
1701: def : InstRW<[WLat30, MCD], (instregex "STIDP$")>;
1702: def : InstRW<[WLat30, WLat30, MCD], (instregex "STSI$")>;
1703: def : InstRW<[WLat30, WLat30, MCD], (instregex "STFL(E)?$")>;
1704: def : InstRW<[WLat30, MCD], (instregex "ECAG$")>;
1705: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECTG$")>;
1706: def : InstRW<[WLat30, MCD], (instregex "PTF$")>;
1707: def : InstRW<[WLat30, MCD], (instregex "PCKMO$")>;
1708: def : InstRW<[WLat30, WLat30, MCD], (instregex "QPACI$")>;
1709: 
1710: //===----------------------------------------------------------------------===//
1711: // System: Miscellaneous Instructions
1712: //===----------------------------------------------------------------------===//
1713: 
1714: def : InstRW<[WLat30, MCD], (instregex "SVC$")>;
1715: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "MC$")>;
1716: def : InstRW<[WLat30, MCD], (instregex "DIAG$")>;
1717: def : InstRW<[WLat1, FXb, NormalGr], (instregex "TRACE$")>;
1718: def : InstRW<[WLat1, FXb, GroupAlone], (instregex "TRACG$")>;
1719: def : InstRW<[WLat30, MCD], (instregex "TRAP(2|4)$")>;
1720: def : InstRW<[WLat30, MCD], (instregex "SIG(P|A)$")>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1721-1754 / 第 1721-1754 行
```tablegen
1721: def : InstRW<[WLat30, MCD], (instregex "SIE$")>;
1722: 
1723: //===----------------------------------------------------------------------===//
1724: // System: CPU-Measurement Facility Instructions
1725: //===----------------------------------------------------------------------===//
1726: 
1727: def : InstRW<[WLat1, FXb, NormalGr], (instregex "LPP$")>;
1728: def : InstRW<[WLat30, WLat30, MCD], (instregex "ECPGA$")>;
1729: def : InstRW<[WLat30, WLat30, MCD], (instregex "E(C|P)CTR$")>;
1730: def : InstRW<[WLat30, MCD], (instregex "LCCTL$")>;
1731: def : InstRW<[WLat30, MCD], (instregex "L(P|S)CTL$")>;
1732: def : InstRW<[WLat30, MCD], (instregex "Q(S|CTR)I$")>;
1733: def : InstRW<[WLat30, MCD], (instregex "S(C|P)CTR$")>;
1734: 
1735: //===----------------------------------------------------------------------===//
1736: // System: I/O Instructions
1737: //===----------------------------------------------------------------------===//
1738: 
1739: def : InstRW<[WLat30, MCD], (instregex "(C|H|R|X)SCH$")>;
1740: def : InstRW<[WLat30, MCD], (instregex "(M|S|ST|T)SCH$")>;
1741: def : InstRW<[WLat30, MCD], (instregex "RCHP$")>;
1742: def : InstRW<[WLat30, MCD], (instregex "SCHM$")>;
1743: def : InstRW<[WLat30, MCD], (instregex "STC(PS|RW)$")>;
1744: def : InstRW<[WLat30, MCD], (instregex "TPE?I$")>;
1745: def : InstRW<[WLat30, MCD], (instregex "SAL$")>;
1746: 
1747: //===----------------------------------------------------------------------===//
1748: // NOPs
1749: //===----------------------------------------------------------------------===//
1750: 
1751: def : InstRW<[WLat1, FXb, NormalGr], (instregex "NOP(R)?(Opt)?$")>;
1752: def : InstRW<[WLat1, VBU, NormalGr], (instregex "J(G)?NOP$")>;
1753: }
1754: 
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
