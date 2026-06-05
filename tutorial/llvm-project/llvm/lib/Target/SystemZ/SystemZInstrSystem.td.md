# SystemZInstrSystem.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZInstrSystem.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: //==- SystemZInstrSystem.td - SystemZ system instructions -*- tblgen-*-----==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // The instructions in this file implement SystemZ system-level instructions.
  10: // Most of these instructions are privileged or semi-privileged.  They are
  11: // not used for code generation, but are provided for use with the assembler
  12: // and disassembler only.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: //===----------------------------------------------------------------------===//
  17: // Program-Status Word Instructions.
  18: //===----------------------------------------------------------------------===//
  19: 
  20: // Extract PSW.
  21: let hasSideEffects = 1, Uses = [CC] in
  22:   def EPSW : InherentDualRRE<"epsw", 0xB98D, GR32>;
  23: 
  24: // Load PSW (extended).
  25: let hasSideEffects = 1, Defs = [CC] in {
  26:   def LPSW : SideEffectUnaryS<"lpsw", 0x8200, null_frag, 8>;
  27:   def LPSWE : SideEffectUnaryS<"lpswe", 0xB2B2, null_frag, 16>;
  28: }
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `EPSW`, `LPSW`, `LPSWE`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `EPSW`, `LPSW`, `LPSWE` 等 TableGen 记录。

### Lines 29-56 / 第 29-56 行
```tablegen
  29: let Predicates = [FeatureBEAREnhancement], hasSideEffects = 1, Defs = [CC] in
  30:   def LPSWEY : SideEffectUnarySIY<"lpswey", 0xEB71, 16>;
  31: 
  32: // Insert PSW key.
  33: let Uses = [R2L], Defs = [R2L] in
  34:   def IPK : SideEffectInherentS<"ipk", 0xB20B, null_frag>;
  35: 
  36: // Set PSW key from address.
  37: let hasSideEffects = 1 in
  38:   def SPKA : SideEffectAddressS<"spka", 0xB20A, null_frag>;
  39: 
  40: // Set system mask.
  41: let hasSideEffects = 1 in
  42:   def SSM : SideEffectUnaryS<"ssm", 0x8000, null_frag, 1>;
  43: 
  44: // Store then AND/OR system mask.
  45: let hasSideEffects = 1 in {
  46:   def STNSM : StoreSI<"stnsm", 0xAC, null_frag, imm32zx8>;
  47:   def STOSM : StoreSI<"stosm", 0xAD, null_frag, imm32zx8>;
  48: }
  49: 
  50: // Insert address space control.
  51: let hasSideEffects = 1 in
  52:   def IAC : InherentRRE<"iac", 0xB224, GR32, null_frag>;
  53: 
  54: // Set address space control (fast).
  55: let hasSideEffects = 1 in {
  56:   def SAC : SideEffectAddressS<"sac", 0xB219, null_frag>;
```
- **EN**: This block declares or refines TableGen records such as `LPSWEY`, `IPK`, `SPKA`, `SSM`, `STNSM`, `STOSM`.
- **CN**: 该代码块声明或细化了 `LPSWEY`, `IPK`, `SPKA`, `SSM`, `STNSM`, `STOSM` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57:   def SACF : SideEffectAddressS<"sacf", 0xB279, null_frag>;
  58: }
  59: 
  60: //===----------------------------------------------------------------------===//
  61: // Control Register Instructions.
  62: //===----------------------------------------------------------------------===//
  63: 
  64: let hasSideEffects = 1 in {
  65:   // Load control.
  66:   def LCTL : LoadMultipleRS<"lctl", 0xB7, CR64>;
  67:   def LCTLG : LoadMultipleRSY<"lctlg", 0xEB2F, CR64>;
  68: 
  69:   // Store control.
  70:   def STCTL : StoreMultipleRS<"stctl", 0xB6, CR64>;
  71:   def STCTG : StoreMultipleRSY<"stctg", 0xEB25, CR64>;
  72: }
  73: 
  74: // Extract primary ASN (and instance).
  75: let hasSideEffects = 1 in {
  76:   def EPAR : InherentRRE<"epar", 0xB226, GR32, null_frag>;
  77:   def EPAIR : InherentRRE<"epair", 0xB99A, GR64, null_frag>;
  78: }
  79: 
  80: // Extract secondary ASN (and instance).
  81: let hasSideEffects = 1 in {
  82:   def ESAR : InherentRRE<"esar", 0xB227, GR32, null_frag>;
  83:   def ESAIR : InherentRRE<"esair", 0xB99B, GR64, null_frag>;
  84: }
```
- **EN**: This block declares or refines TableGen records such as `SACF`, `LCTL`, `LCTLG`, `STCTL`, `STCTG`, `EPAR`.
- **CN**: 该代码块声明或细化了 `SACF`, `LCTL`, `LCTLG`, `STCTL`, `STCTG`, `EPAR` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85: 
  86: // Set secondary ASN (and instance).
  87: let hasSideEffects = 1 in {
  88:   def SSAR : SideEffectUnaryRRE<"ssar", 0xB225, GR32, null_frag>;
  89:   def SSAIR : SideEffectUnaryRRE<"ssair", 0xB99F, GR64, null_frag>;
  90: }
  91: 
  92: // Extract and set extended authority.
  93: let hasSideEffects = 1 in
  94:   def ESEA : UnaryTiedRRE<"esea", 0xB99D, GR32>;
  95: 
  96: //===----------------------------------------------------------------------===//
  97: // Prefix-Register Instructions.
  98: //===----------------------------------------------------------------------===//
  99: 
 100: // Set prefix.
 101: let hasSideEffects = 1 in
 102:   def SPX : SideEffectUnaryS<"spx", 0xB210, null_frag, 4>;
 103: 
 104: // Store prefix.
 105: let hasSideEffects = 1 in
 106:   def STPX : StoreInherentS<"stpx", 0xB211, null_frag, 4>;
 107: 
 108: //===----------------------------------------------------------------------===//
 109: // Breaking-Event-Address-Register Instructions.
 110: //===----------------------------------------------------------------------===//
 111: 
 112: let Predicates = [FeatureBEAREnhancement] in {
```
- **EN**: This block declares or refines TableGen records such as `SSAR`, `SSAIR`, `ESEA`, `SPX`, `STPX`.
- **CN**: 该代码块声明或细化了 `SSAR`, `SSAIR`, `ESEA`, `SPX`, `STPX` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113:   // Load BEAR.
 114:   let hasSideEffects = 1 in
 115:     def LBEAR : SideEffectUnaryS<"lbear", 0xB200, null_frag, 8>;
 116: 
 117:   // Store BEAR.
 118:   let hasSideEffects = 1 in
 119:     def STBEAR : StoreInherentS<"stbear", 0xB201, null_frag, 8>;
 120: }
 121: 
 122: //===----------------------------------------------------------------------===//
 123: // Storage-Key and Real Memory Instructions.
 124: //===----------------------------------------------------------------------===//
 125: 
 126: // Insert storage key extended.
 127: let hasSideEffects = 1 in
 128:   def ISKE : BinaryRRE<"iske", 0xB229, null_frag, GR32, GR64>;
 129: 
 130: // Insert virtual storage key.
 131: let hasSideEffects = 1 in
 132:   def IVSK : BinaryRRE<"ivsk", 0xB223, null_frag, GR32, GR64>;
 133: 
 134: // Set storage key extended.
 135: let hasSideEffects = 1, Defs = [CC] in
 136:   defm SSKE : SideEffectTernaryRRFcOpt<"sske", 0xB22B, GR32, GR64>;
 137: 
 138: // Reset reference bit extended.
 139: let hasSideEffects = 1, Defs = [CC] in
 140:   def RRBE : SideEffectBinaryRRE<"rrbe", 0xB22A, GR32, GR64>;
```
- **EN**: This block declares or refines TableGen records such as `LBEAR`, `STBEAR`, `ISKE`, `IVSK`, `SSKE`, `RRBE`.
- **CN**: 该代码块声明或细化了 `LBEAR`, `STBEAR`, `ISKE`, `IVSK`, `SSKE`, `RRBE` 等 TableGen 记录。

### Lines 141-168 / 第 141-168 行
```tablegen
 141: 
 142: // Reset reference bits multiple.
 143: let Predicates = [FeatureResetReferenceBitsMultiple], hasSideEffects = 1 in
 144:   def RRBM : UnaryRRE<"rrbm", 0xB9AE, null_frag, GR64, GR64>;
 145: 
 146: // Insert reference bits multiple.
 147: let Predicates = [FeatureInsertReferenceBitsMultiple], hasSideEffects = 1 in
 148:   def IRBM : UnaryRRE<"irbm", 0xB9AC, null_frag, GR64, GR64>;
 149: 
 150: // Perform frame management function.
 151: let hasSideEffects = 1 in
 152:   def PFMF : SideEffectBinaryMemRRE<"pfmf", 0xB9AF, GR32, GR64>;
 153: 
 154: // Test block.
 155: let hasSideEffects = 1, mayStore = 1, Uses = [R0D], Defs = [R0D, CC] in
 156:   def TB : SideEffectBinaryRRE<"tb", 0xB22C, GR64, GR64>;
 157: 
 158: // Page in / out.
 159: let mayLoad = 1, mayStore = 1, Defs = [CC] in {
 160:   def PGIN : SideEffectBinaryRRE<"pgin", 0xB22E, GR64, GR64>;
 161:   def PGOUT : SideEffectBinaryRRE<"pgout", 0xB22F, GR64, GR64>;
 162: }
 163: 
 164: //===----------------------------------------------------------------------===//
 165: // Dynamic-Address-Translation Instructions.
 166: //===----------------------------------------------------------------------===//
 167: 
 168: // Invalidate page table entry.
```
- **EN**: This block declares or refines TableGen records such as `RRBM`, `IRBM`, `PFMF`, `TB`, `PGIN`, `PGOUT`.
- **CN**: 该代码块声明或细化了 `RRBM`, `IRBM`, `PFMF`, `TB`, `PGIN`, `PGOUT` 等 TableGen 记录。

### Lines 169-196 / 第 169-196 行
```tablegen
 169: let hasSideEffects = 1 in
 170:   defm IPTE : SideEffectQuaternaryRRFaOptOpt<"ipte", 0xB221, GR64, GR32, GR32>;
 171: 
 172: // Invalidate DAT table entry.
 173: let hasSideEffects = 1 in
 174:   defm IDTE : SideEffectQuaternaryRRFbOpt<"idte", 0xB98E, GR64, GR64, GR64>;
 175: 
 176: // Reset DAT protection.
 177: let Predicates = [FeatureResetDATProtection], hasSideEffects = 1 in
 178:   defm RDP : SideEffectQuaternaryRRFbOpt<"rdp", 0xB98B, GR64, GR64, GR64>;
 179: 
 180: // Compare and replace DAT table entry.
 181: let Predicates = [FeatureEnhancedDAT2], hasSideEffects = 1, Defs = [CC] in
 182:   defm CRDTE : SideEffectQuaternaryRRFbOpt<"crdte", 0xB98F, GR128, GR128, GR64>;
 183: 
 184: // Purge TLB.
 185: let hasSideEffects = 1 in
 186:   def PTLB : SideEffectInherentS<"ptlb", 0xB20D, null_frag>;
 187: 
 188: // Compare and swap and purge.
 189: let hasSideEffects = 1, Defs = [CC] in {
 190:   def CSP : CmpSwapRRE<"csp", 0xB250, GR128, GR64>;
 191:   def CSPG : CmpSwapRRE<"cspg", 0xB98A, GR128, GR64>;
 192: }
 193: 
 194: // Load page-table-entry address.
 195: let hasSideEffects = 1, Defs = [CC] in
 196:   def LPTEA : TernaryRRFb<"lptea", 0xB9AA, GR64, GR64, GR64>;
```
- **EN**: This block declares or refines TableGen records such as `IPTE`, `IDTE`, `RDP`, `CRDTE`, `PTLB`, `CSP`.
- **CN**: 该代码块声明或细化了 `IPTE`, `IDTE`, `RDP`, `CRDTE`, `PTLB`, `CSP` 等 TableGen 记录。

### Lines 197-224 / 第 197-224 行
```tablegen
 197: 
 198: // Load real address.
 199: let hasSideEffects = 1, Defs = [CC] in {
 200:   defm LRA : LoadAddressRXPair<"lra", 0xB1, 0xE313, null_frag>;
 201:   def LRAG : LoadAddressRXY<"lrag", 0xE303, null_frag, laaddr20pair>;
 202: }
 203: 
 204: // Store real address.
 205: def STRAG : StoreSSE<"strag", 0xE502>;
 206: 
 207: // Load using real address.
 208: let mayLoad = 1 in {
 209:  def LURA : UnaryRRE<"lura", 0xB24B, null_frag, GR32, GR64>;
 210:  def LURAG : UnaryRRE<"lurag", 0xB905, null_frag, GR64, GR64>;
 211: }
 212: 
 213: // Store using real address.
 214: let mayStore = 1 in {
 215:  def STURA : SideEffectBinaryRRE<"stura", 0xB246, GR32, GR64>;
 216:  def STURG : SideEffectBinaryRRE<"sturg", 0xB925, GR64, GR64>;
 217: }
 218: 
 219: // Test protection.
 220: let hasSideEffects = 1, Defs = [CC] in
 221:   def TPROT : SideEffectBinarySSE<"tprot", 0xE501>;
 222: 
 223: //===----------------------------------------------------------------------===//
 224: // Memory-move Instructions.
```
- **EN**: This block declares or refines TableGen records such as `LRA`, `LRAG`, `STRAG`, `LURA`, `LURAG`, `STURA`.
- **CN**: 该代码块声明或细化了 `LRA`, `LRAG`, `STRAG`, `LURA`, `LURAG`, `STURA` 等 TableGen 记录。

### Lines 225-252 / 第 225-252 行
```tablegen
 225: //===----------------------------------------------------------------------===//
 226: 
 227: // Move with key.
 228: let mayLoad = 1, mayStore = 1, Defs = [CC] in
 229:   def MVCK : MemoryBinarySSd<"mvck", 0xD9, GR64>;
 230: 
 231: // Move to primary / secondary.
 232: let mayLoad = 1, mayStore = 1, Defs = [CC] in {
 233:   def MVCP : MemoryBinarySSd<"mvcp", 0xDA, GR64>;
 234:   def MVCS : MemoryBinarySSd<"mvcs", 0xDB, GR64>;
 235: }
 236: 
 237: // Move with source / destination key.
 238: let mayLoad = 1, mayStore = 1, Uses = [R0L, R1L] in {
 239:   def MVCSK : SideEffectBinarySSE<"mvcsk", 0xE50E>;
 240:   def MVCDK : SideEffectBinarySSE<"mvcdk", 0xE50F>;
 241: }
 242: 
 243: // Move with optional specifications.
 244: let mayLoad = 1, mayStore = 1, Uses = [R0L] in
 245:   def MVCOS : SideEffectTernarySSF<"mvcos", 0xC80, GR64>;
 246: 
 247: // Move page.
 248: let mayLoad = 1, mayStore = 1, Uses = [R0L], Defs = [CC] in
 249:   def MVPG : SideEffectBinaryRRE<"mvpg", 0xB254, GR64, GR64>;
 250: 
 251: //===----------------------------------------------------------------------===//
 252: // Address-Space Instructions.
```
- **EN**: This block declares or refines TableGen records such as `MVCK`, `MVCP`, `MVCS`, `MVCSK`, `MVCDK`, `MVCOS`.
- **CN**: 该代码块声明或细化了 `MVCK`, `MVCP`, `MVCS`, `MVCSK`, `MVCDK`, `MVCOS` 等 TableGen 记录。

### Lines 253-280 / 第 253-280 行
```tablegen
 253: //===----------------------------------------------------------------------===//
 254: 
 255: // Load address space parameters.
 256: let hasSideEffects = 1, Defs = [CC] in
 257:   def LASP : SideEffectBinarySSE<"lasp", 0xE500>;
 258: 
 259: // Purge ALB.
 260: let hasSideEffects = 1 in
 261:   def PALB : SideEffectInherentRRE<"palb", 0xB248>;
 262: 
 263: // Program call.
 264: let hasSideEffects = 1 in
 265:   def PC : SideEffectAddressS<"pc", 0xB218, null_frag>;
 266: 
 267: // Program return.
 268: let hasSideEffects = 1, Defs = [CC] in
 269:   def PR : SideEffectInherentE<"pr", 0x0101>;
 270: 
 271: // Program transfer (with instance).
 272: let hasSideEffects = 1 in {
 273:   def PT : SideEffectBinaryRRE<"pt", 0xB228, GR32, GR64>;
 274:   def PTI : SideEffectBinaryRRE<"pti", 0xB99E, GR64, GR64>;
 275: }
 276: 
 277: // Resume program.
 278: let hasSideEffects = 1, Defs = [CC] in
 279:   def RP : SideEffectAddressS<"rp", 0xB277, null_frag>;
 280: 
```
- **EN**: This block declares or refines TableGen records such as `LASP`, `PALB`, `PC`, `PR`, `PT`, `PTI`.
- **CN**: 该代码块声明或细化了 `LASP`, `PALB`, `PC`, `PR`, `PT`, `PTI` 等 TableGen 记录。

### Lines 281-308 / 第 281-308 行
```tablegen
 281: // Branch in subspace group.
 282: let hasSideEffects = 1 in
 283:   def BSG : UnaryRRE<"bsg", 0xB258, null_frag, GR64, GR64>;
 284: 
 285: // Branch and set authority.
 286: let hasSideEffects = 1 in
 287:   def BSA : UnaryRRE<"bsa", 0xB25A, null_frag, GR64, GR64>;
 288: 
 289: // Test access.
 290: let Defs = [CC] in
 291:   def TAR : SideEffectBinaryRRE<"tar", 0xB24C, AR32, GR32>;
 292: 
 293: //===----------------------------------------------------------------------===//
 294: // Linkage-Stack Instructions.
 295: //===----------------------------------------------------------------------===//
 296: 
 297: // Branch and stack.
 298: let hasSideEffects = 1 in
 299:   def BAKR : SideEffectBinaryRRE<"bakr", 0xB240, GR64, GR64>;
 300: 
 301: // Extract stacked registers.
 302: let hasSideEffects = 1 in {
 303:   def EREG : SideEffectBinaryRRE<"ereg", 0xB249, GR32, GR32>;
 304:   def EREGG : SideEffectBinaryRRE<"eregg", 0xB90E, GR64, GR64>;
 305: }
 306: 
 307: // Extract stacked state.
 308: let hasSideEffects = 1, Defs = [CC] in
```
- **EN**: This block declares or refines TableGen records such as `BSG`, `BSA`, `TAR`, `BAKR`, `EREG`, `EREGG`.
- **CN**: 该代码块声明或细化了 `BSG`, `BSA`, `TAR`, `BAKR`, `EREG`, `EREGG` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309:   def ESTA : UnaryRRE<"esta", 0xB24A, null_frag, GR128, GR32>;
 310: 
 311: // Modify stacked state.
 312: let hasSideEffects = 1 in
 313:   def MSTA : SideEffectUnaryRRE<"msta", 0xB247, GR128, null_frag>;
 314: 
 315: //===----------------------------------------------------------------------===//
 316: // Time-Related Instructions.
 317: //===----------------------------------------------------------------------===//
 318: 
 319: // Perform timing facility function.
 320: let hasSideEffects = 1, mayLoad = 1, Uses = [R0L, R1D], Defs = [CC] in
 321:   def PTFF : SideEffectInherentE<"ptff", 0x0104>;
 322: 
 323: // Set clock.
 324: let hasSideEffects = 1, Defs = [CC] in
 325:   def SCK : SideEffectUnaryS<"sck", 0xB204, null_frag, 8>;
 326: 
 327: // Set clock programmable field.
 328: let hasSideEffects = 1, Uses = [R0L] in
 329:   def SCKPF : SideEffectInherentE<"sckpf", 0x0107>;
 330: 
 331: // Set clock comparator.
 332: let hasSideEffects = 1 in
 333:   def SCKC : SideEffectUnaryS<"sckc", 0xB206, null_frag, 8>;
 334: 
 335: // Set CPU timer.
 336: let hasSideEffects = 1 in
```
- **EN**: This block declares or refines TableGen records such as `ESTA`, `MSTA`, `PTFF`, `SCK`, `SCKPF`, `SCKC`.
- **CN**: 该代码块声明或细化了 `ESTA`, `MSTA`, `PTFF`, `SCK`, `SCKPF`, `SCKC` 等 TableGen 记录。

### Lines 337-364 / 第 337-364 行
```tablegen
 337:   def SPT : SideEffectUnaryS<"spt", 0xB208, null_frag, 8>;
 338: 
 339: // Store clock (fast / extended).
 340: let hasSideEffects = 1, Defs = [CC] in {
 341:   def STCK  : StoreInherentS<"stck",  0xB205, null_frag, 8>;
 342:   def STCKF : StoreInherentS<"stckf", 0xB27C, z_stckf, 8>;
 343:   def STCKE : StoreInherentS<"stcke", 0xB278, null_frag, 16>;
 344: }
 345: 
 346: // Store clock comparator.
 347: let hasSideEffects = 1 in
 348:   def STCKC : StoreInherentS<"stckc", 0xB207, null_frag, 8>;
 349: 
 350: // Store CPU timer.
 351: let hasSideEffects = 1 in
 352:   def STPT : StoreInherentS<"stpt", 0xB209, null_frag, 8>;
 353: 
 354: //===----------------------------------------------------------------------===//
 355: // CPU-Related Instructions.
 356: //===----------------------------------------------------------------------===//
 357: 
 358: // Store CPU address.
 359: let hasSideEffects = 1 in
 360:   def STAP : StoreInherentS<"stap", 0xB212, null_frag, 2>;
 361: 
 362: // Store CPU ID.
 363: let hasSideEffects = 1 in
 364:   def STIDP : StoreInherentS<"stidp", 0xB202, null_frag, 8>;
```
- **EN**: This block declares or refines TableGen records such as `SPT`, `STCK`, `STCKF`, `STCKE`, `STCKC`, `STPT`.
- **CN**: 该代码块声明或细化了 `SPT`, `STCK`, `STCKF`, `STCKE`, `STCKC`, `STPT` 等 TableGen 记录。

### Lines 365-392 / 第 365-392 行
```tablegen
 365: 
 366: // Store system information.
 367: let hasSideEffects = 1, Uses = [R0L, R1L], Defs = [R0L, CC] in
 368:   def STSI : StoreInherentS<"stsi", 0xB27D, null_frag, 0>;
 369: 
 370: // Store facility list.
 371: let hasSideEffects = 1 in
 372:   def STFL : StoreInherentS<"stfl", 0xB2B1, null_frag, 4>;
 373: 
 374: // Store facility list extended.
 375: let hasSideEffects = 1, Uses = [R0D], Defs = [R0D, CC] in
 376:   def STFLE : StoreInherentS<"stfle", 0xB2B0, null_frag, 0>;
 377: 
 378: // Extract CPU attribute.
 379: let hasSideEffects = 1 in
 380:   def ECAG : BinaryRSY<"ecag", 0xEB4C, null_frag, GR64>;
 381: 
 382: // Extract CPU time.
 383: let hasSideEffects = 1, mayLoad = 1, Defs = [R0D, R1D] in
 384:   def ECTG : SideEffectTernarySSF<"ectg", 0xC81, GR64>;
 385: 
 386: // Perform topology function.
 387: let hasSideEffects = 1 in
 388:   def PTF : UnaryTiedRRE<"ptf", 0xB9A2, GR64>;
 389: 
 390: // Perform cryptographic key management operation.
 391: let Predicates = [FeatureMessageSecurityAssist3],
 392:     hasSideEffects = 1, Uses = [R0L, R1D] in
```
- **EN**: This block declares or refines TableGen records such as `STSI`, `STFL`, `STFLE`, `ECAG`, `ECTG`, `PTF`.
- **CN**: 该代码块声明或细化了 `STSI`, `STFL`, `STFLE`, `ECAG`, `ECTG`, `PTF` 等 TableGen 记录。

### Lines 393-420 / 第 393-420 行
```tablegen
 393:   def PCKMO : SideEffectInherentRRE<"pckmo", 0xB928>;
 394: 
 395: // Query processor activity counter information.
 396: let Predicates = [FeatureProcessorActivityInstrumentation],
 397:     hasSideEffects = 1, Uses = [R0D], Defs = [R0D, CC] in
 398:   def QPACI : StoreInherentS<"qpaci", 0xB28F, null_frag, 0>;
 399: 
 400: //===----------------------------------------------------------------------===//
 401: // Miscellaneous Instructions.
 402: //===----------------------------------------------------------------------===//
 403: 
 404: // Supervisor call.
 405: let hasSideEffects = 1, isCall = 1, Defs = [CC] in
 406:   def SVC : SideEffectUnaryI<"svc", 0x0A, imm32zx8>;
 407: 
 408: // Monitor call.
 409: let hasSideEffects = 1, isCall = 1 in
 410:   def MC : SideEffectBinarySI<"mc", 0xAF, imm32zx8>;
 411: 
 412: // Diagnose.
 413: let hasSideEffects = 1, isCall = 1 in
 414:   def DIAG : SideEffectTernaryRS<"diag", 0x83, GR32, GR32>;
 415: 
 416: // Trace.
 417: let hasSideEffects = 1, mayLoad = 1 in {
 418:   def TRACE : SideEffectTernaryRS<"trace", 0x99, GR32, GR32>;
 419:   def TRACG : SideEffectTernaryRSY<"tracg", 0xEB0F, GR64, GR64>;
 420: }
```
- **EN**: This block declares or refines TableGen records such as `PCKMO`, `QPACI`, `SVC`, `MC`, `DIAG`, `TRACE`.
- **CN**: 该代码块声明或细化了 `PCKMO`, `QPACI`, `SVC`, `MC`, `DIAG`, `TRACE` 等 TableGen 记录。

### Lines 421-448 / 第 421-448 行
```tablegen
 421: 
 422: // Trap.
 423: let hasSideEffects = 1 in {
 424:   def TRAP2 : SideEffectInherentE<"trap2", 0x01FF>;
 425:   def TRAP4 : SideEffectAddressS<"trap4", 0xB2FF, null_frag>;
 426: }
 427: 
 428: // Signal processor.
 429: let hasSideEffects = 1, Defs = [CC] in
 430:   def SIGP : SideEffectTernaryRS<"sigp", 0xAE, GR64, GR64>;
 431: 
 432: // Signal adapter.
 433: let hasSideEffects = 1, Uses = [R0D, R1D, R2D, R3D], Defs = [CC] in
 434:   def SIGA : SideEffectAddressS<"siga", 0xB274, null_frag>;
 435: 
 436: // Start interpretive execution.
 437: let hasSideEffects = 1, Defs = [CC] in
 438:   def SIE : SideEffectUnaryS<"sie", 0xB214, null_frag, 0>;
 439: 
 440: //===----------------------------------------------------------------------===//
 441: // CPU-Measurement Facility Instructions (SA23-2260).
 442: //===----------------------------------------------------------------------===//
 443: 
 444: // Load program parameter
 445: let hasSideEffects = 1 in
 446:   def LPP : SideEffectUnaryS<"lpp", 0xB280, null_frag, 8>;
 447: 
 448: // Extract coprocessor-group address.
```
- **EN**: This block declares or refines TableGen records such as `TRAP2`, `TRAP4`, `SIGP`, `SIGA`, `SIE`, `LPP`.
- **CN**: 该代码块声明或细化了 `TRAP2`, `TRAP4`, `SIGP`, `SIGA`, `SIE`, `LPP` 等 TableGen 记录。

### Lines 449-476 / 第 449-476 行
```tablegen
 449: let hasSideEffects = 1, Defs = [CC] in
 450:   def ECPGA : UnaryRRE<"ecpga", 0xB2ED, null_frag, GR32, GR64>;
 451: 
 452: // Extract CPU counter.
 453: let hasSideEffects = 1, Defs = [CC] in
 454:   def ECCTR : UnaryRRE<"ecctr", 0xB2E4, null_frag, GR64, GR64>;
 455: 
 456: // Extract peripheral counter.
 457: let hasSideEffects = 1, Defs = [CC] in
 458:   def EPCTR : UnaryRRE<"epctr", 0xB2E5, null_frag, GR64, GR64>;
 459: 
 460: // Load CPU-counter-set controls.
 461: let hasSideEffects = 1, Defs = [CC] in
 462:   def LCCTL : SideEffectUnaryS<"lcctl", 0xB284, null_frag, 8>;
 463: 
 464: // Load peripheral-counter-set controls.
 465: let hasSideEffects = 1, Defs = [CC] in
 466:   def LPCTL : SideEffectUnaryS<"lpctl", 0xB285, null_frag, 8>;
 467: 
 468: // Load sampling controls.
 469: let hasSideEffects = 1, Defs = [CC] in
 470:   def LSCTL : SideEffectUnaryS<"lsctl", 0xB287, null_frag, 0>;
 471: 
 472: // Query sampling information.
 473: let hasSideEffects = 1 in
 474:   def QSI : StoreInherentS<"qsi", 0xB286, null_frag, 0>;
 475: 
 476: // Query counter information.
```
- **EN**: This block declares or refines TableGen records such as `ECPGA`, `ECCTR`, `EPCTR`, `LCCTL`, `LPCTL`, `LSCTL`.
- **CN**: 该代码块声明或细化了 `ECPGA`, `ECCTR`, `EPCTR`, `LCCTL`, `LPCTL`, `LSCTL` 等 TableGen 记录。

### Lines 477-504 / 第 477-504 行
```tablegen
 477: let hasSideEffects = 1 in
 478:   def QCTRI : StoreInherentS<"qctri", 0xB28E, null_frag, 0>;
 479: 
 480: // Set CPU counter.
 481: let hasSideEffects = 1, Defs = [CC] in
 482:   def SCCTR : SideEffectBinaryRRE<"scctr", 0xB2E0, GR64, GR64>;
 483: 
 484: // Set peripheral counter.
 485: let hasSideEffects = 1, Defs = [CC] in
 486:   def SPCTR : SideEffectBinaryRRE<"spctr", 0xB2E1, GR64, GR64>;
 487: 
 488: //===----------------------------------------------------------------------===//
 489: // I/O Instructions (Principles of Operation, Chapter 14).
 490: //===----------------------------------------------------------------------===//
 491: 
 492: // Clear subchannel.
 493: let hasSideEffects = 1, Uses = [R1L], Defs = [CC] in
 494:   def CSCH : SideEffectInherentS<"csch", 0xB230, null_frag>;
 495: 
 496: // Halt subchannel.
 497: let hasSideEffects = 1, Uses = [R1L], Defs = [CC] in
 498:   def HSCH : SideEffectInherentS<"hsch", 0xB231, null_frag>;
 499: 
 500: // Modify subchannel.
 501: let hasSideEffects = 1, Uses = [R1L], Defs = [CC] in
 502:   def MSCH : SideEffectUnaryS<"msch", 0xB232, null_frag, 0>;
 503: 
 504: // Resume subchannel.
```
- **EN**: This block declares or refines TableGen records such as `QCTRI`, `SCCTR`, `SPCTR`, `CSCH`, `HSCH`, `MSCH`.
- **CN**: 该代码块声明或细化了 `QCTRI`, `SCCTR`, `SPCTR`, `CSCH`, `HSCH`, `MSCH` 等 TableGen 记录。

### Lines 505-532 / 第 505-532 行
```tablegen
 505: let hasSideEffects = 1, Uses = [R1L], Defs = [CC] in
 506:   def RSCH : SideEffectInherentS<"rsch", 0xB238, null_frag>;
 507: 
 508: // Start subchannel.
 509: let hasSideEffects = 1, Uses = [R1L], Defs = [CC] in
 510:   def SSCH : SideEffectUnaryS<"ssch", 0xB233, null_frag, 0>;
 511: 
 512: // Store subchannel.
 513: let hasSideEffects = 1, Uses = [R1L], Defs = [CC] in
 514:   def STSCH : StoreInherentS<"stsch", 0xB234, null_frag, 0>;
 515: 
 516: // Test subchannel.
 517: let hasSideEffects = 1, Uses = [R1L], Defs = [CC] in
 518:   def TSCH : StoreInherentS<"tsch", 0xB235, null_frag, 0>;
 519: 
 520: // Cancel subchannel.
 521: let hasSideEffects = 1, Uses = [R1L], Defs = [CC] in
 522:   def XSCH : SideEffectInherentS<"xsch", 0xB276, null_frag>;
 523: 
 524: // Reset channel path.
 525: let hasSideEffects = 1, Uses = [R1L], Defs = [CC] in
 526:   def RCHP : SideEffectInherentS<"rchp", 0xB23B, null_frag>;
 527: 
 528: // Set channel monitor.
 529: let hasSideEffects = 1, mayLoad = 1, Uses = [R1L, R2D] in
 530:   def SCHM : SideEffectInherentS<"schm", 0xB23C, null_frag>;
 531: 
 532: // Store channel path status.
```
- **EN**: This block declares or refines TableGen records such as `RSCH`, `SSCH`, `STSCH`, `TSCH`, `XSCH`, `RCHP`.
- **CN**: 该代码块声明或细化了 `RSCH`, `SSCH`, `STSCH`, `TSCH`, `XSCH`, `RCHP` 等 TableGen 记录。

### Lines 533-551 / 第 533-551 行
```tablegen
 533: let hasSideEffects = 1 in
 534:   def STCPS : StoreInherentS<"stcps", 0xB23A, null_frag, 0>;
 535: 
 536: // Store channel report word.
 537: let hasSideEffects = 1, Defs = [CC] in
 538:   def STCRW : StoreInherentS<"stcrw", 0xB239, null_frag, 0>;
 539: 
 540: // Test pending interruption.
 541: let hasSideEffects = 1, Defs = [CC] in
 542:   def TPI : StoreInherentS<"tpi", 0xB236, null_frag, 0>;
 543: 
 544: // Test pending external interruption.
 545: let hasSideEffects = 1, Defs = [CC], Predicates = [FeatureTestPendingExternalInterruption] in
 546:   def TPEI : UnaryRRE<"tpei", 0xB9A1, null_frag, GR64, GR64>;
 547: 
 548: // Set address limit.
 549: let hasSideEffects = 1, Uses = [R1L] in
 550:   def SAL : SideEffectInherentS<"sal", 0xB237, null_frag>;
 551: 
```
- **EN**: This block declares or refines TableGen records such as `STCPS`, `STCRW`, `TPI`, `TPEI`, `SAL`.
- **CN**: 该代码块声明或细化了 `STCPS`, `STCRW`, `TPI`, `TPEI`, `SAL` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
