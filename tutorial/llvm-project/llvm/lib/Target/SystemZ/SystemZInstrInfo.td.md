# SystemZInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZInstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines SystemZ backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 SystemZ 后端元数据；具体而言，它定义或实现目标指令信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //===-- SystemZInstrInfo.td - General SystemZ instructions ----*- tblgen-*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: def IsTargetXPLINK64      : Predicate<"Subtarget->isTargetXPLINK64()">;
  10: def IsTargetELF           : Predicate<"Subtarget->isTargetELF()">;
  11: 
  12: //===----------------------------------------------------------------------===//
  13: // Stack allocation
  14: //===----------------------------------------------------------------------===//
  15: 
  16: // These pseudos carry values needed to compute the MaxcallFrameSize of the
  17: // function.  The callseq_start node requires the hasSideEffects flag.
  18: let usesCustomInserter = 1, hasNoSchedulingInfo = 1, hasSideEffects = 1 in {
  19:   def ADJCALLSTACKDOWN : Pseudo<(outs), (ins i64imm:$amt1, i64imm:$amt2),
  20:                                 [(callseq_start timm:$amt1, timm:$amt2)]>;
  21:   def ADJCALLSTACKUP   : Pseudo<(outs), (ins i64imm:$amt1, i64imm:$amt2),
  22:                                 [(callseq_end timm:$amt1, timm:$amt2)]>;
  23: }
  24: 
  25: // Takes as input the value of the stack pointer after a dynamic allocation
  26: // has been made.  Sets the output to the address of the dynamically-
  27: // allocated area itself, skipping the outgoing arguments.
  28: //
  29: // This expands to an LA or LAY instruction.  We restrict the offset
  30: // to the range of LA and keep the LAY range in reserve for when
  31: // the size of the outgoing arguments is added.
  32: def ADJDYNALLOC : Pseudo<(outs GR64:$dst), (ins dynalloc12only:$src),
  33:                          [(set GR64:$dst, dynalloc12only:$src)]>;
  34: 
  35: let Defs = [R15D, CC], Uses = [R15D], hasNoSchedulingInfo = 1,
  36:     usesCustomInserter = 1 in
  37:   def PROBED_ALLOCA : Pseudo<(outs GR64:$dst),
  38:                              (ins GR64:$oldSP, GR64:$space),
  39:            [(set GR64:$dst, (z_probed_alloca GR64:$oldSP, GR64:$space))]>;
  40: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `IsTargetXPLINK64`, `IsTargetELF`, `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP`, `ADJDYNALLOC`, `PROBED_ALLOCA`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `IsTargetXPLINK64`, `IsTargetELF`, `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP`, `ADJDYNALLOC`, `PROBED_ALLOCA` 等 TableGen 记录。

### Lines 41-80 / 第 41-80 行
```tablegen
  41: let Defs = [R1D, R15D, CC], Uses = [R15D], hasNoSchedulingInfo = 1,
  42:     hasSideEffects = 1 in
  43:   def PROBED_STACKALLOC : Pseudo<(outs), (ins i64imm:$stacksize), []>;
  44: 
  45: let Defs = [R3D, CC], Uses = [R3D, R4D], hasNoSchedulingInfo = 1,
  46:     hasSideEffects = 1 in
  47:   def XPLINK_STACKALLOC : Pseudo<(outs), (ins), []>;
  48: 
  49: //===----------------------------------------------------------------------===//
  50: // Branch instructions
  51: //===----------------------------------------------------------------------===//
  52: 
  53: // Conditional branches.
  54: let isBranch = 1, isTerminator = 1, Uses = [CC] in {
  55:   // It's easier for LLVM to handle these branches in their raw BRC/BRCL form
  56:   // with the condition-code mask being the first operand.  It seems friendlier
  57:   // to use mnemonic forms like JE and JLH when writing out the assembly though.
  58:   let isCodeGenOnly = 1 in {
  59:     // An assembler extended mnemonic for BRC.
  60:     def BRC  : CondBranchRI <"j#",  0xA74, z_br_ccmask>;
  61:     // An assembler extended mnemonic for BRCL.  (The extension is "G"
  62:     // rather than "L" because "JL" is "Jump if Less".)
  63:     def BRCL : CondBranchRIL<"jg#", 0xC04>;
  64:     let isIndirectBranch = 1 in {
  65:       def BC  : CondBranchRX<"b#",  0x47>;
  66:       def BCR : CondBranchRR<"b#r", 0x07>;
  67:       def BIC : CondBranchRXY<"bi#", 0xe347>,
  68:                 Requires<[FeatureMiscellaneousExtensions2]>;
  69:     }
  70:   }
  71: 
  72:   // Allow using the raw forms directly from the assembler (and occasional
  73:   // special code generation needs) as well.
  74:   def BRCAsm  : AsmCondBranchRI <"brc",  0xA74>;
  75:   def BRCLAsm : AsmCondBranchRIL<"brcl", 0xC04>;
  76:   let isIndirectBranch = 1 in {
  77:     def BCAsm  : AsmCondBranchRX<"bc",  0x47>;
  78:     def BCRAsm : AsmCondBranchRR<"bcr", 0x07>;
  79:     def BICAsm : AsmCondBranchRXY<"bic", 0xe347>,
  80:                  Requires<[FeatureMiscellaneousExtensions2]>;
```
- **EN**: This block declares or refines TableGen records such as `PROBED_STACKALLOC`, `XPLINK_STACKALLOC`, `BRC`, `BRCL`, `BC`, `BCR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `PROBED_STACKALLOC`, `XPLINK_STACKALLOC`, `BRC`, `BRCL`, `BC`, `BCR` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 81-120 / 第 81-120 行
```tablegen
  81:   }
  82: 
  83:   // Define AsmParser extended mnemonics for each general condition-code mask
  84:   // (integer or floating-point)
  85:   foreach V = [ "E", "NE", "H", "NH", "L", "NL", "HE", "NHE", "LE", "NLE",
  86:                 "Z", "NZ", "P", "NP", "M", "NM", "LH", "NLH", "O", "NO" ] in {
  87:     def JAsm#V  : FixedCondBranchRI <CV<V>, "j#",  0xA74>;
  88:     def JGAsm#V : FixedCondBranchRIL<CV<V>, "j{g|l}#", 0xC04>;
  89:     let isIndirectBranch = 1 in {
  90:       def BAsm#V  : FixedCondBranchRX <CV<V>, "b#",  0x47>;
  91:       def BRAsm#V : FixedCondBranchRR <CV<V>, "b#r", 0x07>;
  92:       def BIAsm#V : FixedCondBranchRXY<CV<V>, "bi#", 0xe347>,
  93:                     Requires<[FeatureMiscellaneousExtensions2]>;
  94:     }
  95:   }
  96: }
  97: 
  98: // Unconditional branches.  These are in fact simply variants of the
  99: // conditional branches with the condition mask set to "always".
 100: let isBranch = 1, isTerminator = 1, isBarrier = 1 in {
 101:   def J  : FixedCondBranchRI <CondAlways, "j",  0xA74, br>;
 102:   def JG : FixedCondBranchRIL<CondAlways, "j{g|lu}", 0xC04>;
 103:   let isIndirectBranch = 1 in {
 104:     def B  : FixedCondBranchRX<CondAlways, "b",  0x47>;
 105:     def BR : FixedCondBranchRR<CondAlways, "br", 0x07, brind>;
 106:     def BI : FixedCondBranchRXY<CondAlways, "bi", 0xe347, brind>,
 107:              Requires<[FeatureMiscellaneousExtensions2]>;
 108:   }
 109: }
 110: 
 111: // NOPs.  These are again variants of the conditional branches, with the
 112: // condition mask set to "never".
 113: defm NOP  : NeverCondBranchRX<"nop", 0x47>;
 114: defm NOPR : NeverCondBranchRR<"nopr", 0x07>;
 115: def JNOP  : NeverCondBranchRI<"jnop", 0xA74>;
 116: def JGNOP : NeverCondBranchRIL<"j{g|l}nop", 0xC04>;
 117: 
 118: // Fused compare-and-branch instructions.
 119: //
 120: // These instructions do not use or clobber the condition codes.
```
- **EN**: This block declares or refines TableGen records such as `JAsm`, `JGAsm`, `BAsm`, `BRAsm`, `BIAsm`, `J`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `JAsm`, `JGAsm`, `BAsm`, `BRAsm`, `BIAsm`, `J` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 121-160 / 第 121-160 行
```tablegen
 121: // We nevertheless pretend that the relative compare-and-branch
 122: // instructions clobber CC, so that we can lower them to separate
 123: // comparisons and BRCLs if the branch ends up being out of range.
 124: let isBranch = 1, isTerminator = 1 in {
 125:   // As for normal branches, we handle these instructions internally in
 126:   // their raw CRJ-like form, but use assembly macros like CRJE when writing
 127:   // them out.  Using the *Pair multiclasses, we also create the raw forms.
 128:   let Defs = [CC] in {
 129:     defm CRJ   : CmpBranchRIEbPair<"crj",   0xEC76, GR32>;
 130:     defm CGRJ  : CmpBranchRIEbPair<"cgrj",  0xEC64, GR64>;
 131:     defm CIJ   : CmpBranchRIEcPair<"cij",   0xEC7E, GR32, imm32sx8>;
 132:     defm CGIJ  : CmpBranchRIEcPair<"cgij",  0xEC7C, GR64, imm64sx8>;
 133:     defm CLRJ  : CmpBranchRIEbPair<"clrj",  0xEC77, GR32>;
 134:     defm CLGRJ : CmpBranchRIEbPair<"clgrj", 0xEC65, GR64>;
 135:     defm CLIJ  : CmpBranchRIEcPair<"clij",  0xEC7F, GR32, imm32zx8>;
 136:     defm CLGIJ : CmpBranchRIEcPair<"clgij", 0xEC7D, GR64, imm64zx8>;
 137:   }
 138:   let isIndirectBranch = 1 in {
 139:     defm CRB   : CmpBranchRRSPair<"crb",   0xECF6, GR32>;
 140:     defm CGRB  : CmpBranchRRSPair<"cgrb",  0xECE4, GR64>;
 141:     defm CIB   : CmpBranchRISPair<"cib",   0xECFE, GR32, imm32sx8>;
 142:     defm CGIB  : CmpBranchRISPair<"cgib",  0xECFC, GR64, imm64sx8>;
 143:     defm CLRB  : CmpBranchRRSPair<"clrb",  0xECF7, GR32>;
 144:     defm CLGRB : CmpBranchRRSPair<"clgrb", 0xECE5, GR64>;
 145:     defm CLIB  : CmpBranchRISPair<"clib",  0xECFF, GR32, imm32zx8>;
 146:     defm CLGIB : CmpBranchRISPair<"clgib", 0xECFD, GR64, imm64zx8>;
 147:   }
 148: 
 149:   // Define AsmParser mnemonics for each integer condition-code mask.
 150:   foreach V = [ "E", "H", "L", "HE", "LE", "LH",
 151:                 "NE", "NH", "NL", "NHE", "NLE", "NLH" ] in {
 152:     let Defs = [CC] in {
 153:       def CRJAsm#V   : FixedCmpBranchRIEb<ICV<V>, "crj",   0xEC76, GR32>;
 154:       def CGRJAsm#V  : FixedCmpBranchRIEb<ICV<V>, "cgrj",  0xEC64, GR64>;
 155:       def CIJAsm#V   : FixedCmpBranchRIEc<ICV<V>, "cij",   0xEC7E, GR32,
 156:                                           imm32sx8>;
 157:       def CGIJAsm#V  : FixedCmpBranchRIEc<ICV<V>, "cgij",  0xEC7C, GR64,
 158:                                           imm64sx8>;
 159:       def CLRJAsm#V  : FixedCmpBranchRIEb<ICV<V>, "clrj",  0xEC77, GR32>;
 160:       def CLGRJAsm#V : FixedCmpBranchRIEb<ICV<V>, "clgrj", 0xEC65, GR64>;
```
- **EN**: This block declares or refines TableGen records such as `CRJ`, `CGRJ`, `CIJ`, `CGIJ`, `CLRJ`, `CLGRJ`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `CRJ`, `CGRJ`, `CIJ`, `CGIJ`, `CLRJ`, `CLGRJ` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 161-200 / 第 161-200 行
```tablegen
 161:       def CLIJAsm#V  : FixedCmpBranchRIEc<ICV<V>, "clij",  0xEC7F, GR32,
 162:                                           imm32zx8>;
 163:       def CLGIJAsm#V : FixedCmpBranchRIEc<ICV<V>, "clgij", 0xEC7D, GR64,
 164:                                           imm64zx8>;
 165:     }
 166:     let isIndirectBranch = 1 in {
 167:       def CRBAsm#V   : FixedCmpBranchRRS<ICV<V>, "crb",   0xECF6, GR32>;
 168:       def CGRBAsm#V  : FixedCmpBranchRRS<ICV<V>, "cgrb",  0xECE4, GR64>;
 169:       def CIBAsm#V   : FixedCmpBranchRIS<ICV<V>, "cib",   0xECFE, GR32,
 170:                                          imm32sx8>;
 171:       def CGIBAsm#V  : FixedCmpBranchRIS<ICV<V>, "cgib",  0xECFC, GR64,
 172:                                          imm64sx8>;
 173:       def CLRBAsm#V  : FixedCmpBranchRRS<ICV<V>, "clrb",  0xECF7, GR32>;
 174:       def CLGRBAsm#V : FixedCmpBranchRRS<ICV<V>, "clgrb", 0xECE5, GR64>;
 175:       def CLIBAsm#V  : FixedCmpBranchRIS<ICV<V>, "clib",  0xECFF, GR32,
 176:                                          imm32zx8>;
 177:       def CLGIBAsm#V : FixedCmpBranchRIS<ICV<V>, "clgib", 0xECFD, GR64,
 178:                                          imm64zx8>;
 179:     }
 180:   }
 181: }
 182: 
 183: // Decrement a register and branch if it is nonzero.  These don't clobber CC,
 184: // but we might need to split long relative branches into sequences that do.
 185: let isBranch = 1, isTerminator = 1 in {
 186:   let Defs = [CC] in {
 187:     def BRCT  : BranchUnaryRI<"brct",  0xA76, GR32>;
 188:     def BRCTG : BranchUnaryRI<"brctg", 0xA77, GR64>;
 189:   }
 190:   // This doesn't need to clobber CC since we never need to split it.
 191:   def BRCTH : BranchUnaryRIL<"brcth", 0xCC6, GRH32>,
 192:               Requires<[FeatureHighWord]>;
 193: 
 194:   def BCT   : BranchUnaryRX<"bct",  0x46,GR32>;
 195:   def BCTR  : BranchUnaryRR<"bctr", 0x06, GR32>;
 196:   def BCTG  : BranchUnaryRXY<"bctg",  0xE346, GR64>;
 197:   def BCTGR : BranchUnaryRRE<"bctgr", 0xB946, GR64>;
 198: }
 199: 
 200: let isBranch = 1, isTerminator = 1 in {
```
- **EN**: This block declares or refines TableGen records such as `CLIJAsm`, `CLGIJAsm`, `CRBAsm`, `CGRBAsm`, `CIBAsm`, `CGIBAsm`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `CLIJAsm`, `CLGIJAsm`, `CRBAsm`, `CGRBAsm`, `CIBAsm`, `CGIBAsm` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 201-240 / 第 201-240 行
```tablegen
 201:   let Defs = [CC] in {
 202:     def BRXH  : BranchBinaryRSI<"brxh",  0x84, GR32>;
 203:     def BRXLE : BranchBinaryRSI<"brxle", 0x85, GR32>;
 204:     def BRXHG : BranchBinaryRIEe<"brxhg", 0xEC44, GR64>;
 205:     def BRXLG : BranchBinaryRIEe<"brxlg", 0xEC45, GR64>;
 206:   }
 207:   def BXH   : BranchBinaryRS<"bxh",  0x86, GR32>;
 208:   def BXLE  : BranchBinaryRS<"bxle", 0x87, GR32>;
 209:   def BXHG  : BranchBinaryRSY<"bxhg",  0xEB44, GR64>;
 210:   def BXLEG : BranchBinaryRSY<"bxleg", 0xEB45, GR64>;
 211: }
 212: 
 213: //===----------------------------------------------------------------------===//
 214: // Trap instructions
 215: //===----------------------------------------------------------------------===//
 216: 
 217: // Unconditional trap.
 218: let hasCtrlDep = 1, hasSideEffects = 1 in
 219:   def Trap : Alias<4, (outs), (ins), [(trap)]>;
 220: 
 221: // Conditional trap.
 222: let hasCtrlDep = 1, Uses = [CC], hasSideEffects = 1 in
 223:   def CondTrap : Alias<4, (outs), (ins cond4:$valid, cond4:$R1), []>;
 224: 
 225: // Fused compare-and-trap instructions.
 226: let hasCtrlDep = 1, hasSideEffects = 1 in {
 227:   // These patterns work the same way as for compare-and-branch.
 228:   defm CRT   : CmpBranchRRFcPair<"crt",   0xB972, GR32>;
 229:   defm CGRT  : CmpBranchRRFcPair<"cgrt",  0xB960, GR64>;
 230:   defm CLRT  : CmpBranchRRFcPair<"clrt",  0xB973, GR32>;
 231:   defm CLGRT : CmpBranchRRFcPair<"clgrt", 0xB961, GR64>;
 232:   defm CIT   : CmpBranchRIEaPair<"cit",   0xEC72, GR32, imm32sx16>;
 233:   defm CGIT  : CmpBranchRIEaPair<"cgit",  0xEC70, GR64, imm64sx16>;
 234:   defm CLFIT : CmpBranchRIEaPair<"clfit", 0xEC73, GR32, imm32zx16>;
 235:   defm CLGIT : CmpBranchRIEaPair<"clgit", 0xEC71, GR64, imm64zx16>;
 236:   let Predicates = [FeatureMiscellaneousExtensions] in {
 237:     defm CLT  : CmpBranchRSYbPair<"clt",  0xEB23, GR32>;
 238:     defm CLGT : CmpBranchRSYbPair<"clgt", 0xEB2B, GR64>;
 239:   }
 240: 
```
- **EN**: This block declares or refines TableGen records such as `BRXH`, `BRXLE`, `BRXHG`, `BRXLG`, `BXH`, `BXLE`.
- **CN**: 该代码块声明或细化了 `BRXH`, `BRXLE`, `BRXHG`, `BRXLG`, `BXH`, `BXLE` 等 TableGen 记录。

### Lines 241-280 / 第 241-280 行
```tablegen
 241:   foreach V = [ "E", "H", "L", "HE", "LE", "LH",
 242:                 "NE", "NH", "NL", "NHE", "NLE", "NLH" ] in {
 243:     def CRTAsm#V   : FixedCmpBranchRRFc<ICV<V>, "crt",   0xB972, GR32>;
 244:     def CGRTAsm#V  : FixedCmpBranchRRFc<ICV<V>, "cgrt",  0xB960, GR64>;
 245:     def CLRTAsm#V  : FixedCmpBranchRRFc<ICV<V>, "clrt",  0xB973, GR32>;
 246:     def CLGRTAsm#V : FixedCmpBranchRRFc<ICV<V>, "clgrt", 0xB961, GR64>;
 247:     def CITAsm#V   : FixedCmpBranchRIEa<ICV<V>, "cit",   0xEC72, GR32,
 248:                                          imm32sx16>;
 249:     def CGITAsm#V  : FixedCmpBranchRIEa<ICV<V>, "cgit",  0xEC70, GR64,
 250:                                          imm64sx16>;
 251:     def CLFITAsm#V : FixedCmpBranchRIEa<ICV<V>, "clfit", 0xEC73, GR32,
 252:                                          imm32zx16>;
 253:     def CLGITAsm#V : FixedCmpBranchRIEa<ICV<V>, "clgit", 0xEC71, GR64,
 254:                                          imm64zx16>;
 255:     let Predicates = [FeatureMiscellaneousExtensions] in {
 256:       def CLTAsm#V  : FixedCmpBranchRSYb<ICV<V>, "clt",  0xEB23, GR32>;
 257:       def CLGTAsm#V : FixedCmpBranchRSYb<ICV<V>, "clgt", 0xEB2B, GR64>;
 258:     }
 259:   }
 260: }
 261: 
 262: //===----------------------------------------------------------------------===//
 263: // Call and return instructions
 264: //===----------------------------------------------------------------------===//
 265: 
 266: // Define the general form of the call instructions for the asm parser.
 267: // These instructions don't hard-code %r14 as the return address register.
 268: let isCall = 1, Defs = [CC] in {
 269:   def BRAS  : CallRI <"bras", 0xA75>;
 270:   def BRASL : CallRIL<"brasl", 0xC05>;
 271:   def BAS   : CallRX <"bas", 0x4D>;
 272:   def BASR  : CallRR <"basr", 0x0D>;
 273: }
 274: 
 275: // A symbol in the ADA (z/OS only).
 276: def adasym : Operand<i64>;
 277: 
 278: // z/OS XPLINK
 279: let Predicates = [IsTargetXPLINK64] in {
 280:   let isCall = 1, Defs = [R7D, CC], Uses = [FPC] in {
```
- **EN**: This block declares or refines TableGen records such as `CRTAsm`, `CGRTAsm`, `CLRTAsm`, `CLGRTAsm`, `CITAsm`, `CGITAsm`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `CRTAsm`, `CGRTAsm`, `CLRTAsm`, `CLGRTAsm`, `CITAsm`, `CGITAsm` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 281-320 / 第 281-320 行
```tablegen
 281:     def CallBRASL_XPLINK64 : Alias<8, (outs), (ins pcrel32:$I2, variable_ops),
 282:                           [(z_call pcrel32:$I2)]>;
 283:     def CallBASR_XPLINK64  : Alias<4, (outs), (ins ADDR64:$R2, variable_ops),
 284:                           [(z_call ADDR64:$R2)]>;
 285:   }
 286: 
 287:   let isCall = 1, Defs = [R3D, CC], Uses = [FPC] in {
 288:     def CallBASR_STACKEXT  : Alias<4, (outs), (ins ADDR64:$R2), []>;
 289:   }
 290: 
 291:   let hasNoSchedulingInfo = 1, Defs = [CC] in {
 292:     def ADA_ENTRY : Alias<12, (outs ADDR64:$Reg), (ins adasym:$addr,
 293:                               ADDR64:$ADA, imm64:$Offset),
 294:                             [(set i64:$Reg, (z_ada_entry i64:$addr,
 295:                               i64:$ADA, i64:$Offset))]>;
 296:   }
 297:   let mayLoad = 1, AddedComplexity = 20, hasNoSchedulingInfo = 1, Defs = [CC] in {
 298:     def ADA_ENTRY_VALUE : Alias<12, (outs ADDR64:$Reg), (ins adasym:$addr,
 299:                               ADDR64:$ADA, imm64:$Offset),
 300:                             [(set i64:$Reg, (z_load (z_ada_entry
 301:                               iPTR:$addr, iPTR:$ADA, i64:$Offset)))]>;
 302:  }
 303: }
 304: 
 305: // Regular calls.
 306: // z/Linux ELF
 307: let Predicates = [IsTargetELF] in {
 308:   let isCall = 1, Defs = [R14D, CC], Uses = [FPC] in {
 309:     def CallBRASL : Alias<6, (outs), (ins pcrel32:$I2, variable_ops),
 310:                           [(z_call pcrel32:$I2)]>;
 311:     def CallBASR  : Alias<2, (outs), (ins ADDR64:$R2, variable_ops),
 312:                           [(z_call ADDR64:$R2)]>;
 313:   }
 314: 
 315:   // TLS calls.  These will be lowered into a call to __tls_get_offset,
 316:   // with an extra relocation specifying the TLS symbol.
 317:   let isCall = 1, Defs = [R14D, CC] in {
 318:     def TLS_GDCALL : Alias<6, (outs), (ins tlssym:$I2, variable_ops),
 319:                            [(z_tls_gdcall tglobaltlsaddr:$I2)]>;
 320:     def TLS_LDCALL : Alias<6, (outs), (ins tlssym:$I2, variable_ops),
```
- **EN**: This block declares or refines TableGen records such as `CallBRASL_XPLINK64`, `CallBASR_XPLINK64`, `CallBASR_STACKEXT`, `ADA_ENTRY`, `ADA_ENTRY_VALUE`, `CallBRASL`.
- **CN**: 该代码块声明或细化了 `CallBRASL_XPLINK64`, `CallBASR_XPLINK64`, `CallBASR_STACKEXT`, `ADA_ENTRY`, `ADA_ENTRY_VALUE`, `CallBRASL` 等 TableGen 记录。

### Lines 321-360 / 第 321-360 行
```tablegen
 321:                            [(z_tls_ldcall tglobaltlsaddr:$I2)]>;
 322:   }
 323: }
 324: 
 325: // Sibling calls. Indirect sibling calls must be via R6 for XPLink,
 326: // R1 used for ELF
 327: let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1 in {
 328:   def CallJG : Alias<6, (outs), (ins pcrel32:$I2),
 329:                      [(z_sibcall pcrel32:$I2)]>;
 330:   def CallBR : Alias<2, (outs), (ins ADDR64:$R2),
 331:                      [(z_sibcall ADDR64:$R2)]>;
 332: }
 333: 
 334: // Conditional sibling calls.
 335: let CCMaskFirst = 1, isCall = 1, isTerminator = 1, isReturn = 1 in {
 336:   def CallBRCL : Alias<6, (outs), (ins cond4:$valid, cond4:$R1,
 337:                                    pcrel32:$I2), []>;
 338:   def CallBCR : Alias<2, (outs), (ins cond4:$valid, cond4:$R1,
 339:                                   ADDR64:$R2), []>;
 340: }
 341: 
 342: // Fused compare and conditional sibling calls.
 343: let isCall = 1, isTerminator = 1, isReturn = 1 in {
 344:   def CRBCall : Alias<6, (outs), (ins GR32:$R1, GR32:$R2, cond4:$M3, ADDR64:$R4), []>;
 345:   def CGRBCall : Alias<6, (outs), (ins GR64:$R1, GR64:$R2, cond4:$M3, ADDR64:$R4), []>;
 346:   def CIBCall : Alias<6, (outs), (ins GR32:$R1, imm32sx8:$I2, cond4:$M3, ADDR64:$R4), []>;
 347:   def CGIBCall : Alias<6, (outs), (ins GR64:$R1, imm64sx8:$I2, cond4:$M3, ADDR64:$R4), []>;
 348:   def CLRBCall : Alias<6, (outs), (ins GR32:$R1, GR32:$R2, cond4:$M3, ADDR64:$R4), []>;
 349:   def CLGRBCall : Alias<6, (outs), (ins GR64:$R1, GR64:$R2, cond4:$M3, ADDR64:$R4), []>;
 350:   def CLIBCall : Alias<6, (outs), (ins GR32:$R1, imm32zx8:$I2, cond4:$M3, ADDR64:$R4), []>;
 351:   def CLGIBCall : Alias<6, (outs), (ins GR64:$R1, imm64zx8:$I2, cond4:$M3, ADDR64:$R4), []>;
 352: }
 353: 
 354: let Predicates = [IsTargetXPLINK64] in {
 355:   // A return instruction (b 2(%r7)).
 356:   let isReturn = 1, isTerminator = 1, isBarrier = 1, hasCtrlDep = 1 in
 357:     def Return_XPLINK : Alias<4, (outs), (ins), [(z_retglue)]>;
 358: 
 359:   // A conditional return instruction (bc <cond>, 2(%r7)).
 360:   let isReturn = 1, isTerminator = 1, hasCtrlDep = 1, CCMaskFirst = 1, Uses = [CC] in
```
- **EN**: This block declares or refines TableGen records such as `CallJG`, `CallBR`, `CallBRCL`, `CallBCR`, `CRBCall`, `CGRBCall`.
- **CN**: 该代码块声明或细化了 `CallJG`, `CallBR`, `CallBRCL`, `CallBCR`, `CRBCall`, `CGRBCall` 等 TableGen 记录。

### Lines 361-400 / 第 361-400 行
```tablegen
 361:     def CondReturn_XPLINK : Alias<4, (outs), (ins cond4:$valid, cond4:$R1), []>;
 362: }
 363: 
 364: let Predicates = [IsTargetELF] in {
 365:   // A return instruction (br %r14).
 366:   let isReturn = 1, isTerminator = 1, isBarrier = 1, hasCtrlDep = 1 in
 367:     def Return : Alias<2, (outs), (ins), [(z_retglue)]>;
 368: 
 369:   // A conditional return instruction (bcr <cond>, %r14).
 370:   let isReturn = 1, isTerminator = 1, hasCtrlDep = 1, CCMaskFirst = 1, Uses = [CC] in
 371:     def CondReturn : Alias<2, (outs), (ins cond4:$valid, cond4:$R1), []>;
 372: }
 373: 
 374: // Fused compare and conditional returns.
 375: let isReturn = 1, isTerminator = 1, hasCtrlDep = 1 in {
 376:   def CRBReturn : Alias<6, (outs), (ins GR32:$R1, GR32:$R2, cond4:$M3), []>;
 377:   def CGRBReturn : Alias<6, (outs), (ins GR64:$R1, GR64:$R2, cond4:$M3), []>;
 378:   def CIBReturn : Alias<6, (outs), (ins GR32:$R1, imm32sx8:$I2, cond4:$M3), []>;
 379:   def CGIBReturn : Alias<6, (outs), (ins GR64:$R1, imm64sx8:$I2, cond4:$M3), []>;
 380:   def CLRBReturn : Alias<6, (outs), (ins GR32:$R1, GR32:$R2, cond4:$M3), []>;
 381:   def CLGRBReturn : Alias<6, (outs), (ins GR64:$R1, GR64:$R2, cond4:$M3), []>;
 382:   def CLIBReturn : Alias<6, (outs), (ins GR32:$R1, imm32zx8:$I2, cond4:$M3), []>;
 383:   def CLGIBReturn : Alias<6, (outs), (ins GR64:$R1, imm64zx8:$I2, cond4:$M3), []>;
 384: }
 385: 
 386: //===----------------------------------------------------------------------===//
 387: // Select instructions
 388: //===----------------------------------------------------------------------===//
 389: 
 390: def Select32    : SelectWrapper<i32, GR32>,
 391:                   Requires<[FeatureNoLoadStoreOnCond]>;
 392: def Select64    : SelectWrapper<i64, GR64>,
 393:                   Requires<[FeatureNoLoadStoreOnCond]>;
 394: 
 395: // We don't define 32-bit Mux stores if we don't have STOCFH, because the
 396: // low-only STOC should then always be used if possible.
 397: defm CondStore8Mux  : CondStores<GRX32, nonvolatile_truncstorei8,
 398:                                  nonvolatile_anyextloadi8, bdxaddr20only>,
 399:                       Requires<[FeatureHighWord]>;
 400: defm CondStore16Mux : CondStores<GRX32, nonvolatile_truncstorei16,
```
- **EN**: This block declares or refines TableGen records such as `CondReturn_XPLINK`, `Return`, `CondReturn`, `CRBReturn`, `CGRBReturn`, `CIBReturn`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `CondReturn_XPLINK`, `Return`, `CondReturn`, `CRBReturn`, `CGRBReturn`, `CIBReturn` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 401-440 / 第 401-440 行
```tablegen
 401:                                  nonvolatile_anyextloadi16, bdxaddr20only>,
 402:                       Requires<[FeatureHighWord]>;
 403: defm CondStore32Mux : CondStores<GRX32, simple_store,
 404:                                  simple_load, bdxaddr20only>,
 405:                       Requires<[FeatureLoadStoreOnCond2]>;
 406: defm CondStore8     : CondStores<GR32, nonvolatile_truncstorei8,
 407:                                  nonvolatile_anyextloadi8, bdxaddr20only>;
 408: defm CondStore16    : CondStores<GR32, nonvolatile_truncstorei16,
 409:                                  nonvolatile_anyextloadi16, bdxaddr20only>;
 410: defm CondStore32    : CondStores<GR32, simple_store,
 411:                                  simple_load, bdxaddr20only>;
 412: 
 413: defm : CondStores64<CondStore8, CondStore8Inv, nonvolatile_truncstorei8,
 414:                     nonvolatile_anyextloadi8, bdxaddr20only>;
 415: defm : CondStores64<CondStore16, CondStore16Inv, nonvolatile_truncstorei16,
 416:                     nonvolatile_anyextloadi16, bdxaddr20only>;
 417: defm : CondStores64<CondStore32, CondStore32Inv, nonvolatile_truncstorei32,
 418:                     nonvolatile_anyextloadi32, bdxaddr20only>;
 419: defm CondStore64 : CondStores<GR64, simple_store,
 420:                               simple_load, bdxaddr20only>;
 421: 
 422: //===----------------------------------------------------------------------===//
 423: // Move instructions
 424: //===----------------------------------------------------------------------===//
 425: 
 426: // Register moves.
 427: let isMoveReg = 1 in {
 428:   def LR  : UnaryRR <"lr",  0x18,   null_frag, GR32, GR32>;
 429:   def LGR : UnaryRRE<"lgr", 0xB904, null_frag, GR64, GR64>;
 430: }
 431: 
 432: let Defs = [CC], CCValues = 0xE, CompareZeroCCMask = 0xE in {
 433:   def LTR  : UnaryRR <"ltr",  0x12,   null_frag, GR32, GR32>;
 434:   def LTGR : UnaryRRE<"ltgr", 0xB902, null_frag, GR64, GR64>;
 435: }
 436: 
 437: let usesCustomInserter = 1, hasNoSchedulingInfo = 1 in
 438:   def PAIR128 : Pseudo<(outs GR128:$dst), (ins GR64:$hi, GR64:$lo), []>;
 439: 
 440: // Immediate moves.
```
- **EN**: This block declares or refines TableGen records such as `CondStore32Mux`, `CondStore8`, `CondStore16`, `CondStore32`, `CondStore64`, `LR`.
- **CN**: 该代码块声明或细化了 `CondStore32Mux`, `CondStore8`, `CondStore16`, `CondStore32`, `CondStore64`, `LR` 等 TableGen 记录。

### Lines 441-480 / 第 441-480 行
```tablegen
 441: let isAsCheapAsAMove = 1, isMoveImm = 1, isReMaterializable = 1 in {
 442:   // 16-bit sign-extended immediates.  LHIMux expands to LHI or IIHF,
 443:   // deopending on the choice of register.
 444:   def LHIMux : UnaryRIPseudo<bitconvert, GRX32, imm32sx16>,
 445:                Requires<[FeatureHighWord]>;
 446:   def LHI  : UnaryRI<"lhi",  0xA78, bitconvert, GR32, imm32sx16>;
 447:   def LGHI : UnaryRI<"lghi", 0xA79, bitconvert, GR64, imm64sx16>;
 448: 
 449:   // Other 16-bit immediates.
 450:   def LLILL : UnaryRI<"llill", 0xA5F, bitconvert, GR64, imm64ll16>;
 451:   def LLILH : UnaryRI<"llilh", 0xA5E, bitconvert, GR64, imm64lh16>;
 452:   def LLIHL : UnaryRI<"llihl", 0xA5D, bitconvert, GR64, imm64hl16>;
 453:   def LLIHH : UnaryRI<"llihh", 0xA5C, bitconvert, GR64, imm64hh16>;
 454: 
 455:   // 32-bit immediates.
 456:   def LGFI  : UnaryRIL<"lgfi",  0xC01, bitconvert, GR64, imm64sx32>;
 457:   def LLILF : UnaryRIL<"llilf", 0xC0F, bitconvert, GR64, imm64lf32>;
 458:   def LLIHF : UnaryRIL<"llihf", 0xC0E, bitconvert, GR64, imm64hf32>;
 459: }
 460: def LLGFI : InstAlias<"llgfi\t$R1, $RI1", (LLILF GR64:$R1, imm64lf32:$RI1)>;
 461: def LLGHI : InstAlias<"llghi\t$R1, $RI1", (LLILL GR64:$R1, imm64ll16:$RI1)>;
 462: 
 463: // Register loads.
 464: let canFoldAsLoad = 1, SimpleBDXLoad = 1, mayLoad = 1,
 465:     isReMaterializable = 1 in {
 466:   // Expands to L, LY or LFH, depending on the choice of register.
 467:   def LMux : UnaryRXYPseudo<"l", z_load, GRX32, 4>,
 468:              Requires<[FeatureHighWord]>;
 469:   defm L : UnaryRXPair<"l", 0x58, 0xE358, z_load, GR32, 4>;
 470:   def LFH : UnaryRXY<"lfh", 0xE3CA, z_load, GRH32, 4>,
 471:             Requires<[FeatureHighWord]>;
 472:   def LG : UnaryRXY<"lg", 0xE304, z_load, GR64, 8>;
 473: 
 474:   // These instructions are split after register allocation, so we don't
 475:   // want a custom inserter.
 476:   let Has20BitOffset = 1, HasIndex = 1, Is128Bit = 1 in {
 477:     def L128 : Pseudo<(outs GR128:$dst), (ins bdxaddr20only128:$src),
 478:                       [(set GR128:$dst, (load bdxaddr20only128:$src))]>;
 479:   }
 480: }
```
- **EN**: This block declares or refines TableGen records such as `LHIMux`, `LHI`, `LGHI`, `LLILL`, `LLILH`, `LLIHL`.
- **CN**: 该代码块声明或细化了 `LHIMux`, `LHI`, `LGHI`, `LLILL`, `LLILH`, `LLIHL` 等 TableGen 记录。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: let Defs = [CC], CCValues = 0xE, CompareZeroCCMask = 0xE in {
 482:   def LT  : UnaryRXY<"lt",  0xE312, z_load, GR32, 4>;
 483:   def LTG : UnaryRXY<"ltg", 0xE302, z_load, GR64, 8>;
 484: }
 485: 
 486: let canFoldAsLoad = 1, isReMaterializable =1 in {
 487:   def LRL  : UnaryRILPC<"lrl",  0xC4D, aligned_z_load, GR32>;
 488:   def LGRL : UnaryRILPC<"lgrl", 0xC48, aligned_z_load, GR64>;
 489: }
 490: 
 491: // Load and zero rightmost byte.
 492: let Predicates = [FeatureLoadAndZeroRightmostByte] in {
 493:   def LZRF : UnaryRXY<"lzrf", 0xE33B, null_frag, GR32, 4>;
 494:   def LZRG : UnaryRXY<"lzrg", 0xE32A, null_frag, GR64, 8>;
 495:   def : Pat<(and (i32 (z_load bdxaddr20only:$src)), 0xffffff00),
 496:             (LZRF bdxaddr20only:$src)>;
 497:   def : Pat<(and (i64 (z_load bdxaddr20only:$src)), 0xffffffffffffff00),
 498:             (LZRG bdxaddr20only:$src)>;
 499: }
 500: 
 501: // Load and trap.
 502: let Predicates = [FeatureLoadAndTrap], hasSideEffects = 1 in {
 503:   def LAT   : UnaryRXY<"lat",   0xE39F, null_frag, GR32, 4>;
 504:   def LFHAT : UnaryRXY<"lfhat", 0xE3C8, null_frag, GRH32, 4>;
 505:   def LGAT  : UnaryRXY<"lgat",  0xE385, null_frag, GR64, 8>;
 506: }
 507: 
 508: // Register stores.
 509: let SimpleBDXStore = 1, mayStore = 1 in {
 510:   // Expands to ST, STY or STFH, depending on the choice of register.
 511:   def STMux : StoreRXYPseudo<store, GRX32, 4>,
 512:               Requires<[FeatureHighWord]>;
 513:   defm ST : StoreRXPair<"st", 0x50, 0xE350, store, GR32, 4>;
 514:   def STFH : StoreRXY<"stfh", 0xE3CB, store, GRH32, 4>,
 515:              Requires<[FeatureHighWord]>;
 516:   def STG : StoreRXY<"stg", 0xE324, store, GR64, 8>;
 517: 
 518:   // These instructions are split after register allocation, so we don't
 519:   // want a custom inserter.
 520:   let Has20BitOffset = 1, HasIndex = 1, Is128Bit = 1 in {
```
- **EN**: This block declares or refines TableGen records such as `LT`, `LTG`, `LRL`, `LGRL`, `LZRF`, `LZRG`.
- **CN**: 该代码块声明或细化了 `LT`, `LTG`, `LRL`, `LGRL`, `LZRF`, `LZRG` 等 TableGen 记录。

### Lines 521-560 / 第 521-560 行
```tablegen
 521:     def ST128 : Pseudo<(outs), (ins GR128:$src, bdxaddr20only128:$dst),
 522:                        [(store GR128:$src, bdxaddr20only128:$dst)]>;
 523:   }
 524: }
 525: def STRL  : StoreRILPC<"strl", 0xC4F, aligned_store, GR32>;
 526: def STGRL : StoreRILPC<"stgrl", 0xC4B, aligned_store, GR64>;
 527: 
 528: // 8-bit immediate stores to 8-bit fields.
 529: defm MVI : StoreSIPair<"mvi", 0x92, 0xEB52, truncstorei8, imm32zx8trunc>;
 530: 
 531: // 16-bit immediate stores to 16-, 32- or 64-bit fields.
 532: def MVHHI : StoreSIL<"mvhhi", 0xE544, truncstorei16, imm32sx16trunc>;
 533: def MVHI  : StoreSIL<"mvhi",  0xE54C, store,         imm32sx16>;
 534: def MVGHI : StoreSIL<"mvghi", 0xE548, store,         imm64sx16>;
 535: 
 536: // Memory-to-memory moves.
 537: let mayLoad = 1, mayStore = 1 in
 538:   defm MVC : MemorySS<"mvc", 0xD2, z_mvc>;
 539: let mayLoad = 1, mayStore = 1, Defs = [CC] in {
 540:   def MVCL  : SideEffectBinaryMemMemRR<"mvcl", 0x0E, GR128, GR128>;
 541:   def MVCLE : SideEffectTernaryMemMemRS<"mvcle", 0xA8, GR128, GR128>;
 542:   def MVCLU : SideEffectTernaryMemMemRSY<"mvclu", 0xEB8E, GR128, GR128>;
 543: }
 544: 
 545: // Memset[Length][Byte] pseudos.
 546: def MemsetImmImm : MemsetPseudo<imm64, imm32zx8trunc>;
 547: def MemsetImmReg : MemsetPseudo<imm64, GR32>;
 548: def MemsetRegImm : MemsetPseudo<ADDR64, imm32zx8trunc>;
 549: def MemsetRegReg : MemsetPseudo<ADDR64, GR32>;
 550: 
 551: // Move right.
 552: let Predicates = [FeatureMiscellaneousExtensions3],
 553:     mayLoad = 1, mayStore = 1, Uses = [R0L] in
 554:   def MVCRL : SideEffectBinarySSE<"mvcrl", 0xE50A>;
 555: 
 556: // String moves.
 557: let mayLoad = 1, mayStore = 1, Defs = [CC] in
 558:   defm MVST : StringRRE<"mvst", 0xB255, z_stpcpy>;
 559: 
 560: //===----------------------------------------------------------------------===//
```
- **EN**: This block declares or refines TableGen records such as `ST128`, `STRL`, `STGRL`, `MVI`, `MVHHI`, `MVHI`.
- **CN**: 该代码块声明或细化了 `ST128`, `STRL`, `STGRL`, `MVI`, `MVHHI`, `MVHI` 等 TableGen 记录。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: // Conditional move instructions
 562: //===----------------------------------------------------------------------===//
 563: 
 564: let Predicates = [FeatureMiscellaneousExtensions3], Uses = [CC] in {
 565:   // Select.
 566:   let isCommutable = 1 in {
 567:     // Expands to SELR or SELFHR or a branch-and-move sequence,
 568:     // depending on the choice of registers.
 569:     def  SELRMux : CondBinaryRRFaPseudo<"MUXselr", GRX32, GRX32, GRX32>;
 570:     defm SELFHR  : CondBinaryRRFaPair<"selfhr", 0xB9C0, GRH32, GRH32, GRH32>;
 571:     defm SELR    : CondBinaryRRFaPair<"selr",   0xB9F0, GR32, GR32, GR32>;
 572:     defm SELGR   : CondBinaryRRFaPair<"selgr",  0xB9E3, GR64, GR64, GR64>;
 573:   }
 574: 
 575:   // Define AsmParser extended mnemonics for each general condition-code mask.
 576:   foreach V = [ "E", "NE", "H", "NH", "L", "NL", "HE", "NHE", "LE", "NLE",
 577:                 "Z", "NZ", "P", "NP", "M", "NM", "LH", "NLH", "O", "NO" ] in {
 578:     def SELRAsm#V   : FixedCondBinaryRRFa<CV<V>, "selr",   0xB9F0,
 579:                                           GR32, GR32, GR32>;
 580:     def SELFHRAsm#V : FixedCondBinaryRRFa<CV<V>, "selfhr", 0xB9C0,
 581:                                           GRH32, GRH32, GRH32>;
 582:     def SELGRAsm#V  : FixedCondBinaryRRFa<CV<V>, "selgr",  0xB9E3,
 583:                                           GR64, GR64, GR64>;
 584:   }
 585: }
 586: 
 587: let Predicates = [FeatureLoadStoreOnCond2], Uses = [CC] in {
 588:   // Load immediate on condition.  Matched via DAG pattern and created
 589:   // by the PeepholeOptimizer via FoldImmediate.
 590: 
 591:   // Expands to LOCHI or LOCHHI, depending on the choice of register.
 592:   def LOCHIMux : CondBinaryRIEPseudo<GRX32, imm32sx16>;
 593:   defm LOCHHI  : CondBinaryRIEPair<"lochhi", 0xEC4E, GRH32, imm32sx16>;
 594:   defm LOCHI   : CondBinaryRIEPair<"lochi",  0xEC42, GR32, imm32sx16>;
 595:   defm LOCGHI  : CondBinaryRIEPair<"locghi", 0xEC46, GR64, imm64sx16>;
 596: 
 597:   // Move register on condition.  Matched via DAG pattern and
 598:   // created by early if-conversion.
 599:   let isCommutable = 1 in {
 600:     // Expands to LOCR or LOCFHR or a branch-and-move sequence,
```
- **EN**: This block declares or refines TableGen records such as `SELRMux`, `SELFHR`, `SELR`, `SELGR`, `SELRAsm`, `SELFHRAsm`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `SELRMux`, `SELFHR`, `SELR`, `SELGR`, `SELRAsm`, `SELFHRAsm` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 601-640 / 第 601-640 行
```tablegen
 601:     // depending on the choice of registers.
 602:     def LOCRMux : CondBinaryRRFPseudo<"MUXlocr", GRX32, GRX32>;
 603:     defm LOCFHR : CondBinaryRRFPair<"locfhr", 0xB9E0, GRH32, GRH32>;
 604:   }
 605: 
 606:   // Load on condition.  Matched via DAG pattern.
 607:   // Expands to LOC or LOCFH, depending on the choice of register.
 608:   defm LOCMux : CondUnaryRSYPseudoAndMemFold<"MUXloc", simple_load, GRX32, 4>;
 609:   defm LOCFH : CondUnaryRSYPair<"locfh", 0xEBE0, simple_load, GRH32, 4>;
 610: 
 611:   // Store on condition.  Expanded from CondStore* pseudos.
 612:   // Expands to STOC or STOCFH, depending on the choice of register.
 613:   def STOCMux : CondStoreRSYPseudo<GRX32, 4>;
 614:   defm STOCFH : CondStoreRSYPair<"stocfh", 0xEBE1, GRH32, 4>;
 615: 
 616:   // Define AsmParser extended mnemonics for each general condition-code mask.
 617:   foreach V = [ "E", "NE", "H", "NH", "L", "NL", "HE", "NHE", "LE", "NLE",
 618:                 "Z", "NZ", "P", "NP", "M", "NM", "LH", "NLH", "O", "NO" ] in {
 619:     def LOCHIAsm#V  : FixedCondBinaryRIE<CV<V>, "lochi",  0xEC42, GR32,
 620:                                          imm32sx16>;
 621:     def LOCGHIAsm#V : FixedCondBinaryRIE<CV<V>, "locghi", 0xEC46, GR64,
 622:                                          imm64sx16>;
 623:     def LOCHHIAsm#V : FixedCondBinaryRIE<CV<V>, "lochhi", 0xEC4E, GRH32,
 624:                                          imm32sx16>;
 625:     def LOCFHRAsm#V : FixedCondBinaryRRF<CV<V>, "locfhr", 0xB9E0, GRH32, GRH32>;
 626:     def LOCFHAsm#V  : FixedCondUnaryRSY<CV<V>, "locfh",  0xEBE0, GRH32, 4>;
 627:     def STOCFHAsm#V : FixedCondStoreRSY<CV<V>, "stocfh", 0xEBE1, GRH32, 4>;
 628:   }
 629: }
 630: 
 631: let Predicates = [FeatureLoadStoreOnCond], Uses = [CC] in {
 632:   // Move register on condition.  Matched via DAG pattern and
 633:   // created by early if-conversion.
 634:   let isCommutable = 1 in {
 635:     defm LOCR  : CondBinaryRRFPair<"locr",  0xB9F2, GR32, GR32>;
 636:     defm LOCGR : CondBinaryRRFPair<"locgr", 0xB9E2, GR64, GR64>;
 637:   }
 638: 
 639:   // Load on condition.  Matched via DAG pattern.
 640:   defm LOC  : CondUnaryRSYPair<"loc",  0xEBF2, simple_load, GR32, 4>;
```
- **EN**: This block declares or refines TableGen records such as `LOCRMux`, `LOCFHR`, `LOCMux`, `LOCFH`, `STOCMux`, `STOCFH`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `LOCRMux`, `LOCFHR`, `LOCMux`, `LOCFH`, `STOCMux`, `STOCFH` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 641-680 / 第 641-680 行
```tablegen
 641:   defm LOCG : CondUnaryRSYPairAndMemFold<"locg", 0xEBE2, simple_load, GR64, 8>;
 642: 
 643:   // Store on condition.  Expanded from CondStore* pseudos.
 644:   defm STOC  : CondStoreRSYPair<"stoc",  0xEBF3, GR32, 4>;
 645:   defm STOCG : CondStoreRSYPair<"stocg", 0xEBE3, GR64, 8>;
 646: 
 647:   // Define AsmParser extended mnemonics for each general condition-code mask.
 648:   foreach V = [ "E", "NE", "H", "NH", "L", "NL", "HE", "NHE", "LE", "NLE",
 649:                 "Z", "NZ", "P", "NP", "M", "NM", "LH", "NLH", "O", "NO" ] in {
 650:     def LOCRAsm#V   : FixedCondBinaryRRF<CV<V>, "locr",  0xB9F2, GR32, GR32>;
 651:     def LOCGRAsm#V  : FixedCondBinaryRRF<CV<V>, "locgr", 0xB9E2, GR64, GR64>;
 652:     def LOCAsm#V    : FixedCondUnaryRSY<CV<V>, "loc",   0xEBF2, GR32, 4>;
 653:     def LOCGAsm#V   : FixedCondUnaryRSY<CV<V>, "locg",  0xEBE2, GR64, 8>;
 654:     def STOCAsm#V   : FixedCondStoreRSY<CV<V>, "stoc",  0xEBF3, GR32, 4>;
 655:     def STOCGAsm#V  : FixedCondStoreRSY<CV<V>, "stocg", 0xEBE3, GR64, 8>;
 656:   }
 657: }
 658: //===----------------------------------------------------------------------===//
 659: // Sign extensions
 660: //===----------------------------------------------------------------------===//
 661: //
 662: // Note that putting these before zero extensions mean that we will prefer
 663: // them for anyextload*.  There's not really much to choose between the two
 664: // either way, but signed-extending loads have a short LH and a long LHY,
 665: // while zero-extending loads have only the long LLH.
 666: //
 667: //===----------------------------------------------------------------------===//
 668: 
 669: // 32-bit extensions from registers.
 670: def LBR : UnaryRRE<"lbr", 0xB926, sext8,  GR32, GR32>;
 671: def LHR : UnaryRRE<"lhr", 0xB927, sext16, GR32, GR32>;
 672: 
 673: // 64-bit extensions from registers.
 674: def LGBR : UnaryRRE<"lgbr", 0xB906, sext8,  GR64, GR64>;
 675: def LGHR : UnaryRRE<"lghr", 0xB907, sext16, GR64, GR64>;
 676: def LGFR : UnaryRRE<"lgfr", 0xB914, sext32, GR64, GR32>;
 677: 
 678: let Defs = [CC], CCValues = 0xE, CompareZeroCCMask = 0xE in
 679:   def LTGFR : UnaryRRE<"ltgfr", 0xB912, null_frag, GR64, GR32>;
 680: 
```
- **EN**: This block declares or refines TableGen records such as `LOCG`, `STOC`, `STOCG`, `LOCRAsm`, `LOCGRAsm`, `LOCAsm`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `LOCG`, `STOC`, `STOCG`, `LOCRAsm`, `LOCGRAsm`, `LOCAsm` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 681-720 / 第 681-720 行
```tablegen
 681: // Match 32-to-64-bit sign extensions in which the source is already
 682: // in a 64-bit register.
 683: def : Pat<(sext_inreg GR64:$src, i32),
 684:           (LGFR (EXTRACT_SUBREG GR64:$src, subreg_l32))>;
 685: 
 686: let isReMaterializable = 1 in {
 687:   // 32-bit extensions from 8-bit memory.  LBMux expands to LB or LBH,
 688:   // depending on the choice of register.
 689:   def LBMux : UnaryRXYPseudo<"lb", z_asextloadi8, GRX32, 1>,
 690:               Requires<[FeatureHighWord]>;
 691:   def LB  : UnaryRXY<"lb", 0xE376, z_asextloadi8, GR32, 1>;
 692:   def LBH : UnaryRXY<"lbh", 0xE3C0, z_asextloadi8, GRH32, 1>,
 693:             Requires<[FeatureHighWord]>;
 694: 
 695:   // 32-bit extensions from 16-bit memory.  LHMux expands to LH or LHH,
 696:   // depending on the choice of register.
 697:   def LHMux : UnaryRXYPseudo<"lh", z_asextloadi16, GRX32, 2>,
 698:               Requires<[FeatureHighWord]>;
 699:   defm LH   : UnaryRXPair<"lh", 0x48, 0xE378, z_asextloadi16, GR32, 2>;
 700:   def  LHH  : UnaryRXY<"lhh", 0xE3C4, z_asextloadi16, GRH32, 2>,
 701:               Requires<[FeatureHighWord]>;
 702:   def  LHRL : UnaryRILPC<"lhrl", 0xC45, aligned_z_asextloadi16, GR32>;
 703: 
 704:   // 64-bit extensions from memory.
 705:   def LGB   : UnaryRXY<"lgb", 0xE377, z_asextloadi8,  GR64, 1>;
 706:   def LGH   : UnaryRXY<"lgh", 0xE315, z_asextloadi16, GR64, 2>;
 707:   def LGF   : UnaryRXY<"lgf", 0xE314, z_asextloadi32, GR64, 4>;
 708:   def LGHRL : UnaryRILPC<"lghrl", 0xC44, aligned_z_asextloadi16, GR64>;
 709:   def LGFRL : UnaryRILPC<"lgfrl", 0xC4C, aligned_z_asextloadi32, GR64>;
 710: }
 711: let Defs = [CC], CCValues = 0xE, CompareZeroCCMask = 0xE in
 712:   def LTGF : UnaryRXY<"ltgf", 0xE332, z_asextloadi32, GR64, 4>;
 713: 
 714: //===----------------------------------------------------------------------===//
 715: // Zero extensions
 716: //===----------------------------------------------------------------------===//
 717: 
 718: // 32-bit extensions from registers.
 719: 
 720: // Expands to LLCR or RISB[LH]G, depending on the choice of registers.
```
- **EN**: This block declares or refines TableGen records such as `LBMux`, `LB`, `LBH`, `LHMux`, `LH`, `LHH`.
- **CN**: 该代码块声明或细化了 `LBMux`, `LB`, `LBH`, `LHMux`, `LH`, `LHH` 等 TableGen 记录。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: def LLCRMux : UnaryRRPseudo<"llcr", zext8, GRX32, GRX32>,
 722:               Requires<[FeatureHighWord]>;
 723: def LLCR    : UnaryRRE<"llcr", 0xB994, zext8,  GR32, GR32>;
 724: // Expands to LLHR or RISB[LH]G, depending on the choice of registers.
 725: def LLHRMux : UnaryRRPseudo<"llhr", zext16, GRX32, GRX32>,
 726:               Requires<[FeatureHighWord]>;
 727: def LLHR    : UnaryRRE<"llhr", 0xB995, zext16, GR32, GR32>;
 728: 
 729: // 64-bit extensions from registers.
 730: def LLGCR : UnaryRRE<"llgcr", 0xB984, zext8,  GR64, GR64>;
 731: def LLGHR : UnaryRRE<"llghr", 0xB985, zext16, GR64, GR64>;
 732: def LLGFR : UnaryRRE<"llgfr", 0xB916, zext32, GR64, GR32>;
 733: 
 734: // Match 32-to-64-bit zero extensions in which the source is already
 735: // in a 64-bit register.
 736: def : Pat<(and GR64:$src, 0xffffffff),
 737:           (LLGFR (EXTRACT_SUBREG GR64:$src, subreg_l32))>;
 738: 
 739: let isReMaterializable = 1 in {
 740:   // 32-bit extensions from 8-bit memory.  LLCMux expands to LLC or LLCH,
 741:   // depending on the choice of register.
 742:   def LLCMux : UnaryRXYPseudo<"llc", z_azextloadi8, GRX32, 1>,
 743:                Requires<[FeatureHighWord]>;
 744:   def LLC  : UnaryRXY<"llc", 0xE394, z_azextloadi8, GR32, 1>;
 745:   def LLCH : UnaryRXY<"llch", 0xE3C2, z_azextloadi8, GRH32, 1>,
 746:              Requires<[FeatureHighWord]>;
 747: 
 748:   // 32-bit extensions from 16-bit memory.  LLHMux expands to LLH or LLHH,
 749:   // depending on the choice of register.
 750:   def LLHMux : UnaryRXYPseudo<"llh", z_azextloadi16, GRX32, 2>,
 751:                Requires<[FeatureHighWord]>;
 752:   def LLH   : UnaryRXY<"llh", 0xE395, z_azextloadi16, GR32, 2>;
 753:   def LLHH  : UnaryRXY<"llhh", 0xE3C6, z_azextloadi16, GRH32, 2>,
 754:               Requires<[FeatureHighWord]>;
 755:   def LLHRL : UnaryRILPC<"llhrl", 0xC42, aligned_z_azextloadi16, GR32>;
 756: 
 757:   // 64-bit extensions from memory.
 758:   def LLGC   : UnaryRXY<"llgc", 0xE390, z_azextloadi8,  GR64, 1>;
 759:   def LLGH   : UnaryRXY<"llgh", 0xE391, z_azextloadi16, GR64, 2>;
 760:   def LLGF   : UnaryRXY<"llgf", 0xE316, z_azextloadi32, GR64, 4>;
```
- **EN**: This block declares or refines TableGen records such as `LLCRMux`, `LLCR`, `LLHRMux`, `LLHR`, `LLGCR`, `LLGHR`.
- **CN**: 该代码块声明或细化了 `LLCRMux`, `LLCR`, `LLHRMux`, `LLHR`, `LLGCR`, `LLGHR` 等 TableGen 记录。

### Lines 761-800 / 第 761-800 行
```tablegen
 761:   def LLGHRL : UnaryRILPC<"llghrl", 0xC46, aligned_z_azextloadi16, GR64>;
 762:   def LLGFRL : UnaryRILPC<"llgfrl", 0xC4E, aligned_z_azextloadi32, GR64>;
 763: }
 764: 
 765: // 31-to-64-bit zero extensions.
 766: def LLGTR : UnaryRRE<"llgtr", 0xB917, null_frag, GR64, GR64>;
 767: def LLGT  : UnaryRXY<"llgt",  0xE317, null_frag, GR64, 4>;
 768: def : Pat<(and GR64:$src, 0x7fffffff),
 769:           (LLGTR GR64:$src)>;
 770: def : Pat<(and (i64 (z_azextloadi32 bdxaddr20only:$src)), 0x7fffffff),
 771:           (LLGT bdxaddr20only:$src)>;
 772: 
 773: // Load and zero rightmost byte.
 774: let Predicates = [FeatureLoadAndZeroRightmostByte] in {
 775:   def LLZRGF : UnaryRXY<"llzrgf", 0xE33A, null_frag, GR64, 4>;
 776:   def : Pat<(and (i64 (z_azextloadi32 bdxaddr20only:$src)), 0xffffff00),
 777:             (LLZRGF bdxaddr20only:$src)>;
 778: }
 779: 
 780: // Load and trap.
 781: let Predicates = [FeatureLoadAndTrap], hasSideEffects = 1 in {
 782:   def LLGFAT : UnaryRXY<"llgfat", 0xE39D, null_frag, GR64, 4>;
 783:   def LLGTAT : UnaryRXY<"llgtat", 0xE39C, null_frag, GR64, 4>;
 784: }
 785: 
 786: // Extend GR64s to GR128s.
 787: let usesCustomInserter = 1, hasNoSchedulingInfo = 1 in
 788:   def ZEXT128 : Pseudo<(outs GR128:$dst), (ins GR64:$src), []>;
 789: 
 790: //===----------------------------------------------------------------------===//
 791: // "Any" extensions
 792: //===----------------------------------------------------------------------===//
 793: 
 794: // Use subregs to populate the "don't care" bits in a 32-bit to 64-bit anyext.
 795: def : Pat<(i64 (anyext GR32:$src)),
 796:           (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GR32:$src, subreg_l32)>;
 797: 
 798: // Extend GR64s to GR128s.
 799: let usesCustomInserter = 1, hasNoSchedulingInfo = 1 in
 800:   def AEXT128 : Pseudo<(outs GR128:$dst), (ins GR64:$src), []>;
```
- **EN**: This block declares or refines TableGen records such as `LLGHRL`, `LLGFRL`, `LLGTR`, `LLGT`, `LLZRGF`, `LLGFAT`.
- **CN**: 该代码块声明或细化了 `LLGHRL`, `LLGFRL`, `LLGTR`, `LLGT`, `LLZRGF`, `LLGFAT` 等 TableGen 记录。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: 
 802: //===----------------------------------------------------------------------===//
 803: // Truncations
 804: //===----------------------------------------------------------------------===//
 805: 
 806: // Truncations of 64-bit registers to 32-bit registers.
 807: def : Pat<(i32 (trunc GR64:$src)),
 808:           (EXTRACT_SUBREG GR64:$src, subreg_l32)>;
 809: 
 810: // Truncations of 32-bit registers to 8-bit memory.  STCMux expands to
 811: // STC, STCY or STCH, depending on the choice of register.
 812: def STCMux : StoreRXYPseudo<truncstorei8, GRX32, 1>,
 813:              Requires<[FeatureHighWord]>;
 814: defm STC : StoreRXPair<"stc", 0x42, 0xE372, truncstorei8, GR32, 1>;
 815: def STCH : StoreRXY<"stch", 0xE3C3, truncstorei8, GRH32, 1>,
 816:            Requires<[FeatureHighWord]>;
 817: 
 818: // Truncations of 32-bit registers to 16-bit memory.  STHMux expands to
 819: // STH, STHY or STHH, depending on the choice of register.
 820: def STHMux : StoreRXYPseudo<truncstorei16, GRX32, 1>,
 821:              Requires<[FeatureHighWord]>;
 822: defm STH : StoreRXPair<"sth", 0x40, 0xE370, truncstorei16, GR32, 2>;
 823: def STHH : StoreRXY<"sthh", 0xE3C7, truncstorei16, GRH32, 2>,
 824:            Requires<[FeatureHighWord]>;
 825: def STHRL : StoreRILPC<"sthrl", 0xC47, aligned_truncstorei16, GR32>;
 826: 
 827: // Truncations of 64-bit registers to memory.
 828: defm : StoreGR64Pair<STC, STCY, truncstorei8>;
 829: defm : StoreGR64Pair<STH, STHY, truncstorei16>;
 830: def  : StoreGR64PC<STHRL, aligned_truncstorei16>;
 831: defm : StoreGR64Pair<ST, STY, truncstorei32>;
 832: def  : StoreGR64PC<STRL, aligned_truncstorei32>;
 833: 
 834: // Store characters under mask -- not (yet) used for codegen.
 835: defm STCM : StoreBinaryRSPair<"stcm", 0xBE, 0xEB2D, GR32, 0>;
 836: def STCMH : StoreBinaryRSY<"stcmh", 0xEB2C, GRH32, 0>;
 837: 
 838: //===----------------------------------------------------------------------===//
 839: // Multi-register moves
 840: //===----------------------------------------------------------------------===//
```
- **EN**: This block declares or refines TableGen records such as `STCMux`, `STC`, `STCH`, `STHMux`, `STH`, `STHH`.
- **CN**: 该代码块声明或细化了 `STCMux`, `STC`, `STCH`, `STHMux`, `STH`, `STHH` 等 TableGen 记录。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: 
 842: // Multi-register loads.
 843: defm LM : LoadMultipleRSPair<"lm", 0x98, 0xEB98, GR32>;
 844: def LMG : LoadMultipleRSY<"lmg", 0xEB04, GR64>;
 845: def LMH : LoadMultipleRSY<"lmh", 0xEB96, GRH32>;
 846: def LMD : LoadMultipleSSe<"lmd", 0xEF, GR64>;
 847: 
 848: // Multi-register stores.
 849: defm STM : StoreMultipleRSPair<"stm", 0x90, 0xEB90, GR32>;
 850: def STMG : StoreMultipleRSY<"stmg", 0xEB24, GR64>;
 851: def STMH : StoreMultipleRSY<"stmh", 0xEB26, GRH32>;
 852: 
 853: //===----------------------------------------------------------------------===//
 854: // Byte swaps
 855: //===----------------------------------------------------------------------===//
 856: 
 857: // Byte-swapping register moves.
 858: def LRVR  : UnaryRRE<"lrvr",  0xB91F, bswap, GR32, GR32>;
 859: def LRVGR : UnaryRRE<"lrvgr", 0xB90F, bswap, GR64, GR64>;
 860: 
 861: // Byte-swapping loads.
 862: def LRVH : UnaryRXY<"lrvh", 0xE31F, z_loadbswap16, GR32, 2>;
 863: def LRV  : UnaryRXY<"lrv",  0xE31E, z_loadbswap32, GR32, 4>;
 864: def LRVG : UnaryRXY<"lrvg", 0xE30F, z_loadbswap64, GR64, 8>;
 865: 
 866: // Byte-swapping stores.
 867: def STRVH : StoreRXY<"strvh", 0xE33F, z_storebswap16, GR32, 2>;
 868: def STRV  : StoreRXY<"strv",  0xE33E, z_storebswap32, GR32, 4>;
 869: def STRVG : StoreRXY<"strvg", 0xE32F, z_storebswap64, GR64, 8>;
 870: 
 871: // Byte-swapping memory-to-memory moves.
 872: let mayLoad = 1, mayStore = 1 in
 873:   def MVCIN : SideEffectBinarySSa<"mvcin", 0xE8>;
 874: 
 875: //===----------------------------------------------------------------------===//
 876: // Load address instructions
 877: //===----------------------------------------------------------------------===//
 878: 
 879: // Load BDX-style addresses.
 880: let isAsCheapAsAMove = 1, isReMaterializable = 1 in
```
- **EN**: This block declares or refines TableGen records such as `LM`, `LMG`, `LMH`, `LMD`, `STM`, `STMG`.
- **CN**: 该代码块声明或细化了 `LM`, `LMG`, `LMH`, `LMD`, `STM`, `STMG` 等 TableGen 记录。

### Lines 881-920 / 第 881-920 行
```tablegen
 881:   defm LA : LoadAddressRXPair<"la", 0x41, 0xE371, bitconvert>;
 882: 
 883: // Load a PC-relative address.  There's no version of this instruction
 884: // with a 16-bit offset, so there's no relaxation.
 885: let isAsCheapAsAMove = 1, isMoveImm = 1, isReMaterializable = 1 in
 886:   def LARL : LoadAddressRIL<"larl", 0xC00, bitconvert>;
 887: 
 888: // Load the Global Offset Table address.  This will be lowered into a
 889: //     larl $R1, _GLOBAL_OFFSET_TABLE_
 890: // instruction.
 891: def GOT : Alias<6, (outs GR64:$R1), (ins),
 892:                 [(set GR64:$R1, (global_offset_table))]>;
 893: 
 894: // Load (logical) indexed address.
 895: let Predicates = [FeatureMiscellaneousExtensions4] in {
 896:   defm LXAB  : LoadIndexedAddressRXY<"lxab",  0xE360, sext32>;
 897:   defm LXAH  : LoadIndexedAddressRXY<"lxah",  0xE362, sext32, shl1>;
 898:   defm LXAF  : LoadIndexedAddressRXY<"lxaf",  0xE364, sext32, shl2>;
 899:   defm LXAG  : LoadIndexedAddressRXY<"lxag",  0xE366, sext32, shl3>;
 900:   defm LXAQ  : LoadIndexedAddressRXY<"lxaq",  0xE368, sext32, shl4>;
 901:   defm LLXAB : LoadIndexedAddressRXY<"llxab", 0xE361, zext32>;
 902:   defm LLXAH : LoadIndexedAddressRXY<"llxah", 0xE363, zext32, shl1>;
 903:   defm LLXAF : LoadIndexedAddressRXY<"llxaf", 0xE365, zext32, shl2>;
 904:   defm LLXAG : LoadIndexedAddressRXY<"llxag", 0xE367, zext32, shl3>;
 905:   defm LLXAQ : LoadIndexedAddressRXY<"llxaq", 0xE369, zext32, shl4>;
 906: 
 907:   // Peepholes to use load (logical) indexed address to implement
 908:   // add + shift of an already extended value.
 909:   def : Pat<(add ADDR64:$base, (shl1 (assertsext32 ADDR64:$index))),
 910:             (LXAH ADDR64:$base, 0, (EXTRACT_SUBREG ADDR64:$index, subreg_l32))>;
 911:   def : Pat<(add ADDR64:$base, (shl2 (assertsext32 ADDR64:$index))),
 912:             (LXAF ADDR64:$base, 0, (EXTRACT_SUBREG ADDR64:$index, subreg_l32))>;
 913:   def : Pat<(add ADDR64:$base, (shl3 (assertsext32 ADDR64:$index))),
 914:             (LXAG ADDR64:$base, 0, (EXTRACT_SUBREG ADDR64:$index, subreg_l32))>;
 915:   def : Pat<(add ADDR64:$base, (shl4 (assertsext32 ADDR64:$index))),
 916:             (LXAQ ADDR64:$base, 0, (EXTRACT_SUBREG ADDR64:$index, subreg_l32))>;
 917:   def : Pat<(add ADDR64:$base, (shl1 (assertzext32 ADDR64:$index))),
 918:             (LLXAH ADDR64:$base, 0, (EXTRACT_SUBREG ADDR64:$index, subreg_l32))>;
 919:   def : Pat<(add ADDR64:$base, (shl2 (assertzext32 ADDR64:$index))),
 920:             (LLXAF ADDR64:$base, 0, (EXTRACT_SUBREG ADDR64:$index, subreg_l32))>;
```
- **EN**: This block declares or refines TableGen records such as `LA`, `LARL`, `GOT`, `LXAB`, `LXAH`, `LXAF`.
- **CN**: 该代码块声明或细化了 `LA`, `LARL`, `GOT`, `LXAB`, `LXAH`, `LXAF` 等 TableGen 记录。

### Lines 921-960 / 第 921-960 行
```tablegen
 921:   def : Pat<(add ADDR64:$base, (shl3 (assertzext32 ADDR64:$index))),
 922:             (LLXAG ADDR64:$base, 0, (EXTRACT_SUBREG ADDR64:$index, subreg_l32))>;
 923:   def : Pat<(add ADDR64:$base, (shl4 (assertzext32 ADDR64:$index))),
 924:             (LLXAQ ADDR64:$base, 0, (EXTRACT_SUBREG ADDR64:$index, subreg_l32))>;
 925: }
 926: 
 927: //===----------------------------------------------------------------------===//
 928: // Absolute and Negation
 929: //===----------------------------------------------------------------------===//
 930: 
 931: let Defs = [CC] in {
 932:   let CCValues = 0xF, CompareZeroCCMask = 0x8 in {
 933:     def LPR  : UnaryRR <"lpr",  0x10,   abs, GR32, GR32>;
 934:     def LPGR : UnaryRRE<"lpgr", 0xB900, abs, GR64, GR64>;
 935:   }
 936:   let CCValues = 0xE, CompareZeroCCMask = 0xE in
 937:     def LPGFR : UnaryRRE<"lpgfr", 0xB910, null_frag, GR64, GR32>;
 938: }
 939: defm : SXU<abs, LPGFR>;
 940: 
 941: let Defs = [CC] in {
 942:   let CCValues = 0xF, CompareZeroCCMask = 0x8 in {
 943:     def LNR  : UnaryRR <"lnr",  0x11,   z_inegabs, GR32, GR32>;
 944:     def LNGR : UnaryRRE<"lngr", 0xB901, z_inegabs, GR64, GR64>;
 945:   }
 946:   let CCValues = 0xE, CompareZeroCCMask = 0xE in
 947:     def LNGFR : UnaryRRE<"lngfr", 0xB911, null_frag, GR64, GR32>;
 948: }
 949: defm : SXU<z_inegabs, LNGFR>;
 950: 
 951: let Defs = [CC] in {
 952:   let CCValues = 0xF, CompareZeroCCMask = 0x8 in {
 953:     def LCR  : UnaryRR <"lcr",  0x13,   ineg, GR32, GR32>;
 954:     def LCGR : UnaryRRE<"lcgr", 0xB903, ineg, GR64, GR64>;
 955:   }
 956:   let CCValues = 0xE, CompareZeroCCMask = 0xE in
 957:     def LCGFR : UnaryRRE<"lcgfr", 0xB913, null_frag, GR64, GR32>;
 958: }
 959: defm : SXU<ineg, LCGFR>;
 960: 
```
- **EN**: This block declares or refines TableGen records such as `LPR`, `LPGR`, `LPGFR`, `LNR`, `LNGR`, `LNGFR`.
- **CN**: 该代码块声明或细化了 `LPR`, `LPGR`, `LPGFR`, `LNR`, `LNGR`, `LNGFR` 等 TableGen 记录。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: //===----------------------------------------------------------------------===//
 962: // Insertion
 963: //===----------------------------------------------------------------------===//
 964: 
 965: let isCodeGenOnly = 1 in
 966:   defm IC32 : BinaryRXPair<"ic", 0x43, 0xE373, inserti8, GR32, z_azextloadi8, 1>;
 967: defm IC : BinaryRXPair<"ic", 0x43, 0xE373, inserti8, GR64, z_azextloadi8, 1>;
 968: 
 969: defm : InsertMem<"inserti8", IC32,  GR32, z_azextloadi8, bdxaddr12pair>;
 970: defm : InsertMem<"inserti8", IC32Y, GR32, z_azextloadi8, bdxaddr20pair>;
 971: 
 972: defm : InsertMem<"inserti8", IC,  GR64, z_azextloadi8, bdxaddr12pair>;
 973: defm : InsertMem<"inserti8", ICY, GR64, z_azextloadi8, bdxaddr20pair>;
 974: 
 975: // Insert characters under mask -- not (yet) used for codegen.
 976: let Defs = [CC] in {
 977:   defm ICM : TernaryRSPair<"icm", 0xBF, 0xEB81, GR32, 0>;
 978:   def ICMH : TernaryRSY<"icmh", 0xEB80, GRH32, 0>;
 979: }
 980: 
 981: // Insertions of a 16-bit immediate, leaving other bits unaffected.
 982: // We don't have or_as_insert equivalents of these operations because
 983: // OI is available instead.
 984: //
 985: // IIxMux expands to II[LH]x, depending on the choice of register.
 986: def IILMux : BinaryRIPseudo<insertll, GRX32, imm32ll16>,
 987:              Requires<[FeatureHighWord]>;
 988: def IIHMux : BinaryRIPseudo<insertlh, GRX32, imm32lh16>,
 989:              Requires<[FeatureHighWord]>;
 990: def IILL : BinaryRI<"iill", 0xA53, insertll, GR32, imm32ll16>;
 991: def IILH : BinaryRI<"iilh", 0xA52, insertlh, GR32, imm32lh16>;
 992: def IIHL : BinaryRI<"iihl", 0xA51, insertll, GRH32, imm32ll16>;
 993: def IIHH : BinaryRI<"iihh", 0xA50, insertlh, GRH32, imm32lh16>;
 994: def IILL64 : BinaryAliasRI<insertll64, GR64, imm64ll16>;
 995: def IILH64 : BinaryAliasRI<insertlh64, GR64, imm64lh16>;
 996: def IIHL64 : BinaryAliasRI<inserthl64, GR64, imm64hl16>;
 997: def IIHH64 : BinaryAliasRI<inserthh64, GR64, imm64hh16>;
 998: 
 999: // ...likewise for 32-bit immediates.  For GR32s this is a general
1000: // full-width move.  (We use IILF rather than something like LLILF
```
- **EN**: This block declares or refines TableGen records such as `IC32`, `IC`, `ICM`, `ICMH`, `IILMux`, `IIHMux`.
- **CN**: 该代码块声明或细化了 `IC32`, `IC`, `ICM`, `ICMH`, `IILMux`, `IIHMux` 等 TableGen 记录。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: // for 32-bit moves because IILF leaves the upper 32 bits of the
1002: // GR64 unchanged.)
1003: let isAsCheapAsAMove = 1, isMoveImm = 1, isReMaterializable = 1 in {
1004:   def IIFMux : UnaryRIPseudo<bitconvert, GRX32, uimm32>,
1005:                Requires<[FeatureHighWord]>;
1006:   def IILF : UnaryRIL<"iilf", 0xC09, bitconvert, GR32, uimm32>;
1007:   def IIHF : UnaryRIL<"iihf", 0xC08, bitconvert, GRH32, uimm32>;
1008: }
1009: def LFI : InstAlias<"lfi\t$R1, $RI1", (IILF GR32:$R1, uimm32:$RI1)>;
1010: def IILF64 : BinaryAliasRIL<insertlf, GR64, imm64lf32>;
1011: def IIHF64 : BinaryAliasRIL<inserthf, GR64, imm64hf32>;
1012: 
1013: // An alternative model of inserthf, with the first operand being
1014: // a zero-extended value.
1015: def : Pat<(or (zext32 GR32:$src), imm64hf32:$imm),
1016:           (IIHF64 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GR32:$src, subreg_l32),
1017:                   imm64hf32:$imm)>;
1018: 
1019: //===----------------------------------------------------------------------===//
1020: // Addition
1021: //===----------------------------------------------------------------------===//
1022: 
1023: // Addition producing a signed overflow flag.
1024: let Defs = [CC], CCValues = 0xF, CCIfNoSignedWrap = 1 in {
1025:   // Addition of a register.
1026:   let isCommutable = 1 in {
1027:     defm AR : BinaryRRAndK<"ar", 0x1A, 0xB9F8, z_sadd, GR32, GR32>;
1028:     defm AGR : BinaryRREAndK<"agr", 0xB908, 0xB9E8, z_sadd, GR64, GR64>;
1029:   }
1030:   def AGFR : BinaryRRE<"agfr", 0xB918, null_frag, GR64, GR32>;
1031: 
1032:   // Addition to a high register.
1033:   def AHHHR : BinaryRRFa<"ahhhr", 0xB9C8, null_frag, GRH32, GRH32, GRH32>,
1034:               Requires<[FeatureHighWord]>;
1035:   def AHHLR : BinaryRRFa<"ahhlr", 0xB9D8, null_frag, GRH32, GRH32, GR32>,
1036:               Requires<[FeatureHighWord]>;
1037: 
1038:   // Addition of signed 16-bit immediates.
1039:   defm AHIMux : BinaryRIAndKPseudo<"ahimux", z_sadd, GRX32, imm32sx16>;
1040:   defm AHI  : BinaryRIAndK<"ahi",  0xA7A, 0xECD8, z_sadd, GR32, imm32sx16>;
```
- **EN**: This block declares or refines TableGen records such as `IIFMux`, `IILF`, `IIHF`, `LFI`, `IILF64`, `IIHF64`.
- **CN**: 该代码块声明或细化了 `IIFMux`, `IILF`, `IIHF`, `LFI`, `IILF64`, `IIHF64` 等 TableGen 记录。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041:   defm AGHI : BinaryRIAndK<"aghi", 0xA7B, 0xECD9, z_sadd, GR64, imm64sx16>;
1042: 
1043:   // Addition of signed 32-bit immediates.
1044:   def AFIMux : BinaryRIPseudo<z_sadd, GRX32, simm32>,
1045:                Requires<[FeatureHighWord]>;
1046:   def AFI  : BinaryRIL<"afi",  0xC29, z_sadd, GR32, simm32>;
1047:   def AIH  : BinaryRIL<"aih",  0xCC8, z_sadd, GRH32, simm32>,
1048:              Requires<[FeatureHighWord]>;
1049:   def AGFI : BinaryRIL<"agfi", 0xC28, z_sadd, GR64, imm64sx32>;
1050: 
1051:   // Addition of memory.
1052:   defm AH  : BinaryRXPair<"ah", 0x4A, 0xE37A, z_sadd, GR32, z_asextloadi16, 2>;
1053:   defm A   : BinaryRXPairAndPseudo<"a",  0x5A, 0xE35A, z_sadd, GR32, z_load, 4>;
1054:   def  AGH : BinaryRXY<"agh", 0xE338, z_sadd, GR64, z_asextloadi16, 2>,
1055:              Requires<[FeatureMiscellaneousExtensions2]>;
1056:   def  AGF : BinaryRXY<"agf", 0xE318, z_sadd, GR64, z_asextloadi32, 4>;
1057:   defm AG  : BinaryRXYAndPseudo<"ag",  0xE308, z_sadd, GR64, z_load, 8>;
1058: 
1059:   // Addition to memory.
1060:   def ASI  : BinarySIY<"asi",  0xEB6A, add, imm32sx8>;
1061:   def AGSI : BinarySIY<"agsi", 0xEB7A, add, imm64sx8>;
1062: }
1063: defm : SXB<z_sadd, GR64, AGFR>;
1064: 
1065: // Addition producing a carry.
1066: let Defs = [CC], CCValues = 0xF, IsLogical = 1 in {
1067:   // Addition of a register.
1068:   let isCommutable = 1 in {
1069:     defm ALR : BinaryRRAndK<"alr", 0x1E, 0xB9FA, z_uadd, GR32, GR32>;
1070:     defm ALGR : BinaryRREAndK<"algr", 0xB90A, 0xB9EA, z_uadd, GR64, GR64>;
1071:   }
1072:   def ALGFR : BinaryRRE<"algfr", 0xB91A, null_frag, GR64, GR32>;
1073: 
1074:   // Addition to a high register.
1075:   def ALHHHR : BinaryRRFa<"alhhhr", 0xB9CA, null_frag, GRH32, GRH32, GRH32>,
1076:                Requires<[FeatureHighWord]>;
1077:   def ALHHLR : BinaryRRFa<"alhhlr", 0xB9DA, null_frag, GRH32, GRH32, GR32>,
1078:                Requires<[FeatureHighWord]>;
1079: 
1080:   // Addition of signed 16-bit immediates.
```
- **EN**: This block declares or refines TableGen records such as `AGHI`, `AFIMux`, `AFI`, `AIH`, `AGFI`, `AH`.
- **CN**: 该代码块声明或细化了 `AGHI`, `AFIMux`, `AFI`, `AIH`, `AGFI`, `AH` 等 TableGen 记录。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081:   def ALHSIK  : BinaryRIE<"alhsik",  0xECDA, z_uadd, GR32, imm32sx16>,
1082:                 Requires<[FeatureDistinctOps]>;
1083:   def ALGHSIK : BinaryRIE<"alghsik", 0xECDB, z_uadd, GR64, imm64sx16>,
1084:                 Requires<[FeatureDistinctOps]>;
1085: 
1086:   // Addition of unsigned 32-bit immediates.
1087:   def ALFI  : BinaryRIL<"alfi",  0xC2B, z_uadd, GR32, uimm32>;
1088:   def ALGFI : BinaryRIL<"algfi", 0xC2A, z_uadd, GR64, imm64zx32>;
1089: 
1090:   // Addition of signed 32-bit immediates.
1091:   def ALSIH : BinaryRIL<"alsih", 0xCCA, null_frag, GRH32, simm32>,
1092:               Requires<[FeatureHighWord]>;
1093: 
1094:   // Addition of memory.
1095:   defm AL   : BinaryRXPairAndPseudo<"al", 0x5E, 0xE35E, z_uadd, GR32, z_load, 4>;
1096:   def  ALGF : BinaryRXY<"algf", 0xE31A, z_uadd, GR64, z_azextloadi32, 4>;
1097:   defm ALG  : BinaryRXYAndPseudo<"alg",  0xE30A, z_uadd, GR64, z_load, 8>;
1098: 
1099:   // Addition to memory.
1100:   def ALSI  : BinarySIY<"alsi",  0xEB6E, null_frag, imm32sx8>;
1101:   def ALGSI : BinarySIY<"algsi", 0xEB7E, null_frag, imm64sx8>;
1102: }
1103: defm : ZXB<z_uadd, GR64, ALGFR>;
1104: 
1105: // Addition producing and using a carry.
1106: let Defs = [CC], Uses = [CC], CCValues = 0xF, IsLogical = 1 in {
1107:   // Addition of a register.
1108:   def ALCR  : BinaryRRE<"alcr",  0xB998, z_addcarry, GR32, GR32>;
1109:   def ALCGR : BinaryRRE<"alcgr", 0xB988, z_addcarry, GR64, GR64>;
1110: 
1111:   // Addition of memory.
1112:   def ALC  : BinaryRXY<"alc",  0xE398, z_addcarry, GR32, z_load, 4>;
1113:   def ALCG : BinaryRXY<"alcg", 0xE388, z_addcarry, GR64, z_load, 8>;
1114: }
1115: 
1116: // Addition that does not modify the condition code.
1117: def ALSIHN : BinaryRIL<"alsihn", 0xCCB, null_frag, GRH32, simm32>,
1118:              Requires<[FeatureHighWord]>;
1119: 
1120: 
```
- **EN**: This block declares or refines TableGen records such as `ALHSIK`, `ALGHSIK`, `ALFI`, `ALGFI`, `ALSIH`, `AL`.
- **CN**: 该代码块声明或细化了 `ALHSIK`, `ALGHSIK`, `ALFI`, `ALGFI`, `ALSIH`, `AL` 等 TableGen 记录。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: //===----------------------------------------------------------------------===//
1122: // Subtraction
1123: //===----------------------------------------------------------------------===//
1124: 
1125: // Subtraction producing a signed overflow flag.
1126: let Defs = [CC], CCValues = 0xF, CompareZeroCCMask = 0x8,
1127:     CCIfNoSignedWrap = 1 in {
1128:   // Subtraction of a register.
1129:   defm SR : BinaryRRAndK<"sr", 0x1B, 0xB9F9, z_ssub, GR32, GR32>;
1130:   def SGFR : BinaryRRE<"sgfr", 0xB919, null_frag, GR64, GR32>;
1131:   defm SGR : BinaryRREAndK<"sgr", 0xB909, 0xB9E9, z_ssub, GR64, GR64>;
1132: 
1133:   // Subtraction from a high register.
1134:   def SHHHR : BinaryRRFa<"shhhr", 0xB9C9, null_frag, GRH32, GRH32, GRH32>,
1135:               Requires<[FeatureHighWord]>;
1136:   def SHHLR : BinaryRRFa<"shhlr", 0xB9D9, null_frag, GRH32, GRH32, GR32>,
1137:               Requires<[FeatureHighWord]>;
1138: 
1139:   // Subtraction of memory.
1140:   defm SH  : BinaryRXPair<"sh", 0x4B, 0xE37B, z_ssub, GR32, z_asextloadi16, 2>;
1141:   defm S   : BinaryRXPairAndPseudo<"s", 0x5B, 0xE35B, z_ssub, GR32, z_load, 4>;
1142:   def  SGH : BinaryRXY<"sgh", 0xE339, z_ssub, GR64, z_asextloadi16, 2>,
1143:              Requires<[FeatureMiscellaneousExtensions2]>;
1144:   def  SGF : BinaryRXY<"sgf", 0xE319, z_ssub, GR64, z_asextloadi32, 4>;
1145:   defm SG  : BinaryRXYAndPseudo<"sg",  0xE309, z_ssub, GR64, z_load, 8>;
1146: }
1147: defm : SXB<z_ssub, GR64, SGFR>;
1148: 
1149: // Subtracting an immediate is the same as adding the negated immediate.
1150: let AddedComplexity = 1 in {
1151:   def : Pat<(z_ssub GR32:$src1, imm32sx16n:$src2),
1152:             (AHIMux GR32:$src1, imm32sx16n:$src2)>,
1153:         Requires<[FeatureHighWord]>;
1154:   def : Pat<(z_ssub GR32:$src1, simm32n:$src2),
1155:             (AFIMux GR32:$src1, simm32n:$src2)>,
1156:         Requires<[FeatureHighWord]>;
1157:   def : Pat<(z_ssub GR32:$src1, imm32sx16n:$src2),
1158:             (AHI GR32:$src1, imm32sx16n:$src2)>;
1159:   def : Pat<(z_ssub GR32:$src1, simm32n:$src2),
1160:             (AFI GR32:$src1, simm32n:$src2)>;
```
- **EN**: This block declares or refines TableGen records such as `SR`, `SGFR`, `SGR`, `SHHHR`, `SHHLR`, `SH`.
- **CN**: 该代码块声明或细化了 `SR`, `SGFR`, `SGR`, `SHHHR`, `SHHLR`, `SH` 等 TableGen 记录。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161:   def : Pat<(z_ssub GR64:$src1, imm64sx16n:$src2),
1162:             (AGHI GR64:$src1, imm64sx16n:$src2)>;
1163:   def : Pat<(z_ssub GR64:$src1, imm64sx32n:$src2),
1164:             (AGFI GR64:$src1, imm64sx32n:$src2)>;
1165: }
1166: 
1167: // And vice versa in one special case, where we need to load a
1168: // constant into a register in any case, but the negated constant
1169: // requires fewer instructions to load.
1170: def : Pat<(z_saddo GR64:$src1, imm64lh16n:$src2),
1171:           (SGR GR64:$src1, (LLILH imm64lh16n:$src2))>;
1172: def : Pat<(z_saddo GR64:$src1, imm64lf32n:$src2),
1173:           (SGR GR64:$src1, (LLILF imm64lf32n:$src2))>;
1174: 
1175: // Subtraction producing a carry.
1176: let Defs = [CC], CCValues = 0x7, IsLogical = 1 in {
1177:   // Subtraction of a register.
1178:   defm SLR : BinaryRRAndK<"slr", 0x1F, 0xB9FB, z_usub, GR32, GR32>;
1179:   def SLGFR : BinaryRRE<"slgfr", 0xB91B, null_frag, GR64, GR32>;
1180:   defm SLGR : BinaryRREAndK<"slgr", 0xB90B, 0xB9EB, z_usub, GR64, GR64>;
1181: 
1182:   // Subtraction from a high register.
1183:   def SLHHHR : BinaryRRFa<"slhhhr", 0xB9CB, null_frag, GRH32, GRH32, GRH32>,
1184:                Requires<[FeatureHighWord]>;
1185:   def SLHHLR : BinaryRRFa<"slhhlr", 0xB9DB, null_frag, GRH32, GRH32, GR32>,
1186:                Requires<[FeatureHighWord]>;
1187: 
1188:   // Subtraction of unsigned 32-bit immediates.
1189:   def SLFI  : BinaryRIL<"slfi",  0xC25, z_usub, GR32, uimm32>;
1190:   def SLGFI : BinaryRIL<"slgfi", 0xC24, z_usub, GR64, imm64zx32>;
1191: 
1192:   // Subtraction of memory.
1193:   defm SL   : BinaryRXPairAndPseudo<"sl", 0x5F, 0xE35F, z_usub, GR32, z_load, 4>;
1194:   def  SLGF : BinaryRXY<"slgf", 0xE31B, z_usub, GR64, z_azextloadi32, 4>;
1195:   defm SLG  : BinaryRXYAndPseudo<"slg",  0xE30B, z_usub, GR64, z_load, 8>;
1196: }
1197: defm : ZXB<z_usub, GR64, SLGFR>;
1198: 
1199: // Subtracting an immediate is the same as adding the negated immediate.
1200: let AddedComplexity = 1 in {
```
- **EN**: This block declares or refines TableGen records such as `SLR`, `SLGFR`, `SLGR`, `SLHHHR`, `SLHHLR`, `SLFI`.
- **CN**: 该代码块声明或细化了 `SLR`, `SLGFR`, `SLGR`, `SLHHHR`, `SLHHLR`, `SLFI` 等 TableGen 记录。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201:   def : Pat<(z_usub GR32:$src1, imm32sx16n:$src2),
1202:             (ALHSIK GR32:$src1, imm32sx16n:$src2)>,
1203:         Requires<[FeatureDistinctOps]>;
1204:   def : Pat<(z_usub GR64:$src1, imm64sx16n:$src2),
1205:             (ALGHSIK GR64:$src1, imm64sx16n:$src2)>,
1206:         Requires<[FeatureDistinctOps]>;
1207: }
1208: 
1209: // And vice versa in one special case (but we prefer addition).
1210: def : Pat<(add GR64:$src1, imm64zx32n:$src2),
1211:           (SLGFI GR64:$src1, imm64zx32n:$src2)>;
1212: 
1213: // Subtraction producing and using a carry.
1214: let Defs = [CC], Uses = [CC], CCValues = 0xF, IsLogical = 1 in {
1215:   // Subtraction of a register.
1216:   def SLBR  : BinaryRRE<"slbr",  0xB999, z_subcarry, GR32, GR32>;
1217:   def SLBGR : BinaryRRE<"slbgr", 0xB989, z_subcarry, GR64, GR64>;
1218: 
1219:   // Subtraction of memory.
1220:   def SLB  : BinaryRXY<"slb",  0xE399, z_subcarry, GR32, z_load, 4>;
1221:   def SLBG : BinaryRXY<"slbg", 0xE389, z_subcarry, GR64, z_load, 8>;
1222: }
1223: 
1224: 
1225: //===----------------------------------------------------------------------===//
1226: // AND
1227: //===----------------------------------------------------------------------===//
1228: 
1229: let Defs = [CC] in {
1230:   // ANDs of a register.
1231:   let isCommutable = 1, CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1232:     defm NR : BinaryRRAndK<"nr", 0x14, 0xB9F4, and, GR32, GR32>;
1233:     defm NGR : BinaryRREAndK<"ngr", 0xB980, 0xB9E4, and, GR64, GR64>;
1234:   }
1235: 
1236:   let isConvertibleToThreeAddress = 1 in {
1237:     // ANDs of a 16-bit immediate, leaving other bits unaffected.
1238:     // The CC result only reflects the 16-bit field, not the full register.
1239:     //
1240:     // NIxMux expands to NI[LH]x, depending on the choice of register.
```
- **EN**: This block declares or refines TableGen records such as `SLBR`, `SLBGR`, `SLB`, `SLBG`, `NR`, `NGR`.
- **CN**: 该代码块声明或细化了 `SLBR`, `SLBGR`, `SLB`, `SLBG`, `NR`, `NGR` 等 TableGen 记录。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241:     def NILMux : BinaryRIPseudo<and, GRX32, imm32ll16c>,
1242:                  Requires<[FeatureHighWord]>;
1243:     def NIHMux : BinaryRIPseudo<and, GRX32, imm32lh16c>,
1244:                  Requires<[FeatureHighWord]>;
1245:     def NILL : BinaryRI<"nill", 0xA57, and, GR32, imm32ll16c>;
1246:     def NILH : BinaryRI<"nilh", 0xA56, and, GR32, imm32lh16c>;
1247:     def NIHL : BinaryRI<"nihl", 0xA55, and, GRH32, imm32ll16c>;
1248:     def NIHH : BinaryRI<"nihh", 0xA54, and, GRH32, imm32lh16c>;
1249:     def NILL64 : BinaryAliasRI<and, GR64, imm64ll16c>;
1250:     def NILH64 : BinaryAliasRI<and, GR64, imm64lh16c>;
1251:     def NIHL64 : BinaryAliasRI<and, GR64, imm64hl16c>;
1252:     def NIHH64 : BinaryAliasRI<and, GR64, imm64hh16c>;
1253: 
1254:     // ANDs of a 32-bit immediate, leaving other bits unaffected.
1255:     // The CC result only reflects the 32-bit field, which means we can
1256:     // use it as a zero indicator for i32 operations but not otherwise.
1257:     let CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1258:       // Expands to NILF or NIHF, depending on the choice of register.
1259:       def NIFMux : BinaryRIPseudo<and, GRX32, uimm32>,
1260:                    Requires<[FeatureHighWord]>;
1261:       def NILF : BinaryRIL<"nilf", 0xC0B, and, GR32, uimm32>;
1262:       def NIHF : BinaryRIL<"nihf", 0xC0A, and, GRH32, uimm32>;
1263:     }
1264:     def NILF64 : BinaryAliasRIL<and, GR64, imm64lf32c>;
1265:     def NIHF64 : BinaryAliasRIL<and, GR64, imm64hf32c>;
1266:   }
1267: 
1268:   // ANDs of memory.
1269:   let CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1270:     defm N  : BinaryRXPairAndPseudo<"n", 0x54, 0xE354, and, GR32, z_load, 4>;
1271:     defm NG : BinaryRXYAndPseudo<"ng", 0xE380, and, GR64, z_load, 8>;
1272:   }
1273: 
1274:   // AND to memory
1275:   defm NI : BinarySIPair<"ni", 0x94, 0xEB54, null_frag, imm32zx8>;
1276: 
1277:   // Block AND.
1278:   let mayLoad = 1, mayStore = 1 in
1279:     defm NC : MemorySS<"nc", 0xD4, z_nc>;
1280: }
```
- **EN**: This block declares or refines TableGen records such as `NILMux`, `NIHMux`, `NILL`, `NILH`, `NIHL`, `NIHH`.
- **CN**: 该代码块声明或细化了 `NILMux`, `NIHMux`, `NILL`, `NILH`, `NIHL`, `NIHH` 等 TableGen 记录。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281: defm : RMWIByte<and, bdaddr12pair, NI>;
1282: defm : RMWIByte<and, bdaddr20pair, NIY>;
1283: 
1284: //===----------------------------------------------------------------------===//
1285: // OR
1286: //===----------------------------------------------------------------------===//
1287: 
1288: let Defs = [CC] in {
1289:   // ORs of a register.
1290:   let isCommutable = 1, CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1291:     defm OR : BinaryRRAndK<"or", 0x16, 0xB9F6, or, GR32, GR32>;
1292:     defm OGR : BinaryRREAndK<"ogr", 0xB981, 0xB9E6, or, GR64, GR64>;
1293:   }
1294: 
1295:   // ORs of a 16-bit immediate, leaving other bits unaffected.
1296:   // The CC result only reflects the 16-bit field, not the full register.
1297:   //
1298:   // OIxMux expands to OI[LH]x, depending on the choice of register.
1299:   def OILMux : BinaryRIPseudo<or, GRX32, imm32ll16>,
1300:                Requires<[FeatureHighWord]>;
1301:   def OIHMux : BinaryRIPseudo<or, GRX32, imm32lh16>,
1302:                Requires<[FeatureHighWord]>;
1303:   def OILL : BinaryRI<"oill", 0xA5B, or, GR32, imm32ll16>;
1304:   def OILH : BinaryRI<"oilh", 0xA5A, or, GR32, imm32lh16>;
1305:   def OIHL : BinaryRI<"oihl", 0xA59, or, GRH32, imm32ll16>;
1306:   def OIHH : BinaryRI<"oihh", 0xA58, or, GRH32, imm32lh16>;
1307:   def OILL64 : BinaryAliasRI<or, GR64, imm64ll16>;
1308:   def OILH64 : BinaryAliasRI<or, GR64, imm64lh16>;
1309:   def OIHL64 : BinaryAliasRI<or, GR64, imm64hl16>;
1310:   def OIHH64 : BinaryAliasRI<or, GR64, imm64hh16>;
1311: 
1312:   // ORs of a 32-bit immediate, leaving other bits unaffected.
1313:   // The CC result only reflects the 32-bit field, which means we can
1314:   // use it as a zero indicator for i32 operations but not otherwise.
1315:   let CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1316:     // Expands to OILF or OIHF, depending on the choice of register.
1317:     def OIFMux : BinaryRIPseudo<or, GRX32, uimm32>,
1318:                  Requires<[FeatureHighWord]>;
1319:     def OILF : BinaryRIL<"oilf", 0xC0D, or, GR32, uimm32>;
1320:     def OIHF : BinaryRIL<"oihf", 0xC0C, or, GRH32, uimm32>;
```
- **EN**: This block declares or refines TableGen records such as `OR`, `OGR`, `OILMux`, `OIHMux`, `OILL`, `OILH`.
- **CN**: 该代码块声明或细化了 `OR`, `OGR`, `OILMux`, `OIHMux`, `OILL`, `OILH` 等 TableGen 记录。

### Lines 1321-1360 / 第 1321-1360 行
```tablegen
1321:   }
1322:   def OILF64 : BinaryAliasRIL<or, GR64, imm64lf32>;
1323:   def OIHF64 : BinaryAliasRIL<or, GR64, imm64hf32>;
1324: 
1325:   // ORs of memory.
1326:   let CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1327:     defm O  : BinaryRXPairAndPseudo<"o", 0x56, 0xE356, or, GR32, z_load, 4>;
1328:     defm OG : BinaryRXYAndPseudo<"og", 0xE381, or, GR64, z_load, 8>;
1329:   }
1330: 
1331:   // OR to memory
1332:   defm OI : BinarySIPair<"oi", 0x96, 0xEB56, null_frag, imm32zx8>;
1333: 
1334:   // Block OR.
1335:   let mayLoad = 1, mayStore = 1 in
1336:     defm OC : MemorySS<"oc", 0xD6, z_oc>;
1337: }
1338: defm : RMWIByte<or, bdaddr12pair, OI>;
1339: defm : RMWIByte<or, bdaddr20pair, OIY>;
1340: 
1341: //===----------------------------------------------------------------------===//
1342: // XOR
1343: //===----------------------------------------------------------------------===//
1344: 
1345: let Defs = [CC] in {
1346:   // XORs of a register.
1347:   let isCommutable = 1, CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1348:     defm XR : BinaryRRAndK<"xr", 0x17, 0xB9F7, xor, GR32, GR32>;
1349:     defm XGR : BinaryRREAndK<"xgr", 0xB982, 0xB9E7, xor, GR64, GR64>;
1350:   }
1351: 
1352:   // XORs of a 32-bit immediate, leaving other bits unaffected.
1353:   // The CC result only reflects the 32-bit field, which means we can
1354:   // use it as a zero indicator for i32 operations but not otherwise.
1355:   let CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1356:     // Expands to XILF or XIHF, depending on the choice of register.
1357:     def XIFMux : BinaryRIPseudo<xor, GRX32, uimm32>,
1358:                  Requires<[FeatureHighWord]>;
1359:     def XILF : BinaryRIL<"xilf", 0xC07, xor, GR32, uimm32>;
1360:     def XIHF : BinaryRIL<"xihf", 0xC06, xor, GRH32, uimm32>;
```
- **EN**: This block declares or refines TableGen records such as `OILF64`, `OIHF64`, `O`, `OG`, `OI`, `OC`.
- **CN**: 该代码块声明或细化了 `OILF64`, `OIHF64`, `O`, `OG`, `OI`, `OC` 等 TableGen 记录。

### Lines 1361-1400 / 第 1361-1400 行
```tablegen
1361:   }
1362:   def XILF64 : BinaryAliasRIL<xor, GR64, imm64lf32>;
1363:   def XIHF64 : BinaryAliasRIL<xor, GR64, imm64hf32>;
1364: 
1365:   // XORs of memory.
1366:   let CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1367:     defm X  : BinaryRXPairAndPseudo<"x",0x57, 0xE357, xor, GR32, z_load, 4>;
1368:     defm XG : BinaryRXYAndPseudo<"xg", 0xE382, xor, GR64, z_load, 8>;
1369:   }
1370: 
1371:   // XOR to memory
1372:   defm XI : BinarySIPair<"xi", 0x97, 0xEB57, null_frag, imm32zx8>;
1373: 
1374:   // Block XOR.
1375:   let mayLoad = 1, mayStore = 1 in
1376:     defm XC : MemorySS<"xc", 0xD7, z_xc>;
1377: }
1378: defm : RMWIByte<xor, bdaddr12pair, XI>;
1379: defm : RMWIByte<xor, bdaddr20pair, XIY>;
1380: 
1381: //===----------------------------------------------------------------------===//
1382: // Combined logical operations
1383: //===----------------------------------------------------------------------===//
1384: 
1385: let Predicates = [FeatureMiscellaneousExtensions3],
1386:     Defs = [CC] in {
1387:   // AND with complement.
1388:   let CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1389:     def NCRK : BinaryRRFa<"ncrk", 0xB9F5, andc, GR32, GR32, GR32>;
1390:     def NCGRK : BinaryRRFa<"ncgrk", 0xB9E5, andc, GR64, GR64, GR64>;
1391:   }
1392: 
1393:   // OR with complement.
1394:   let CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1395:     def OCRK : BinaryRRFa<"ocrk", 0xB975, orc, GR32, GR32, GR32>;
1396:     def OCGRK : BinaryRRFa<"ocgrk", 0xB965, orc, GR64, GR64, GR64>;
1397:   }
1398: 
1399:   // NAND.
1400:   let isCommutable = 1, CCValues = 0xC, CompareZeroCCMask = 0x8 in {
```
- **EN**: This block declares or refines TableGen records such as `XILF64`, `XIHF64`, `X`, `XG`, `XI`, `XC`.
- **CN**: 该代码块声明或细化了 `XILF64`, `XIHF64`, `X`, `XG`, `XI`, `XC` 等 TableGen 记录。

### Lines 1401-1440 / 第 1401-1440 行
```tablegen
1401:     def NNRK : BinaryRRFa<"nnrk", 0xB974, nand, GR32, GR32, GR32>;
1402:     def NNGRK : BinaryRRFa<"nngrk", 0xB964, nand, GR64, GR64, GR64>;
1403:   }
1404: 
1405:   // NOR.
1406:   let isCommutable = 1, CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1407:     def NORK : BinaryRRFa<"nork", 0xB976, nor, GR32, GR32, GR32>;
1408:     def NOGRK : BinaryRRFa<"nogrk", 0xB966, nor, GR64, GR64, GR64>;
1409:     let isAsmParserOnly = 1 in {
1410:       def NOTR : UnaryRRFa<"notr", 0xB976, nor, GR32, GR32>;
1411:       def NOTGR : UnaryRRFa<"notgr", 0xB966, nor, GR64, GR64>;
1412:     }
1413:   }
1414: 
1415:   // NXOR.
1416:   let isCommutable = 1, CCValues = 0xC, CompareZeroCCMask = 0x8 in {
1417:     def NXRK : BinaryRRFa<"nxrk", 0xB977, nxor, GR32, GR32, GR32>;
1418:     def NXGRK : BinaryRRFa<"nxgrk", 0xB967, nxor, GR64, GR64, GR64>;
1419:   }
1420: }
1421: 
1422: //===----------------------------------------------------------------------===//
1423: // Multiplication
1424: //===----------------------------------------------------------------------===//
1425: 
1426: // Multiplication of a register, setting the condition code.  We prefer these
1427: // over MS(G)R if available, even though we cannot use the condition code,
1428: // since they are three-operand instructions.
1429: let Predicates = [FeatureMiscellaneousExtensions2],
1430:     Defs = [CC], isCommutable = 1 in {
1431:   def MSRKC  : BinaryRRFa<"msrkc",  0xB9FD, mul, GR32, GR32, GR32>;
1432:   def MSGRKC : BinaryRRFa<"msgrkc", 0xB9ED, mul, GR64, GR64, GR64>;
1433: }
1434: 
1435: // Multiplication of a register.
1436: let isCommutable = 1 in {
1437:   def MSR  : BinaryRRE<"msr",  0xB252, mul, GR32, GR32>;
1438:   def MSGR : BinaryRRE<"msgr", 0xB90C, mul, GR64, GR64>;
1439: }
1440: def MSGFR : BinaryRRE<"msgfr", 0xB91C, null_frag, GR64, GR32>;
```
- **EN**: This block declares or refines TableGen records such as `NNRK`, `NNGRK`, `NORK`, `NOGRK`, `NOTR`, `NOTGR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `NNRK`, `NNGRK`, `NORK`, `NOGRK`, `NOTR`, `NOTGR` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1441-1480 / 第 1441-1480 行
```tablegen
1441: defm : SXB<mul, GR64, MSGFR>;
1442: 
1443: // Multiplication of a signed 16-bit immediate.
1444: def MHI  : BinaryRI<"mhi",  0xA7C, mul, GR32, imm32sx16>;
1445: def MGHI : BinaryRI<"mghi", 0xA7D, mul, GR64, imm64sx16>;
1446: 
1447: // Multiplication of a signed 32-bit immediate.
1448: def MSFI  : BinaryRIL<"msfi",  0xC21, mul, GR32, simm32>;
1449: def MSGFI : BinaryRIL<"msgfi", 0xC20, mul, GR64, imm64sx32>;
1450: 
1451: // Multiplication of memory.
1452: defm MH   : BinaryRXPair<"mh", 0x4C, 0xE37C, mul, GR32, z_asextloadi16, 2>;
1453: defm MS   : BinaryRXPair<"ms", 0x71, 0xE351, mul, GR32, z_load, 4>;
1454: def  MGH  : BinaryRXY<"mgh", 0xE33C, mul, GR64, z_asextloadi16, 2>,
1455:             Requires<[FeatureMiscellaneousExtensions2]>;
1456: def  MSGF : BinaryRXY<"msgf", 0xE31C, mul, GR64, z_asextloadi32, 4>;
1457: def  MSG  : BinaryRXY<"msg",  0xE30C, mul, GR64, z_load, 8>;
1458: 
1459: // Multiplication of memory, setting the condition code.
1460: let Predicates = [FeatureMiscellaneousExtensions2], Defs = [CC] in {
1461:   defm MSC  : BinaryRXYAndPseudo<"msc",  0xE353, null_frag, GR32, z_load, 4>;
1462:   defm MSGC : BinaryRXYAndPseudo<"msgc", 0xE383, null_frag, GR64, z_load, 8>;
1463: }
1464: 
1465: // Multiplication of a register, producing two results.
1466: def MR   : BinaryRR <"mr",    0x1C,   null_frag, GR128, GR32>;
1467: def MGRK : BinaryRRFa<"mgrk", 0xB9EC, null_frag, GR128, GR64, GR64>,
1468:            Requires<[FeatureMiscellaneousExtensions2]>;
1469: def MLR  : BinaryRRE<"mlr",  0xB996, null_frag, GR128, GR32>;
1470: def MLGR : BinaryRRE<"mlgr", 0xB986, null_frag, GR128, GR64>;
1471: 
1472: def : Pat<(z_smul_lohi GR64:$src1, GR64:$src2),
1473:           (MGRK GR64:$src1, GR64:$src2)>;
1474: def : Pat<(z_umul_lohi GR64:$src1, GR64:$src2),
1475:           (MLGR (AEXT128 GR64:$src1), GR64:$src2)>;
1476: 
1477: // Multiplication of memory, producing two results.
1478: def M   : BinaryRX <"m",   0x5C,   null_frag, GR128, z_load, 4>;
1479: def MFY : BinaryRXY<"mfy", 0xE35C, null_frag, GR128, z_load, 4>;
1480: def MG  : BinaryRXY<"mg",  0xE384, null_frag, GR128, z_load, 8>,
```
- **EN**: This block declares or refines TableGen records such as `MHI`, `MGHI`, `MSFI`, `MSGFI`, `MH`, `MS`.
- **CN**: 该代码块声明或细化了 `MHI`, `MGHI`, `MSFI`, `MSGFI`, `MH`, `MS` 等 TableGen 记录。

### Lines 1481-1520 / 第 1481-1520 行
```tablegen
1481:           Requires<[FeatureMiscellaneousExtensions2]>;
1482: def ML  : BinaryRXY<"ml",  0xE396, null_frag, GR128, z_load, 4>;
1483: def MLG : BinaryRXY<"mlg", 0xE386, null_frag, GR128, z_load, 8>;
1484: 
1485: def : Pat<(z_smul_lohi GR64:$src1, (i64 (z_load bdxaddr20only:$src2))),
1486:           (MG (AEXT128 GR64:$src1), bdxaddr20only:$src2)>;
1487: def : Pat<(z_umul_lohi GR64:$src1, (i64 (z_load bdxaddr20only:$src2))),
1488:           (MLG (AEXT128 GR64:$src1), bdxaddr20only:$src2)>;
1489: 
1490: //===----------------------------------------------------------------------===//
1491: // Division and remainder
1492: //===----------------------------------------------------------------------===//
1493: 
1494: let hasSideEffects = 1 in {  // Do not speculatively execute.
1495:   // Division and remainder, from registers.
1496:   def DR    : BinaryRR <"dr",    0x1D,   null_frag, GR128, GR32>;
1497:   def DSGFR : BinaryRRE<"dsgfr", 0xB91D, null_frag, GR128, GR32>;
1498:   def DSGR  : BinaryRRE<"dsgr",  0xB90D, null_frag, GR128, GR64>;
1499:   def DLR   : BinaryRRE<"dlr",   0xB997, null_frag, GR128, GR32>;
1500:   def DLGR  : BinaryRRE<"dlgr",  0xB987, null_frag, GR128, GR64>;
1501: 
1502:   // Division and remainder, from memory.
1503:   def D    : BinaryRX <"d",    0x5D,   null_frag, GR128, z_load, 4>;
1504:   def DSGF : BinaryRXY<"dsgf", 0xE31D, null_frag, GR128, z_load, 4>;
1505:   def DSG  : BinaryRXY<"dsg",  0xE30D, null_frag, GR128, z_load, 8>;
1506:   def DL   : BinaryRXY<"dl",   0xE397, null_frag, GR128, z_load, 4>;
1507:   def DLG  : BinaryRXY<"dlg",  0xE387, null_frag, GR128, z_load, 8>;
1508: }
1509: def : Pat<(z_sdivrem GR64:$src1, GR32:$src2),
1510:           (DSGFR (AEXT128 GR64:$src1), GR32:$src2)>;
1511: def : Pat<(z_sdivrem GR64:$src1, (i32 (z_load bdxaddr20only:$src2))),
1512:           (DSGF (AEXT128 GR64:$src1), bdxaddr20only:$src2)>;
1513: def : Pat<(z_sdivrem GR64:$src1, GR64:$src2),
1514:           (DSGR (AEXT128 GR64:$src1), GR64:$src2)>;
1515: def : Pat<(z_sdivrem GR64:$src1, (i64 (z_load bdxaddr20only:$src2))),
1516:           (DSG (AEXT128 GR64:$src1), bdxaddr20only:$src2)>;
1517: 
1518: def : Pat<(z_udivrem GR32:$src1, GR32:$src2),
1519:           (DLR (ZEXT128 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GR32:$src1,
1520:                                        subreg_l32)), GR32:$src2)>;
```
- **EN**: This block declares or refines TableGen records such as `ML`, `MLG`, `DR`, `DSGFR`, `DSGR`, `DLR`.
- **CN**: 该代码块声明或细化了 `ML`, `MLG`, `DR`, `DSGFR`, `DSGR`, `DLR` 等 TableGen 记录。

### Lines 1521-1560 / 第 1521-1560 行
```tablegen
1521: def : Pat<(z_udivrem GR32:$src1, (i32 (z_load bdxaddr20only:$src2))),
1522:           (DL (ZEXT128 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GR32:$src1,
1523:                                       subreg_l32)), bdxaddr20only:$src2)>;
1524: def : Pat<(z_udivrem GR64:$src1, GR64:$src2),
1525:           (DLGR (ZEXT128 GR64:$src1), GR64:$src2)>;
1526: def : Pat<(z_udivrem GR64:$src1, (i64 (z_load bdxaddr20only:$src2))),
1527:           (DLG (ZEXT128 GR64:$src1), bdxaddr20only:$src2)>;
1528: 
1529: //===----------------------------------------------------------------------===//
1530: // Shifts
1531: //===----------------------------------------------------------------------===//
1532: 
1533: // Logical shift left.
1534: defm SLL : BinaryRSAndK<"sll", 0x89, 0xEBDF, shiftop<shl>, GR32>;
1535: def SLLG : BinaryRSY<"sllg", 0xEB0D, shiftop<shl>, GR64>;
1536: def SLDL : BinaryRS<"sldl", 0x8D, null_frag, GR128>;
1537: 
1538: // Arithmetic shift left.
1539: let Defs = [CC] in {
1540:   defm SLA : BinaryRSAndK<"sla", 0x8B, 0xEBDD, null_frag, GR32>;
1541:   def SLAG : BinaryRSY<"slag", 0xEB0B, null_frag, GR64>;
1542:   def SLDA : BinaryRS<"slda", 0x8F, null_frag, GR128>;
1543: }
1544: 
1545: // Logical shift right.
1546: defm SRL : BinaryRSAndK<"srl", 0x88, 0xEBDE, shiftop<srl>, GR32>;
1547: def SRLG : BinaryRSY<"srlg", 0xEB0C, shiftop<srl>, GR64>;
1548: def SRDL : BinaryRS<"srdl", 0x8C, null_frag, GR128>;
1549: 
1550: // Arithmetic shift right.
1551: let Defs = [CC], CCValues = 0xE, CompareZeroCCMask = 0xE in {
1552:   defm SRA : BinaryRSAndK<"sra", 0x8A, 0xEBDC, shiftop<sra>, GR32>;
1553:   def SRAG : BinaryRSY<"srag", 0xEB0A, shiftop<sra>, GR64>;
1554:   def SRDA : BinaryRS<"srda", 0x8E, null_frag, GR128>;
1555: }
1556: 
1557: // Rotate left.
1558: def RLL  : BinaryRSY<"rll",  0xEB1D, shiftop<rotl>, GR32>;
1559: def RLLG : BinaryRSY<"rllg", 0xEB1C, shiftop<rotl>, GR64>;
1560: 
```
- **EN**: This block declares or refines TableGen records such as `SLL`, `SLLG`, `SLDL`, `SLA`, `SLAG`, `SLDA`.
- **CN**: 该代码块声明或细化了 `SLL`, `SLLG`, `SLDL`, `SLA`, `SLAG`, `SLDA` 等 TableGen 记录。

### Lines 1561-1600 / 第 1561-1600 行
```tablegen
1561: // Rotate second operand left and inserted selected bits into first operand.
1562: // These can act like 32-bit operands provided that the constant start and
1563: // end bits (operands 2 and 3) are in the range [32, 64).
1564: let Defs = [CC] in {
1565:   let isCodeGenOnly = 1 in
1566:     defm RISBG32 : RotateSelectRIEf<"risbg", 0xEC55, GR32, GR32>;
1567:   let CCValues = 0xE, CompareZeroCCMask = 0xE in {
1568:     defm RISBG : RotateSelectRIEf<"risbg", 0xEC55, GR64, GR64>;
1569:     defm RISBGZ : RotateSelectRIEf<"risbgz", 0xEC55, GR64, GR64, 0, 128>;
1570:   }
1571: }
1572: 
1573: // On zEC12 we have a variant of RISBG that does not set CC.
1574: let Predicates = [FeatureMiscellaneousExtensions] in {
1575:   defm RISBGN : RotateSelectRIEf<"risbgn", 0xEC59, GR64, GR64>;
1576:   defm RISBGNZ : RotateSelectRIEf<"risbgnz", 0xEC59, GR64, GR64, 0, 128>;
1577: }
1578: 
1579: // Forms of RISBG that only affect one word of the destination register.
1580: // They do not set CC.
1581: let Predicates = [FeatureHighWord] in {
1582:   def  RISBMux : RotateSelectRIEfPseudo<GRX32, GRX32>;
1583:   defm RISBLL  : RotateSelectAliasRIEf<GR32,  GR32>;
1584:   defm RISBLH  : RotateSelectAliasRIEf<GR32,  GRH32>;
1585:   defm RISBHL  : RotateSelectAliasRIEf<GRH32, GR32>;
1586:   defm RISBHH  : RotateSelectAliasRIEf<GRH32, GRH32>;
1587:   defm RISBLG  : RotateSelectRIEf<"risblg", 0xEC51, GR32, GR64>;
1588:   defm RISBHG  : RotateSelectRIEf<"risbhg", 0xEC5D, GRH32, GR64>;
1589: }
1590: 
1591: // Rotate second operand left and perform a logical operation with selected
1592: // bits of the first operand.  The CC result only describes the selected bits,
1593: // so isn't useful for a full comparison against zero.
1594: let Defs = [CC] in {
1595:   defm RNSBG : RotateSelectRIEf<"rnsbg", 0xEC54, GR64, GR64>;
1596:   defm ROSBG : RotateSelectRIEf<"rosbg", 0xEC56, GR64, GR64>;
1597:   defm RXSBG : RotateSelectRIEf<"rxsbg", 0xEC57, GR64, GR64>;
1598: }
1599: 
1600: //===----------------------------------------------------------------------===//
```
- **EN**: This block declares or refines TableGen records such as `RISBG32`, `RISBG`, `RISBGZ`, `RISBGN`, `RISBGNZ`, `RISBMux`.
- **CN**: 该代码块声明或细化了 `RISBG32`, `RISBG`, `RISBGZ`, `RISBGN`, `RISBGNZ`, `RISBMux` 等 TableGen 记录。

### Lines 1601-1640 / 第 1601-1640 行
```tablegen
1601: // Comparison
1602: //===----------------------------------------------------------------------===//
1603: 
1604: // Signed comparisons.  We put these before the unsigned comparisons because
1605: // some of the signed forms have COMPARE AND BRANCH equivalents whereas none
1606: // of the unsigned forms do.
1607: let Defs = [CC], CCValues = 0xE in {
1608:   // Comparison with a register.
1609:   def CR   : CompareRR <"cr",   0x19,   z_scmp,    GR32, GR32>;
1610:   def CGFR : CompareRRE<"cgfr", 0xB930, null_frag, GR64, GR32>;
1611:   def CGR  : CompareRRE<"cgr",  0xB920, z_scmp,    GR64, GR64>;
1612: 
1613:   // Comparison with a high register.
1614:   def CHHR : CompareRRE<"chhr", 0xB9CD, null_frag, GRH32, GRH32>,
1615:              Requires<[FeatureHighWord]>;
1616:   def CHLR : CompareRRE<"chlr", 0xB9DD, null_frag, GRH32, GR32>,
1617:              Requires<[FeatureHighWord]>;
1618: 
1619:   // Comparison with a signed 16-bit immediate.  CHIMux expands to CHI or CIH,
1620:   // depending on the choice of register.
1621:   def CHIMux : CompareRIPseudo<z_scmp, GRX32, imm32sx16>,
1622:                Requires<[FeatureHighWord]>;
1623:   def CHI  : CompareRI<"chi",  0xA7E, z_scmp, GR32, imm32sx16>;
1624:   def CGHI : CompareRI<"cghi", 0xA7F, z_scmp, GR64, imm64sx16>;
1625: 
1626:   // Comparison with a signed 32-bit immediate.  CFIMux expands to CFI or CIH,
1627:   // depending on the choice of register.
1628:   def CFIMux : CompareRIPseudo<z_scmp, GRX32, simm32>,
1629:                Requires<[FeatureHighWord]>;
1630:   def CFI  : CompareRIL<"cfi",  0xC2D, z_scmp, GR32, simm32>;
1631:   def CIH  : CompareRIL<"cih",  0xCCD, z_scmp, GRH32, simm32>,
1632:              Requires<[FeatureHighWord]>;
1633:   def CGFI : CompareRIL<"cgfi", 0xC2C, z_scmp, GR64, imm64sx32>;
1634: 
1635:   // Comparison with memory.
1636:   defm CH    : CompareRXPair<"ch", 0x49, 0xE379, z_scmp, GR32, z_asextloadi16, 2>;
1637:   def  CMux  : CompareRXYPseudo<z_scmp, GRX32, z_load, 4>,
1638:                Requires<[FeatureHighWord]>;
1639:   defm C     : CompareRXPair<"c",  0x59, 0xE359, z_scmp, GR32, z_load, 4>;
1640:   def  CHF   : CompareRXY<"chf", 0xE3CD, z_scmp, GRH32, z_load, 4>,
```
- **EN**: This block declares or refines TableGen records such as `CR`, `CGFR`, `CGR`, `CHHR`, `CHLR`, `CHIMux`.
- **CN**: 该代码块声明或细化了 `CR`, `CGFR`, `CGR`, `CHHR`, `CHLR`, `CHIMux` 等 TableGen 记录。

### Lines 1641-1680 / 第 1641-1680 行
```tablegen
1641:                Requires<[FeatureHighWord]>;
1642:   def  CGH   : CompareRXY<"cgh", 0xE334, z_scmp, GR64, z_asextloadi16, 2>;
1643:   def  CGF   : CompareRXY<"cgf", 0xE330, z_scmp, GR64, z_asextloadi32, 4>;
1644:   def  CG    : CompareRXY<"cg",  0xE320, z_scmp, GR64, z_load, 8>;
1645:   def  CHRL  : CompareRILPC<"chrl",  0xC65, z_scmp, GR32, aligned_z_asextloadi16>;
1646:   def  CRL   : CompareRILPC<"crl",   0xC6D, z_scmp, GR32, aligned_z_load>;
1647:   def  CGHRL : CompareRILPC<"cghrl", 0xC64, z_scmp, GR64, aligned_z_asextloadi16>;
1648:   def  CGFRL : CompareRILPC<"cgfrl", 0xC6C, z_scmp, GR64, aligned_z_asextloadi32>;
1649:   def  CGRL  : CompareRILPC<"cgrl",  0xC68, z_scmp, GR64, aligned_z_load>;
1650: 
1651:   // Comparison between memory and a signed 16-bit immediate.
1652:   def CHHSI : CompareSIL<"chhsi", 0xE554, z_scmp, z_asextloadi16, imm32sx16>;
1653:   def CHSI  : CompareSIL<"chsi",  0xE55C, z_scmp, z_load, imm32sx16>;
1654:   def CGHSI : CompareSIL<"cghsi", 0xE558, z_scmp, z_load, imm64sx16>;
1655: }
1656: defm : SXB<z_scmp, GR64, CGFR>;
1657: 
1658: // Unsigned comparisons.
1659: let Defs = [CC], CCValues = 0xE, IsLogical = 1 in {
1660:   // Comparison with a register.
1661:   def CLR   : CompareRR <"clr",   0x15,   z_ucmp,    GR32, GR32>;
1662:   def CLGFR : CompareRRE<"clgfr", 0xB931, null_frag, GR64, GR32>;
1663:   def CLGR  : CompareRRE<"clgr",  0xB921, z_ucmp,    GR64, GR64>;
1664: 
1665:   // Comparison with a high register.
1666:   def CLHHR : CompareRRE<"clhhr", 0xB9CF, null_frag, GRH32, GRH32>,
1667:               Requires<[FeatureHighWord]>;
1668:   def CLHLR : CompareRRE<"clhlr", 0xB9DF, null_frag, GRH32, GR32>,
1669:               Requires<[FeatureHighWord]>;
1670: 
1671:   // Comparison with an unsigned 32-bit immediate.  CLFIMux expands to CLFI
1672:   // or CLIH, depending on the choice of register.
1673:   def CLFIMux : CompareRIPseudo<z_ucmp, GRX32, uimm32>,
1674:                 Requires<[FeatureHighWord]>;
1675:   def CLFI  : CompareRIL<"clfi",  0xC2F, z_ucmp, GR32, uimm32>;
1676:   def CLIH  : CompareRIL<"clih",  0xCCF, z_ucmp, GRH32, uimm32>,
1677:               Requires<[FeatureHighWord]>;
1678:   def CLGFI : CompareRIL<"clgfi", 0xC2E, z_ucmp, GR64, imm64zx32>;
1679: 
1680:   // Comparison with memory.
```
- **EN**: This block declares or refines TableGen records such as `CGH`, `CGF`, `CG`, `CHRL`, `CRL`, `CGHRL`.
- **CN**: 该代码块声明或细化了 `CGH`, `CGF`, `CG`, `CHRL`, `CRL`, `CGHRL` 等 TableGen 记录。

### Lines 1681-1720 / 第 1681-1720 行
```tablegen
1681:   def  CLMux  : CompareRXYPseudo<z_ucmp, GRX32, z_load, 4>,
1682:                 Requires<[FeatureHighWord]>;
1683:   defm CL     : CompareRXPair<"cl", 0x55, 0xE355, z_ucmp, GR32, z_load, 4>;
1684:   def  CLHF   : CompareRXY<"clhf", 0xE3CF, z_ucmp, GRH32, z_load, 4>,
1685:                 Requires<[FeatureHighWord]>;
1686:   def  CLGF   : CompareRXY<"clgf", 0xE331, z_ucmp, GR64, z_azextloadi32, 4>;
1687:   def  CLG    : CompareRXY<"clg",  0xE321, z_ucmp, GR64, z_load, 8>;
1688:   def  CLHRL  : CompareRILPC<"clhrl",  0xC67, z_ucmp, GR32,
1689:                              aligned_z_azextloadi16>;
1690:   def  CLRL   : CompareRILPC<"clrl",   0xC6F, z_ucmp, GR32,
1691:                              aligned_z_load>;
1692:   def  CLGHRL : CompareRILPC<"clghrl", 0xC66, z_ucmp, GR64,
1693:                              aligned_z_azextloadi16>;
1694:   def  CLGFRL : CompareRILPC<"clgfrl", 0xC6E, z_ucmp, GR64,
1695:                              aligned_z_azextloadi32>;
1696:   def  CLGRL  : CompareRILPC<"clgrl",  0xC6A, z_ucmp, GR64,
1697:                              aligned_z_load>;
1698: 
1699:   // Comparison between memory and an unsigned 8-bit immediate.
1700:   defm CLI : CompareSIPair<"cli", 0x95, 0xEB55, z_ucmp, z_azextloadi8, imm32zx8>;
1701: 
1702:   // Comparison between memory and an unsigned 16-bit immediate.
1703:   def CLHHSI : CompareSIL<"clhhsi", 0xE555, z_ucmp, z_azextloadi16, imm32zx16>;
1704:   def CLFHSI : CompareSIL<"clfhsi", 0xE55D, z_ucmp, z_load, imm32zx16>;
1705:   def CLGHSI : CompareSIL<"clghsi", 0xE559, z_ucmp, z_load, imm64zx16>;
1706: }
1707: defm : ZXB<z_ucmp, GR64, CLGFR>;
1708: 
1709: // Memory-to-memory comparison.
1710: let mayLoad = 1, Defs = [CC] in {
1711:   defm CLC : CompareMemorySS<"clc", 0xD5, z_clc>;
1712:   def CLCL  : SideEffectBinaryMemMemRR<"clcl", 0x0F, GR128, GR128>;
1713:   def CLCLE : SideEffectTernaryMemMemRS<"clcle", 0xA9, GR128, GR128>;
1714:   def CLCLU : SideEffectTernaryMemMemRSY<"clclu", 0xEB8F, GR128, GR128>;
1715: }
1716: 
1717: // String comparison.
1718: let mayLoad = 1, Defs = [CC] in
1719:   defm CLST : StringRRE<"clst", 0xB25D, z_strcmp>;
1720: 
```
- **EN**: This block declares or refines TableGen records such as `CLMux`, `CL`, `CLHF`, `CLGF`, `CLG`, `CLHRL`.
- **CN**: 该代码块声明或细化了 `CLMux`, `CL`, `CLHF`, `CLGF`, `CLG`, `CLHRL` 等 TableGen 记录。

### Lines 1721-1760 / 第 1721-1760 行
```tablegen
1721: // Test under mask.
1722: let Defs = [CC] in {
1723:   // TMxMux expands to TM[LH]x, depending on the choice of register.
1724:   def TMLMux : CompareRIPseudo<z_tm_reg, GRX32, imm32ll16>,
1725:                Requires<[FeatureHighWord]>;
1726:   def TMHMux : CompareRIPseudo<z_tm_reg, GRX32, imm32lh16>,
1727:                Requires<[FeatureHighWord]>;
1728:   def TMLL : CompareRI<"tmll", 0xA71, z_tm_reg, GR32, imm32ll16>;
1729:   def TMLH : CompareRI<"tmlh", 0xA70, z_tm_reg, GR32, imm32lh16>;
1730:   def TMHL : CompareRI<"tmhl", 0xA73, z_tm_reg, GRH32, imm32ll16>;
1731:   def TMHH : CompareRI<"tmhh", 0xA72, z_tm_reg, GRH32, imm32lh16>;
1732: 
1733:   def TMLL64 : CompareAliasRI<z_tm_reg, GR64, imm64ll16>;
1734:   def TMLH64 : CompareAliasRI<z_tm_reg, GR64, imm64lh16>;
1735:   def TMHL64 : CompareAliasRI<z_tm_reg, GR64, imm64hl16>;
1736:   def TMHH64 : CompareAliasRI<z_tm_reg, GR64, imm64hh16>;
1737: 
1738:   defm TM : CompareSIPair<"tm", 0x91, 0xEB51, z_tm_mem, z_anyextloadi8, imm32zx8>;
1739: }
1740: 
1741: def TML : InstAlias<"tml\t$R, $I", (TMLL GR32:$R, imm32ll16:$I), 0>;
1742: def TMH : InstAlias<"tmh\t$R, $I", (TMLH GR32:$R, imm32lh16:$I), 0>;
1743: 
1744: // Compare logical characters under mask -- not (yet) used for codegen.
1745: let Defs = [CC] in {
1746:   defm CLM : CompareRSPair<"clm", 0xBD, 0xEB21, GR32, 0>;
1747:   def CLMH : CompareRSY<"clmh", 0xEB20, GRH32, 0>;
1748: }
1749: 
1750: //===----------------------------------------------------------------------===//
1751: // Prefetch and execution hint
1752: //===----------------------------------------------------------------------===//
1753: 
1754: let mayLoad = 1, mayStore = 1 in {
1755:   def PFD : PrefetchRXY<"pfd", 0xE336, z_prefetch>;
1756:   def PFDRL : PrefetchRILPC<"pfdrl", 0xC62, z_prefetch>;
1757: }
1758: 
1759: let Predicates = [FeatureExecutionHint], hasSideEffects = 1 in {
1760:   // Branch Prediction Preload
```
- **EN**: This block declares or refines TableGen records such as `TMLMux`, `TMHMux`, `TMLL`, `TMLH`, `TMHL`, `TMHH`.
- **CN**: 该代码块声明或细化了 `TMLMux`, `TMHMux`, `TMLL`, `TMLH`, `TMHL`, `TMHH` 等 TableGen 记录。

### Lines 1761-1800 / 第 1761-1800 行
```tablegen
1761:   def BPP : BranchPreloadSMI<"bpp", 0xC7>;
1762:   def BPRP : BranchPreloadMII<"bprp", 0xC5>;
1763: 
1764:   // Next Instruction Access Intent
1765:   def NIAI : SideEffectBinaryIE<"niai", 0xB2FA, imm32zx4, imm32zx4>;
1766: }
1767: 
1768: //===----------------------------------------------------------------------===//
1769: // Atomic operations
1770: //===----------------------------------------------------------------------===//
1771: 
1772: // A serialization instruction that acts as a barrier for all memory
1773: // accesses, which expands to "bcr 14, 0".
1774: let hasSideEffects = 1 in
1775: def Serialize : Alias<2, (outs), (ins), []>;
1776: 
1777: let Predicates = [FeatureInterlockedAccess1], Defs = [CC] in {
1778:   def LAA   : LoadAndOpRSY<"laa",   0xEBF8, atomic_load_add_i32, GR32>;
1779:   def LAAG  : LoadAndOpRSY<"laag",  0xEBE8, atomic_load_add_i64, GR64>;
1780:   def LAAL  : LoadAndOpRSY<"laal",  0xEBFA, null_frag, GR32>;
1781:   def LAALG : LoadAndOpRSY<"laalg", 0xEBEA, null_frag, GR64>;
1782:   def LAN   : LoadAndOpRSY<"lan",   0xEBF4, atomic_load_and_i32, GR32>;
1783:   def LANG  : LoadAndOpRSY<"lang",  0xEBE4, atomic_load_and_i64, GR64>;
1784:   def LAO   : LoadAndOpRSY<"lao",   0xEBF6, atomic_load_or_i32, GR32>;
1785:   def LAOG  : LoadAndOpRSY<"laog",  0xEBE6, atomic_load_or_i64, GR64>;
1786:   def LAX   : LoadAndOpRSY<"lax",   0xEBF7, atomic_load_xor_i32, GR32>;
1787:   def LAXG  : LoadAndOpRSY<"laxg",  0xEBE7, atomic_load_xor_i64, GR64>;
1788: }
1789: 
1790: def ATOMIC_SWAPW   : AtomicLoadWBinaryReg<z_atomic_swapw>;
1791: 
1792: def ATOMIC_LOADW_AR  : AtomicLoadWBinaryReg<z_atomic_loadw_add>;
1793: def ATOMIC_LOADW_AFI : AtomicLoadWBinaryImm<z_atomic_loadw_add, simm32>;
1794: 
1795: def ATOMIC_LOADW_SR : AtomicLoadWBinaryReg<z_atomic_loadw_sub>;
1796: 
1797: def ATOMIC_LOADW_NR   : AtomicLoadWBinaryReg<z_atomic_loadw_and>;
1798: def ATOMIC_LOADW_NILH : AtomicLoadWBinaryImm<z_atomic_loadw_and, imm32lh16c>;
1799: 
1800: def ATOMIC_LOADW_OR     : AtomicLoadWBinaryReg<z_atomic_loadw_or>;
```
- **EN**: This block declares or refines TableGen records such as `BPP`, `BPRP`, `NIAI`, `Serialize`, `LAA`, `LAAG`.
- **CN**: 该代码块声明或细化了 `BPP`, `BPRP`, `NIAI`, `Serialize`, `LAA`, `LAAG` 等 TableGen 记录。

### Lines 1801-1840 / 第 1801-1840 行
```tablegen
1801: def ATOMIC_LOADW_OILH   : AtomicLoadWBinaryImm<z_atomic_loadw_or, imm32lh16>;
1802: 
1803: def ATOMIC_LOADW_XR     : AtomicLoadWBinaryReg<z_atomic_loadw_xor>;
1804: def ATOMIC_LOADW_XILF   : AtomicLoadWBinaryImm<z_atomic_loadw_xor, uimm32>;
1805: 
1806: def ATOMIC_LOADW_NRi    : AtomicLoadWBinaryReg<z_atomic_loadw_nand>;
1807: def ATOMIC_LOADW_NILHi  : AtomicLoadWBinaryImm<z_atomic_loadw_nand,
1808:                                                imm32lh16c>;
1809: 
1810: def ATOMIC_LOADW_MIN    : AtomicLoadWBinaryReg<z_atomic_loadw_min>;
1811: def ATOMIC_LOADW_MAX    : AtomicLoadWBinaryReg<z_atomic_loadw_max>;
1812: def ATOMIC_LOADW_UMIN   : AtomicLoadWBinaryReg<z_atomic_loadw_umin>;
1813: def ATOMIC_LOADW_UMAX   : AtomicLoadWBinaryReg<z_atomic_loadw_umax>;
1814: 
1815: def ATOMIC_CMP_SWAPW
1816:   : Pseudo<(outs GR32:$dst), (ins bdaddr20only:$addr, GR32:$cmp, GR32:$swap,
1817:                                   ADDR32:$bitshift, ADDR32:$negbitshift,
1818:                                   uimm32:$bitsize),
1819:            [(set GR32:$dst,
1820:                  (z_atomic_cmp_swapw bdaddr20only:$addr, GR32:$cmp, GR32:$swap,
1821:                                      ADDR32:$bitshift, ADDR32:$negbitshift,
1822:                                      uimm32:$bitsize))]> {
1823:   let Defs = [CC];
1824:   let mayLoad = 1;
1825:   let mayStore = 1;
1826:   let usesCustomInserter = 1;
1827:   let hasNoSchedulingInfo = 1;
1828: }
1829: 
1830: // Test and set.
1831: let mayLoad = 1, Defs = [CC] in
1832:   def TS : StoreInherentS<"ts", 0x9300, null_frag, 1>;
1833: 
1834: // Compare and swap.
1835: let Defs = [CC] in {
1836:   defm CS  : CmpSwapRSPair<"cs", 0xBA, 0xEB14, z_atomic_cmp_swap, GR32>;
1837:   def  CSG : CmpSwapRSY<"csg", 0xEB30, z_atomic_cmp_swap, GR64>;
1838: }
1839: 
1840: // Compare double and swap.
```
- **EN**: This block declares or refines TableGen records such as `ATOMIC_LOADW_OILH`, `ATOMIC_LOADW_XR`, `ATOMIC_LOADW_XILF`, `ATOMIC_LOADW_NRi`, `ATOMIC_LOADW_NILHi`, `ATOMIC_LOADW_MIN`.
- **CN**: 该代码块声明或细化了 `ATOMIC_LOADW_OILH`, `ATOMIC_LOADW_XR`, `ATOMIC_LOADW_XILF`, `ATOMIC_LOADW_NRi`, `ATOMIC_LOADW_NILHi`, `ATOMIC_LOADW_MIN` 等 TableGen 记录。

### Lines 1841-1880 / 第 1841-1880 行
```tablegen
1841: let Defs = [CC] in {
1842:   defm CDS  : CmpSwapRSPair<"cds", 0xBB, 0xEB31, null_frag, GR128>;
1843:   def  CDSG : CmpSwapRSY<"cdsg", 0xEB3E, z_atomic_cmp_swap_128, GR128>;
1844: }
1845: 
1846: // Compare and swap and store.
1847: let Uses = [R0L, R1D], Defs = [CC], mayStore = 1, mayLoad = 1 in
1848:   def CSST : SideEffectTernarySSF<"csst", 0xC82, GR64>;
1849: 
1850: // Perform locked operation.
1851: let Uses = [R0L, R1D], Defs = [CC], mayStore = 1, mayLoad =1 in
1852:   def PLO : SideEffectQuaternarySSe<"plo", 0xEE, GR64>;
1853: 
1854: // Load/store pair from/to quadword.
1855: def LPQ  : UnaryRXY<"lpq", 0xE38F, z_atomic_load_128, GR128, 16>;
1856: def STPQ : StoreRXY<"stpq", 0xE38E, z_atomic_store_128, GR128, 16>;
1857: 
1858: // Load pair disjoint.
1859: let Predicates = [FeatureInterlockedAccess1], Defs = [CC] in {
1860:   def LPD  : BinarySSF<"lpd", 0xC84, GR128>;
1861:   def LPDG : BinarySSF<"lpdg", 0xC85, GR128>;
1862: }
1863: 
1864: // Compare and load.
1865: let Predicates = [FeatureConcurrentFunctions], Defs = [CC] in {
1866:   def CAL   : BinarySSF<"cal", 0xC86, GR32>;
1867:   def CALGF : BinarySSF<"calgf", 0xC8F, GR64>;
1868:   def CALG  : BinarySSF<"calg", 0xC87, GR64>;
1869: }
1870: 
1871: // Perform function with concurrent results.
1872: let Predicates = [FeatureConcurrentFunctions], Uses = [R0D], Defs = [CC],
1873:     mayLoad = 1, mayStore = 1, hasSideEffects = 1 in {
1874:   def PFCR : BinaryRSY<"pfcr", 0xEB16, null_frag, GR64>;
1875: }
1876: 
1877: //===----------------------------------------------------------------------===//
1878: // Translate and convert
1879: //===----------------------------------------------------------------------===//
1880: 
```
- **EN**: This block declares or refines TableGen records such as `CDS`, `CDSG`, `CSST`, `PLO`, `LPQ`, `STPQ`.
- **CN**: 该代码块声明或细化了 `CDS`, `CDSG`, `CSST`, `PLO`, `LPQ`, `STPQ` 等 TableGen 记录。

### Lines 1881-1920 / 第 1881-1920 行
```tablegen
1881: let mayLoad = 1, mayStore = 1 in
1882:   def TR : SideEffectBinarySSa<"tr", 0xDC>;
1883: 
1884: let mayLoad = 1, Defs = [CC, R0L, R1D] in {
1885:   def TRT  : SideEffectBinarySSa<"trt", 0xDD>;
1886:   def TRTR : SideEffectBinarySSa<"trtr", 0xD0>;
1887: }
1888: 
1889: let mayLoad = 1, mayStore = 1, Uses = [R0L] in
1890:   def TRE : SideEffectBinaryMemMemRRE<"tre", 0xB2A5, GR128, GR64>;
1891: 
1892: let mayLoad = 1, Uses = [R1D], Defs = [CC] in {
1893:   defm TRTE  : BinaryMemRRFcOpt<"trte",  0xB9BF, GR128, GR64>;
1894:   defm TRTRE : BinaryMemRRFcOpt<"trtre", 0xB9BD, GR128, GR64>;
1895: }
1896: 
1897: let mayLoad = 1, mayStore = 1, Uses = [R0L, R1D], Defs = [CC] in {
1898:   defm TROO : SideEffectTernaryMemMemRRFcOpt<"troo", 0xB993, GR128, GR64>;
1899:   defm TROT : SideEffectTernaryMemMemRRFcOpt<"trot", 0xB992, GR128, GR64>;
1900:   defm TRTO : SideEffectTernaryMemMemRRFcOpt<"trto", 0xB991, GR128, GR64>;
1901:   defm TRTT : SideEffectTernaryMemMemRRFcOpt<"trtt", 0xB990, GR128, GR64>;
1902: }
1903: 
1904: let mayLoad = 1, mayStore = 1, Defs = [CC] in {
1905:   defm CU12 : SideEffectTernaryMemMemRRFcOpt<"cu12", 0xB2A7, GR128, GR128>;
1906:   defm CU14 : SideEffectTernaryMemMemRRFcOpt<"cu14", 0xB9B0, GR128, GR128>;
1907:   defm CU21 : SideEffectTernaryMemMemRRFcOpt<"cu21", 0xB2A6, GR128, GR128>;
1908:   defm CU24 : SideEffectTernaryMemMemRRFcOpt<"cu24", 0xB9B1, GR128, GR128>;
1909:   def  CU41 : SideEffectBinaryMemMemRRE<"cu41", 0xB9B2, GR128, GR128>;
1910:   def  CU42 : SideEffectBinaryMemMemRRE<"cu42", 0xB9B3, GR128, GR128>;
1911: 
1912:   let isAsmParserOnly = 1 in {
1913:     defm CUUTF : SideEffectTernaryMemMemRRFcOpt<"cuutf", 0xB2A6, GR128, GR128>;
1914:     defm CUTFU : SideEffectTernaryMemMemRRFcOpt<"cutfu", 0xB2A7, GR128, GR128>;
1915:   }
1916: }
1917: 
1918: //--------------------------------------------------------------------------
1919: // Setjmp/Longjmp.
1920: //--------------------------------------------------------------------------
```
- **EN**: This block declares or refines TableGen records such as `TR`, `TRT`, `TRTR`, `TRE`, `TRTE`, `TRTRE`.
- **CN**: 该代码块声明或细化了 `TR`, `TRT`, `TRTR`, `TRE`, `TRTE`, `TRTRE` 等 TableGen 记录。

### Lines 1921-1960 / 第 1921-1960 行
```tablegen
1921: let isBarrier = 1,  hasNoSchedulingInfo = 1 in {
1922:   let hasSideEffects = 1, usesCustomInserter = 1 in {
1923:     def EH_SjLj_SetJmp : Pseudo<(outs GR32:$dst), (ins ADDR64:$R2), 
1924:                         [(set GR32:$dst, (z_eh_sjlj_setjmp ADDR64:$R2))]>;
1925:     let isTerminator = 1 in {
1926:       def EH_SjLj_LongJmp : Pseudo<(outs), (ins ADDR64:$R2), 
1927:                            [(z_eh_sjlj_longjmp  ADDR64:$R2)]>;
1928:     }
1929:   }
1930:   let isTerminator = 1, isCodeGenOnly = 1, Size = 0 in {
1931:     def EH_SjLj_Setup : Pseudo<(outs), (ins brtarget32:$dst), []>;
1932:   }
1933: }
1934: 
1935: //===----------------------------------------------------------------------===//
1936: // Message-security assist
1937: //===----------------------------------------------------------------------===//
1938: 
1939: let mayLoad = 1, mayStore = 1, Uses = [R0L, R1D], Defs = [CC] in {
1940:   def KM  : SideEffectBinaryMemMemRRE<"km",  0xB92E, GR128, GR128>;
1941:   def KMC : SideEffectBinaryMemMemRRE<"kmc", 0xB92F, GR128, GR128>;
1942: 
1943:   def KIMD : SideEffectBinaryMemRRE<"kimd", 0xB93E, GR64, GR128>;
1944:   def KLMD : SideEffectBinaryMemRRE<"klmd", 0xB93F, GR64, GR128>;
1945:   def KMAC : SideEffectBinaryMemRRE<"kmac", 0xB91E, GR64, GR128>;
1946: 
1947:   let Predicates = [FeatureMessageSecurityAssist4] in {
1948:     def KMF   : SideEffectBinaryMemMemRRE<"kmf", 0xB92A, GR128, GR128>;
1949:     def KMO   : SideEffectBinaryMemMemRRE<"kmo", 0xB92B, GR128, GR128>;
1950:     def KMCTR : SideEffectTernaryMemMemMemRRFb<"kmctr", 0xB92D,
1951:                                                GR128, GR128, GR128>;
1952:     def PCC   : SideEffectInherentRRE<"pcc", 0xB92C>;
1953:   }
1954: 
1955:   let Predicates = [FeatureMessageSecurityAssist5] in
1956:     def PPNO : SideEffectBinaryMemMemRRE<"ppno", 0xB93C, GR128, GR128>;
1957:   let Predicates = [FeatureMessageSecurityAssist7], isAsmParserOnly = 1 in
1958:     def PRNO : SideEffectBinaryMemMemRRE<"prno", 0xB93C, GR128, GR128>;
1959: 
1960:   let Predicates = [FeatureMessageSecurityAssist8] in
```
- **EN**: This block declares or refines TableGen records such as `EH_SjLj_SetJmp`, `EH_SjLj_LongJmp`, `EH_SjLj_Setup`, `KM`, `KMC`, `KIMD`.
- **CN**: 该代码块声明或细化了 `EH_SjLj_SetJmp`, `EH_SjLj_LongJmp`, `EH_SjLj_Setup`, `KM`, `KMC`, `KIMD` 等 TableGen 记录。

### Lines 1961-2000 / 第 1961-2000 行
```tablegen
1961:     def KMA : SideEffectTernaryMemMemMemRRFb<"kma", 0xB929,
1962:                                               GR128, GR128, GR128>;
1963: 
1964:   let Predicates = [FeatureMessageSecurityAssist9] in
1965:     def KDSA : SideEffectBinaryMemRRE<"kdsa", 0xB93A, GR64, GR128>;
1966: 
1967:   let Predicates = [FeatureMessageSecurityAssist12] in {
1968:     def KIMDOpt : SideEffectTernaryMemMemRRFc<"kimd", 0xB93E, GR64, GR128, imm32zx4>;
1969:     def KLMDOpt : SideEffectTernaryMemMemRRFc<"klmd", 0xB93F, GR64, GR128, imm32zx4>;
1970:   }
1971: }
1972: 
1973: //===----------------------------------------------------------------------===//
1974: // Guarded storage
1975: //===----------------------------------------------------------------------===//
1976: 
1977: // These instructions use and/or modify the guarded storage control
1978: // registers, which we do not otherwise model, so they should have
1979: // hasSideEffects.
1980: let Predicates = [FeatureGuardedStorage], hasSideEffects = 1 in {
1981:   def LGG : UnaryRXY<"lgg", 0xE34C, null_frag, GR64, 8>;
1982:   def LLGFSG : UnaryRXY<"llgfsg", 0xE348, null_frag, GR64, 4>;
1983: 
1984:   let mayLoad = 1 in
1985:     def LGSC : SideEffectBinaryRXY<"lgsc", 0xE34D, GR64>;
1986:   let mayStore = 1 in
1987:     def STGSC : SideEffectBinaryRXY<"stgsc", 0xE349, GR64>;
1988: }
1989: 
1990: //===----------------------------------------------------------------------===//
1991: // Decimal arithmetic
1992: //===----------------------------------------------------------------------===//
1993: 
1994: defm CVB  : BinaryRXPair<"cvb",0x4F, 0xE306, null_frag, GR32, z_load, 4>;
1995: def  CVBG : BinaryRXY<"cvbg", 0xE30E, null_frag, GR64, z_load, 8>;
1996: 
1997: defm CVD  : StoreRXPair<"cvd", 0x4E, 0xE326, null_frag, GR32, 4>;
1998: def  CVDG : StoreRXY<"cvdg", 0xE32E, null_frag, GR64, 8>;
1999: 
2000: let mayLoad = 1, mayStore = 1 in {
```
- **EN**: This block declares or refines TableGen records such as `KMA`, `KDSA`, `KIMDOpt`, `KLMDOpt`, `LGG`, `LLGFSG`.
- **CN**: 该代码块声明或细化了 `KMA`, `KDSA`, `KIMDOpt`, `KLMDOpt`, `LGG`, `LLGFSG` 等 TableGen 记录。

### Lines 2001-2040 / 第 2001-2040 行
```tablegen
2001:   def MVN : SideEffectBinarySSa<"mvn", 0xD1>;
2002:   def MVZ : SideEffectBinarySSa<"mvz", 0xD3>;
2003:   def MVO : SideEffectBinarySSb<"mvo", 0xF1>;
2004: 
2005:   def PACK : SideEffectBinarySSb<"pack", 0xF2>;
2006:   def PKA  : SideEffectBinarySSf<"pka", 0xE9>;
2007:   def PKU  : SideEffectBinarySSf<"pku", 0xE1>;
2008:   def UNPK : SideEffectBinarySSb<"unpk", 0xF3>;
2009:   let Defs = [CC] in {
2010:     def UNPKA : SideEffectBinarySSa<"unpka", 0xEA>;
2011:     def UNPKU : SideEffectBinarySSa<"unpku", 0xE2>;
2012:   }
2013: }
2014: 
2015: let mayLoad = 1, mayStore = 1 in {
2016:   let Defs = [CC] in {
2017:     def AP : SideEffectBinarySSb<"ap", 0xFA>;
2018:     def SP : SideEffectBinarySSb<"sp", 0xFB>;
2019:     def ZAP : SideEffectBinarySSb<"zap", 0xF8>;
2020:     def SRP : SideEffectTernarySSc<"srp", 0xF0>;
2021:   }
2022:   def MP : SideEffectBinarySSb<"mp", 0xFC>;
2023:   def DP : SideEffectBinarySSb<"dp", 0xFD>;
2024:   let Defs = [CC] in {
2025:     def ED : SideEffectBinarySSa<"ed", 0xDE>;
2026:     def EDMK : SideEffectBinarySSa<"edmk", 0xDF>;
2027:   }
2028: }
2029: 
2030: let Defs = [CC] in {
2031:   def CP : CompareSSb<"cp", 0xF9>;
2032:   def TP : TestRSL<"tp", 0xEBC0>;
2033: }
2034: 
2035: //===----------------------------------------------------------------------===//
2036: // Access registers
2037: //===----------------------------------------------------------------------===//
2038: 
2039: // Read a 32-bit access register into a GR32.  As with all GR32 operations,
2040: // the upper 32 bits of the enclosing GR64 remain unchanged, which is useful
```
- **EN**: This block declares or refines TableGen records such as `MVN`, `MVZ`, `MVO`, `PACK`, `PKA`, `PKU`.
- **CN**: 该代码块声明或细化了 `MVN`, `MVZ`, `MVO`, `PACK`, `PKA`, `PKU` 等 TableGen 记录。

### Lines 2041-2080 / 第 2041-2080 行
```tablegen
2041: // when a 64-bit address is stored in a pair of access registers.
2042: def EAR : UnaryRRE<"ear", 0xB24F, null_frag, GR32, AR32>;
2043: 
2044: // Set access register.
2045: def SAR : UnaryRRE<"sar", 0xB24E, null_frag, AR32, GR32>;
2046: 
2047: // Copy access register.
2048: def CPYA : UnaryRRE<"cpya", 0xB24D, null_frag, AR32, AR32>;
2049: 
2050: // Load address extended.
2051: defm LAE : LoadAddressRXPair<"lae", 0x51, 0xE375, null_frag>;
2052: 
2053: // Load access multiple.
2054: defm LAM : LoadMultipleRSPair<"lam", 0x9A, 0xEB9A, AR32>;
2055: 
2056: // Store access multiple.
2057: defm STAM : StoreMultipleRSPair<"stam", 0x9B, 0xEB9B, AR32>;
2058: 
2059: //===----------------------------------------------------------------------===//
2060: // Program mask and addressing mode
2061: //===----------------------------------------------------------------------===//
2062: 
2063: // Extract CC and program mask into a register.  CC ends up in bits 29 and 28.
2064: let Uses = [CC] in
2065:   def IPM : InherentRRE<"ipm", 0xB222, GR32, z_ipm>;
2066: 
2067: // Set CC and program mask from a register.
2068: let hasSideEffects = 1, Defs = [CC] in
2069:   def SPM : SideEffectUnaryRR<"spm", 0x04, GR32>;
2070: 
2071: // Branch and link - like BAS, but also extracts CC and program mask.
2072: let isCall = 1, Uses = [CC], Defs = [CC] in {
2073:   def BAL  : CallRX<"bal", 0x45>;
2074:   def BALR : CallRR<"balr", 0x05>;
2075: }
2076: 
2077: // Test addressing mode.
2078: let Defs = [CC] in
2079:   def TAM : SideEffectInherentE<"tam", 0x010B>;
2080: 
```
- **EN**: This block declares or refines TableGen records such as `EAR`, `SAR`, `CPYA`, `LAE`, `LAM`, `STAM`.
- **CN**: 该代码块声明或细化了 `EAR`, `SAR`, `CPYA`, `LAE`, `LAM`, `STAM` 等 TableGen 记录。

### Lines 2081-2120 / 第 2081-2120 行
```tablegen
2081: // Set addressing mode.
2082: let hasSideEffects = 1 in {
2083:   def SAM24 : SideEffectInherentE<"sam24", 0x010C>;
2084:   def SAM31 : SideEffectInherentE<"sam31", 0x010D>;
2085:   def SAM64 : SideEffectInherentE<"sam64", 0x010E>;
2086: }
2087: 
2088: // Branch and set mode.  Not really a call, but also sets an output register.
2089: let isBranch = 1, isTerminator = 1, isBarrier = 1 in
2090:   def BSM : CallRR<"bsm", 0x0B>;
2091: 
2092: // Branch and save and set mode.
2093: let isCall = 1, Defs = [CC] in
2094:   def BASSM : CallRR<"bassm", 0x0C>;
2095: 
2096: //===----------------------------------------------------------------------===//
2097: // Transactional execution
2098: //===----------------------------------------------------------------------===//
2099: 
2100: let hasSideEffects = 1, Predicates = [FeatureTransactionalExecution] in {
2101:   // Transaction Begin
2102:   let mayStore = 1, usesCustomInserter = 1, Defs = [CC] in {
2103:     def TBEGIN : TestBinarySIL<"tbegin", 0xE560, z_tbegin, imm32zx16>;
2104:     let hasNoSchedulingInfo = 1 in
2105:      def TBEGIN_nofloat : TestBinarySILPseudo<z_tbegin_nofloat, imm32zx16>;
2106:     def TBEGINC : SideEffectBinarySIL<"tbeginc", 0xE561,
2107:                                       int_s390_tbeginc, imm32zx16>;
2108:   }
2109: 
2110:   // Transaction End
2111:   let Defs = [CC] in
2112:     def TEND : TestInherentS<"tend", 0xB2F8, z_tend>;
2113: 
2114:   // Transaction Abort
2115:   let isTerminator = 1, isBarrier = 1, mayStore = 1,
2116:       hasSideEffects = 1 in
2117:     def TABORT : SideEffectAddressS<"tabort", 0xB2FC, int_s390_tabort>;
2118: 
2119:   // Nontransactional Store
2120:   def NTSTG : StoreRXY<"ntstg", 0xE325, int_s390_ntstg, GR64, 8>;
```
- **EN**: This block declares or refines TableGen records such as `SAM24`, `SAM31`, `SAM64`, `BSM`, `BASSM`, `TBEGIN`.
- **CN**: 该代码块声明或细化了 `SAM24`, `SAM31`, `SAM64`, `BSM`, `BASSM`, `TBEGIN` 等 TableGen 记录。

### Lines 2121-2160 / 第 2121-2160 行
```tablegen
2121: 
2122:   // Extract Transaction Nesting Depth
2123:   def ETND : InherentRRE<"etnd", 0xB2EC, GR32, int_s390_etnd>;
2124: }
2125: 
2126: //===----------------------------------------------------------------------===//
2127: // Processor assist
2128: //===----------------------------------------------------------------------===//
2129: 
2130: let Predicates = [FeatureProcessorAssist] in {
2131:   let hasSideEffects = 1 in
2132:     def PPA : SideEffectTernaryRRFc<"ppa", 0xB2E8, GR64, GR64, imm32zx4>;
2133:   def : Pat<(int_s390_ppa_txassist GR32:$src),
2134:             (PPA (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GR32:$src, subreg_l32),
2135:                  zero_reg, 1)>;
2136: }
2137: 
2138: //===----------------------------------------------------------------------===//
2139: // Miscellaneous Instructions.
2140: //===----------------------------------------------------------------------===//
2141: 
2142: // Count leading/trailing zeros.
2143: let Predicates = [FeatureMiscellaneousExtensions4] in {
2144:   def CLZG : UnaryRRE<"clzg", 0xB968, ctlz, GR64, GR64>;
2145:   def CTZG : UnaryRRE<"ctzg", 0xB969, cttz, GR64, GR64>;
2146: }
2147: 
2148: // Find leftmost one, AKA count leading zeros.  The instruction actually
2149: // returns a pair of GR64s, the first giving the number of leading zeros
2150: // and the second giving a copy of the source with the leftmost one bit
2151: // cleared.  We only use the first result here.
2152: let Defs = [CC] in
2153:   def FLOGR : UnaryRRE<"flogr", 0xB983, null_frag, GR128, GR64>;
2154: def : Pat<(i64 (ctlz GR64:$src)),
2155:           (EXTRACT_SUBREG (FLOGR GR64:$src), subreg_h64)>;
2156: 
2157: // Population count.  Counts bits set per byte or doubleword.
2158: let Predicates = [FeatureMiscellaneousExtensions3] in {
2159:   let Defs = [CC] in
2160:     def POPCNTOpt : BinaryRRFc<"popcnt", 0xB9E1, GR64, GR64>;
```
- **EN**: This block declares or refines TableGen records such as `ETND`, `PPA`, `CLZG`, `CTZG`, `FLOGR`, `POPCNTOpt`.
- **CN**: 该代码块声明或细化了 `ETND`, `PPA`, `CLZG`, `CTZG`, `FLOGR`, `POPCNTOpt` 等 TableGen 记录。

### Lines 2161-2200 / 第 2161-2200 行
```tablegen
2161:   def : Pat<(ctpop GR64:$src), (POPCNTOpt GR64:$src, 8)>;
2162: }
2163: let Predicates = [FeaturePopulationCount], Defs = [CC] in
2164:   def POPCNT : UnaryRRE<"popcnt", 0xB9E1, z_popcnt, GR64, GR64>;
2165: 
2166: // Bit deposit and bit extract.
2167: let Predicates = [FeatureMiscellaneousExtensions4] in {
2168:   def BDEPG : BinaryRRFa<"bdepg", 0xB96D, int_s390_bdepg, GR64, GR64, GR64>;
2169:   def BEXTG : BinaryRRFa<"bextg", 0xB96C, int_s390_bextg, GR64, GR64, GR64>;
2170: }
2171: 
2172: // Search a block of memory for a character.
2173: let mayLoad = 1, Defs = [CC] in
2174:   defm SRST : StringRRE<"srst", 0xB25E, z_search_string>;
2175: let mayLoad = 1, Defs = [CC], Uses = [R0L] in
2176:   def SRSTU : SideEffectBinaryMemMemRRE<"srstu", 0xB9BE, GR64, GR64>;
2177: 
2178: // Compare until substring equal.
2179: let mayLoad = 1, Defs = [CC], Uses = [R0L, R1L] in
2180:   def CUSE : SideEffectBinaryMemMemRRE<"cuse", 0xB257, GR128, GR128>;
2181: 
2182: // Compare and form codeword.
2183: let mayLoad = 1, Defs = [CC, R1D, R2D, R3D], Uses = [R1D, R2D, R3D] in
2184:   def CFC : SideEffectAddressS<"cfc", 0xB21A, null_frag>;
2185: 
2186: // Update tree.
2187: let mayLoad = 1, mayStore = 1, Defs = [CC, R0D, R1D, R2D, R3D, R5D],
2188:     Uses = [R0D, R1D, R2D, R3D, R4D, R5D] in
2189:   def UPT : SideEffectInherentE<"upt", 0x0102>;
2190: 
2191: // Checksum.
2192: let mayLoad = 1, Defs = [CC] in
2193:   def CKSM : SideEffectBinaryMemMemRRE<"cksm", 0xB241, GR32, GR128>;
2194: 
2195: // Compression call.
2196: let mayLoad = 1, mayStore = 1, Defs = [CC, R1D], Uses = [R0L, R1D] in
2197:   def CMPSC : SideEffectBinaryMemMemRRE<"cmpsc", 0xB263, GR128, GR128>;
2198: 
2199: // Sort lists.
2200: let Predicates = [FeatureEnhancedSort],
```
- **EN**: This block declares or refines TableGen records such as `POPCNT`, `BDEPG`, `BEXTG`, `SRST`, `SRSTU`, `CUSE`.
- **CN**: 该代码块声明或细化了 `POPCNT`, `BDEPG`, `BEXTG`, `SRST`, `SRSTU`, `CUSE` 等 TableGen 记录。

### Lines 2201-2240 / 第 2201-2240 行
```tablegen
2201:     mayLoad = 1, mayStore = 1, Defs = [CC], Uses = [R0L, R1D] in
2202:   def SORTL : SideEffectBinaryMemMemRRE<"sortl", 0xB938, GR128, GR128>;
2203: 
2204: // Deflate conversion call.
2205: let Predicates = [FeatureDeflateConversion],
2206:     mayLoad = 1, mayStore = 1, Defs = [CC], Uses = [R0L, R1D] in
2207:   def DFLTCC : SideEffectTernaryMemMemRRFa<"dfltcc", 0xB939,
2208:                                            GR128, GR128, GR64>;
2209: 
2210: // NNPA.
2211: let Predicates = [FeatureNNPAssist],
2212:     mayLoad = 1, mayStore = 1, Defs = [R0D, CC], Uses = [R0D, R1D] in
2213:   def NNPA : SideEffectInherentRRE<"nnpa", 0xB93B>;
2214: 
2215: // Execute.
2216: let hasSideEffects = 1 in {
2217:   def EX   : SideEffectBinaryRX<"ex", 0x44, ADDR64>;
2218:   def EXRL : SideEffectBinaryRILPC<"exrl", 0xC60, ADDR64>;
2219:   let hasNoSchedulingInfo = 1 in
2220:     def EXRL_Pseudo : Alias<6, (outs), (ins i64imm:$TargetOpc, ADDR64:$lenMinus1,
2221:                                           bdaddr12only:$bdl1, bdaddr12only:$bd2),
2222:                                           []>;
2223: }
2224: 
2225: //===----------------------------------------------------------------------===//
2226: // .insn directive instructions
2227: //===----------------------------------------------------------------------===//
2228: 
2229: let isCodeGenOnly = 1, hasSideEffects = 1 in {
2230:   def InsnE   : DirectiveInsnE<(outs), (ins imm64zx16:$enc), ".insn e,$enc", []>;
2231:   def InsnRI  : DirectiveInsnRI<(outs), (ins imm64zx32:$enc, AnyReg:$R1,
2232:                                              imm32sx16:$I2),
2233:                                 ".insn ri,$enc,$R1,$I2", []>;
2234:   def InsnRIE : DirectiveInsnRIE<(outs), (ins imm64zx48:$enc, AnyReg:$R1,
2235:                                               AnyReg:$R3, brtarget16:$I2),
2236:                                  ".insn rie,$enc,$R1,$R3,$I2", []>;
2237:   def InsnRIL : DirectiveInsnRIL<(outs), (ins imm64zx48:$enc, AnyReg:$R1,
2238:                                               brtarget32:$I2),
2239:                                  ".insn ril,$enc,$R1,$I2", []>;
2240:   def InsnRILU : DirectiveInsnRIL<(outs), (ins imm64zx48:$enc, AnyReg:$R1,
```
- **EN**: This block declares or refines TableGen records such as `SORTL`, `DFLTCC`, `NNPA`, `EX`, `EXRL`, `EXRL_Pseudo`.
- **CN**: 该代码块声明或细化了 `SORTL`, `DFLTCC`, `NNPA`, `EX`, `EXRL`, `EXRL_Pseudo` 等 TableGen 记录。

### Lines 2241-2280 / 第 2241-2280 行
```tablegen
2241:                                                uimm32:$I2),
2242:                                   ".insn rilu,$enc,$R1,$I2", []>;
2243:   def InsnRIS : DirectiveInsnRIS<(outs),
2244:                                  (ins imm64zx48:$enc, AnyReg:$R1,
2245:                                       imm32sx8:$I2, imm32zx4:$M3,
2246:                                       (bdaddr12only $B4, $D4):$BD4),
2247:                                  ".insn ris,$enc,$R1,$I2,$M3,$BD4", []>;
2248:   def InsnRR : DirectiveInsnRR<(outs),
2249:                                (ins imm64zx16:$enc, AnyReg:$R1, AnyReg:$R2),
2250:                                ".insn rr,$enc,$R1,$R2", []>;
2251:   def InsnRRE : DirectiveInsnRRE<(outs), (ins imm64zx32:$enc,
2252:                                               AnyReg:$R1, AnyReg:$R2),
2253:                                  ".insn rre,$enc,$R1,$R2", []>;
2254:   def InsnRRF : DirectiveInsnRRF<(outs),
2255:                                  (ins imm64zx32:$enc, AnyReg:$R1, AnyReg:$R2,
2256:                                       AnyReg:$R3, imm32zx4:$M4),
2257:                                  ".insn rrf,$enc,$R1,$R2,$R3,$M4", []>;
2258:   def InsnRRS : DirectiveInsnRRS<(outs),
2259:                                  (ins imm64zx48:$enc, AnyReg:$R1,
2260:                                       AnyReg:$R2, imm32zx4:$M3,
2261:                                       (bdaddr12only $B4, $D4):$BD4),
2262:                                  ".insn rrs,$enc,$R1,$R2,$M3,$BD4", []>;
2263:   def InsnRS  : DirectiveInsnRS<(outs),
2264:                                 (ins imm64zx32:$enc, AnyReg:$R1,
2265:                                      AnyReg:$R3, (bdaddr12only $B2, $D2):$BD2),
2266:                                 ".insn rs,$enc,$R1,$R3,$BD2", []>;
2267:   def InsnRSE : DirectiveInsnRSE<(outs),
2268:                                  (ins imm64zx48:$enc, AnyReg:$R1,
2269:                                       AnyReg:$R3, (bdaddr12only $B2, $D2):$BD2),
2270:                                  ".insn rse,$enc,$R1,$R3,$BD2", []>;
2271:   def InsnRSI : DirectiveInsnRSI<(outs),
2272:                                  (ins imm64zx48:$enc, AnyReg:$R1,
2273:                                       AnyReg:$R3, brtarget16:$RI2),
2274:                                  ".insn rsi,$enc,$R1,$R3,$RI2", []>;
2275:   def InsnRSY : DirectiveInsnRSY<(outs),
2276:                                  (ins imm64zx48:$enc, AnyReg:$R1,
2277:                                       AnyReg:$R3, (bdaddr20only $B2, $D2):$BD2),
2278:                                  ".insn rsy,$enc,$R1,$R3,$BD2", []>;
2279:   def InsnRX  : DirectiveInsnRX<(outs), (ins imm64zx32:$enc, AnyReg:$R1,
2280:                                              (bdxaddr12only $B2, $D2, $X2):$XBD2),
```
- **EN**: This block declares or refines TableGen records such as `InsnRIS`, `InsnRR`, `InsnRRE`, `InsnRRF`, `InsnRRS`, `InsnRS`.
- **CN**: 该代码块声明或细化了 `InsnRIS`, `InsnRR`, `InsnRRE`, `InsnRRF`, `InsnRRS`, `InsnRS` 等 TableGen 记录。

### Lines 2281-2320 / 第 2281-2320 行
```tablegen
2281:                                 ".insn rx,$enc,$R1,$XBD2", []>;
2282:   def InsnRXE : DirectiveInsnRXE<(outs), (ins imm64zx48:$enc, AnyReg:$R1,
2283:                                               (bdxaddr12only $B2, $D2, $X2):$XBD2),
2284:                                  ".insn rxe,$enc,$R1,$XBD2", []>;
2285:   def InsnRXF : DirectiveInsnRXF<(outs),
2286:                                  (ins imm64zx48:$enc, AnyReg:$R1,
2287:                                       AnyReg:$R3, (bdxaddr12only $B2, $D2, $X2):$XBD2),
2288:                                  ".insn rxf,$enc,$R1,$R3,$XBD2", []>;
2289:   def InsnRXY : DirectiveInsnRXY<(outs), (ins imm64zx48:$enc, AnyReg:$R1,
2290:                                               (bdxaddr20only $B2, $D2, $X2):$XBD2),
2291:                                  ".insn rxy,$enc,$R1,$XBD2", []>;
2292:   def InsnS : DirectiveInsnS<(outs),
2293:                              (ins imm64zx32:$enc, (bdaddr12only $B2, $D2):$BD2),
2294:                              ".insn s,$enc,$BD2", []>;
2295:   def InsnSI : DirectiveInsnSI<(outs),
2296:                                (ins imm64zx32:$enc, (bdaddr12only $B1, $D1):$BD1,
2297:                                     imm32sx8:$I2),
2298:                                ".insn si,$enc,$BD1,$I2", []>;
2299:   def InsnSIY : DirectiveInsnSIY<(outs),
2300:                                  (ins imm64zx48:$enc,
2301:                                       (bdaddr20only $B1, $D1):$BD1, imm32zx8:$I2),
2302:                                  ".insn siy,$enc,$BD1,$I2", []>;
2303:   def InsnSIL : DirectiveInsnSIL<(outs),
2304:                                  (ins imm64zx48:$enc, (bdaddr12only $B1, $D1):$BD1,
2305:                                       imm32zx16:$I2),
2306:                                  ".insn sil,$enc,$BD1,$I2", []>;
2307:   def InsnSS : DirectiveInsnSS<(outs),
2308:                                (ins imm64zx48:$enc, (bdraddr12only $B1, $D1, $R1):$RBD1,
2309:                                     (bdaddr12only $B2, $D2):$BD2, AnyReg:$R3),
2310:                                ".insn ss,$enc,$RBD1,$BD2,$R3", []>;
2311:   def InsnSSE : DirectiveInsnSSE<(outs),
2312:                                  (ins imm64zx48:$enc,
2313:                                       (bdaddr12only $B1, $D1):$BD1,(bdaddr12only $B2, $D2):$BD2),
2314:                                  ".insn sse,$enc,$BD1,$BD2", []>;
2315:   def InsnSSF : DirectiveInsnSSF<(outs),
2316:                                  (ins imm64zx48:$enc, (bdaddr12only $B1, $D1):$BD1,
2317:                                       (bdaddr12only $B2, $D2):$BD2, AnyReg:$R3),
2318:                                  ".insn ssf,$enc,$BD1,$BD2,$R3", []>;
2319:   def InsnVRI : DirectiveInsnVRI<(outs),
2320:                                  (ins imm64zx48:$enc, VR128:$V1, VR128:$V2,
```
- **EN**: This block declares or refines TableGen records such as `InsnRXE`, `InsnRXF`, `InsnRXY`, `InsnS`, `InsnSI`, `InsnSIY`.
- **CN**: 该代码块声明或细化了 `InsnRXE`, `InsnRXF`, `InsnRXY`, `InsnS`, `InsnSI`, `InsnSIY` 等 TableGen 记录。

### Lines 2321-2360 / 第 2321-2360 行
```tablegen
2321:                                   imm32zx12:$I3, imm32zx4:$M4, imm32zx4:$M5),
2322:                                  ".insn vri,$enc,$V1,$V2,$I3,$M4,$M5", []>;
2323:   def InsnVRR : DirectiveInsnVRR<(outs),
2324:                                  (ins imm64zx48:$enc, VR128:$V1, VR128:$V2,
2325:                                   VR128:$V3, imm32zx4:$M4, imm32zx4:$M5,
2326:                                   imm32zx4:$M6),
2327:                                   ".insn vrr,$enc,$V1,$V2,$V3,$M4,$M5,$M6", []>;
2328:   def InsnVRS : DirectiveInsnVRS<(outs),
2329:                                  (ins imm64zx48:$enc, AnyReg:$R1, VR128:$V3,
2330:                                   (bdaddr12only $B2, $D2):$BD2, imm32zx4:$M4),
2331:                                  ".insn vrs,$enc,$BD2,$M4", []>;
2332:   def InsnVRV : DirectiveInsnVRV<(outs),
2333:                                  (ins imm64zx48:$enc, VR128:$V1,
2334:                                       (bdvaddr12only $B2, $D2, $V2):$VBD2, imm32zx4:$M3),
2335:                                  ".insn vrv,$enc,$V1,$VBD2,$M3", []>;
2336:   def InsnVRX : DirectiveInsnVRX<(outs),
2337:                                  (ins imm64zx48:$enc, VR128:$V1,
2338:                                   (bdxaddr12only $B2, $D2, $X2):$XBD2, imm32zx4:$M3),
2339:                                  ".insn vrx,$enc,$V1,$XBD2,$M3", []>;
2340:   def InsnVSI : DirectiveInsnVSI<(outs),
2341:                                  (ins imm64zx48:$enc, VR128:$V1,
2342:                                   (bdaddr12only $B2, $D2):$BD2, imm32zx8:$I3),
2343:                                   ".insn vsi,$enc,$V1,$BD2,$I3", []>;
2344: }
2345: 
2346: //===----------------------------------------------------------------------===//
2347: // Peepholes.
2348: //===----------------------------------------------------------------------===//
2349: 
2350: // Avoid generating 2 XOR instructions. (xor (and x, y), y) is
2351: // equivalent to (and (xor x, -1), y)
2352: def : Pat<(and (xor GR64:$x, (i64 -1)), GR64:$y),
2353:                           (XGR GR64:$y, (NGR GR64:$y, GR64:$x))>;
2354: 
2355: // Use LCGR/AGHI for i64 xor with -1.
2356: def : Pat<(xor GR64:$x, (i64 -1)),
2357:           (AGHI (LCGR GR64:$x), (i64 -1))>;
2358: 
2359: // Shift/rotate instructions only use the last 6 bits of the second operand
2360: // register, so we can safely use NILL (16 fewer bits than NILF) to only AND the
```
- **EN**: This block declares or refines TableGen records such as `InsnVRR`, `InsnVRS`, `InsnVRV`, `InsnVRX`, `InsnVSI`.
- **CN**: 该代码块声明或细化了 `InsnVRR`, `InsnVRS`, `InsnVRV`, `InsnVRX`, `InsnVSI` 等 TableGen 记录。

### Lines 2361-2400 / 第 2361-2400 行
```tablegen
2361: // last 16 bits.
2362: // Complexity is added so that we match this before we match NILF on the AND
2363: // operation alone.
2364: let AddedComplexity = 4 in {
2365:   def : Pat<(shl GR32:$val, (and GR32:$shift, imm32zx16trunc:$imm)),
2366:             (SLL GR32:$val, (NILL GR32:$shift, imm32zx16trunc:$imm), 0)>;
2367: 
2368:   def : Pat<(sra GR32:$val, (and GR32:$shift, imm32zx16trunc:$imm)),
2369:             (SRA GR32:$val, (NILL GR32:$shift, imm32zx16trunc:$imm), 0)>;
2370: 
2371:   def : Pat<(srl GR32:$val, (and GR32:$shift, imm32zx16trunc:$imm)),
2372:             (SRL GR32:$val, (NILL GR32:$shift, imm32zx16trunc:$imm), 0)>;
2373: 
2374:   def : Pat<(shl GR64:$val, (and GR32:$shift, imm32zx16trunc:$imm)),
2375:             (SLLG GR64:$val, (NILL GR32:$shift, imm32zx16trunc:$imm), 0)>;
2376: 
2377:   def : Pat<(sra GR64:$val, (and GR32:$shift, imm32zx16trunc:$imm)),
2378:             (SRAG GR64:$val, (NILL GR32:$shift, imm32zx16trunc:$imm), 0)>;
2379: 
2380:   def : Pat<(srl GR64:$val, (and GR32:$shift, imm32zx16trunc:$imm)),
2381:             (SRLG GR64:$val, (NILL GR32:$shift, imm32zx16trunc:$imm), 0)>;
2382: 
2383:   def : Pat<(rotl GR32:$val, (and GR32:$shift, imm32zx16trunc:$imm)),
2384:             (RLL GR32:$val, (NILL GR32:$shift, imm32zx16trunc:$imm), 0)>;
2385: 
2386:   def : Pat<(rotl GR64:$val, (and GR32:$shift, imm32zx16trunc:$imm)),
2387:             (RLLG GR64:$val, (NILL GR32:$shift, imm32zx16trunc:$imm), 0)>;
2388: }
2389: 
2390: // Substitute (x*64-s) with (-s), since shift/rotate instructions only
2391: // use the last 6 bits of the second operand register (making it modulo 64).
2392: let AddedComplexity = 4 in {
2393:   def : Pat<(shl GR64:$val, (sub imm32mod64,  GR32:$shift)),
2394:             (SLLG GR64:$val, (LCR GR32:$shift), 0)>;
2395: 
2396:   def : Pat<(sra GR64:$val, (sub imm32mod64,  GR32:$shift)),
2397:             (SRAG GR64:$val, (LCR GR32:$shift), 0)>;
2398: 
2399:   def : Pat<(srl GR64:$val, (sub imm32mod64,  GR32:$shift)),
2400:             (SRLG GR64:$val, (LCR GR32:$shift), 0)>;
```
- **EN**: This span continues the file's main responsibility: defines SystemZ backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2401-2440 / 第 2401-2440 行
```tablegen
2401: 
2402:   def : Pat<(rotl GR64:$val, (sub imm32mod64,  GR32:$shift)),
2403:             (RLLG GR64:$val, (LCR GR32:$shift), 0)>;
2404: }
2405: 
2406: // Peepholes for turning scalar operations into block operations.  The length
2407: // is given as one less for these pseudos.
2408: defm : BlockLoadStore<anyextloadi8, i32, MVCImm, NCImm, OCImm, XCImm, 0>;
2409: defm : BlockLoadStore<anyextloadi16, i32, MVCImm, NCImm, OCImm, XCImm, 1>;
2410: defm : BlockLoadStore<load, i32, MVCImm, NCImm, OCImm, XCImm, 3>;
2411: defm : BlockLoadStore<anyextloadi8, i64, MVCImm, NCImm, OCImm, XCImm, 0>;
2412: defm : BlockLoadStore<anyextloadi16, i64, MVCImm, NCImm, OCImm, XCImm, 1>;
2413: defm : BlockLoadStore<anyextloadi32, i64, MVCImm, NCImm, OCImm, XCImm, 3>;
2414: defm : BlockLoadStore<load, i64, MVCImm, NCImm, OCImm, XCImm, 7>;
2415: 
2416: //===----------------------------------------------------------------------===//
2417: // Mnemonic Aliases
2418: //===----------------------------------------------------------------------===//
2419: 
2420: def JCT   : MnemonicAlias<"jct", "brct">;
2421: def JCTG  : MnemonicAlias<"jctg", "brctg">;
2422: def JC    : MnemonicAlias<"jc", "brc">;
2423: def JCTH  : MnemonicAlias<"jcth", "brcth">;
2424: def JAS   : MnemonicAlias<"jas", "bras">;
2425: def JASL  : MnemonicAlias<"jasl", "brasl">;
2426: def JXH   : MnemonicAlias<"jxh", "brxh">;
2427: def JXLE  : MnemonicAlias<"jxle", "brxle">;
2428: def JXHG  : MnemonicAlias<"jxhg", "brxhg">;
2429: def JXLEG : MnemonicAlias<"jxleg", "brxlg">;
2430: 
2431: def BRU   : MnemonicAlias<"bru", "j">;
2432: def BRUL  : MnemonicAlias<"brul", "jg", "gnu">;
2433: def BRUL_HLASM  : MnemonicAlias<"brul", "jlu", "hlasm">;
2434: 
2435: foreach V = [ "E", "NE", "H", "NH", "L", "NL", "HE", "NHE", "LE", "NLE",
2436:               "Z", "NZ", "P", "NP", "M", "NM", "LH", "NLH", "O", "NO" ] in {
2437:   defm BRUAsm#V  : MnemonicCondBranchAlias <CV<V>, "br#", "j#">;
2438:   defm BRULAsm#V : MnemonicCondBranchAlias <CV<V>, "br#l", "jg#", "gnu">;
2439:   defm BRUL_HLASMAsm#V : MnemonicCondBranchAlias <CV<V>, "br#l", "jl#", "hlasm">;
2440: }
```
- **EN**: This block declares or refines TableGen records such as `JCT`, `JCTG`, `JC`, `JCTH`, `JAS`, `JASL`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `JCT`, `JCTG`, `JC`, `JCTH`, `JAS`, `JASL` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
