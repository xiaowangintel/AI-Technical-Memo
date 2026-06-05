# SystemZRegisterInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZRegisterInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines SystemZ backend metadata using LLVM TableGen DSL; specifically it defines or implements target register information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 SystemZ 后端元数据；具体而言，它定义或实现目标寄存器信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //==- SystemZRegisterInfo.td - SystemZ register definitions -*- tablegen -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: // Class definitions.
  11: //===----------------------------------------------------------------------===//
  12: 
  13: class SystemZReg<string n> : Register<n> {
  14:   let Namespace = "SystemZ";
  15: }
  16: 
  17: class SystemZRegWithSubregs<string n, list<Register> subregs>
  18:   : RegisterWithSubRegs<n, subregs> {
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `SystemZReg`, `SystemZRegWithSubregs`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `SystemZReg`, `SystemZRegWithSubregs` 等 TableGen 记录。

### Lines 19-36 / 第 19-36 行
```tablegen
  19:   let Namespace = "SystemZ";
  20: }
  21: 
  22: let Namespace = "SystemZ" in {
  23: def subreg_h16   : SubRegIndex<16, 16>;
  24: def subreg_l32   : SubRegIndex<32, 0>;  // Also acts as subreg_hl32.
  25: def subreg_h32   : SubRegIndex<32, 32>; // Also acts as subreg_hh32.
  26: def subreg_l64   : SubRegIndex<64, 0>;
  27: def subreg_h64   : SubRegIndex<64, 64>;
  28: def subreg_lh32  : ComposedSubRegIndex<subreg_l64, subreg_h32>;
  29: def subreg_ll32  : ComposedSubRegIndex<subreg_l64, subreg_l32>;
  30: }
  31: 
  32: // Define a register class that contains values of types TYPES and an
  33: // associated operand called NAME.  SIZE is the size and alignment
  34: // of the registers and REGLIST is the list of individual registers.
  35: // If the user provides an alternate order list of regs, it will be used for
  36: // XPLINK. Otherwise, by default, XPLINK will use the regList ordering as well
```
- **EN**: This block declares or refines TableGen records such as `subreg_h16`, `subreg_l32`, `subreg_h32`, `subreg_l64`, `subreg_h64`, `subreg_lh32`.
- **CN**: 该代码块声明或细化了 `subreg_h16`, `subreg_l32`, `subreg_h32`, `subreg_l64`, `subreg_h64`, `subreg_lh32` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37: multiclass SystemZRegClass<string name, list<ValueType> types, int size,
  38:                            dag regList, list<dag> altRegList = [regList],
  39:                            bit allocatable = 1,
  40:                            RegInfoByHwMode RI = RegInfoByHwMode<[], []>> {
  41:   def AsmOperand : AsmOperandClass {
  42:     let Name = name;
  43:     let ParserMethod = "parse"#name;
  44:     let RenderMethod = "addRegOperands";
  45:   }
  46:   let isAllocatable = allocatable in
  47:     def Bit : RegisterClass<"SystemZ", types, size, regList> {
  48:       let Size = size;
  49:       let AltOrders = altRegList;
  50:       let AltOrderSelect = [{
  51:         const SystemZSubtarget &S = MF.getSubtarget<SystemZSubtarget>();
  52:         return S.isTargetXPLINK64();
  53:       }];
  54:       let RegInfos = RI;
```
- **EN**: This block declares or refines TableGen records such as `SystemZRegClass`, `AsmOperand`, `Bit`.
- **CN**: 该代码块声明或细化了 `SystemZRegClass`, `AsmOperand`, `Bit` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55:     }
  56:   def "" : RegisterOperand<!cast<RegisterClass>(name#"Bit")> {
  57:     let ParserMatchClass = !cast<AsmOperandClass>(name#"AsmOperand");
  58:   }
  59: }
  60: 
  61: //===----------------------------------------------------------------------===//
  62: // General-purpose registers
  63: //===----------------------------------------------------------------------===//
  64: 
  65: // Lower 32 bits of one of the 16 64-bit general-purpose registers
  66: class GPR32<bits<16> num, string n> : SystemZReg<n> {
  67:   let HWEncoding = num;
  68: }
  69: 
  70: // One of the 16 64-bit general-purpose registers.
  71: class GPR64<bits<16> num, string n, GPR32 low, GPR32 high>
  72:  : SystemZRegWithSubregs<n, [low, high]> {
```
- **EN**: This block declares or refines TableGen records such as `GPR32`, `GPR64`.
- **CN**: 该代码块声明或细化了 `GPR32`, `GPR64` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73:   let HWEncoding = num;
  74:   let SubRegIndices = [subreg_l32, subreg_h32];
  75:   let CoveredBySubRegs = 1;
  76: }
  77: 
  78: // 8 even-odd pairs of GPR64s.
  79: class GPR128<bits<16> num, string n, GPR64 low, GPR64 high>
  80:  : SystemZRegWithSubregs<n, [high, low]> {
  81:   let HWEncoding = num;
  82:   let SubRegIndices = [subreg_h64, subreg_l64];
  83:   let CoveredBySubRegs = 1;
  84: }
  85: 
  86: // General-purpose registers
  87: foreach I = 0-15 in {
  88:   def R#I#L : GPR32<I, "r"#I>;
  89:   def R#I#H : GPR32<I, "r"#I>;
  90:   def R#I#D : GPR64<I, "r"#I, !cast<GPR32>("R"#I#"L"), !cast<GPR32>("R"#I#"H")>,
```
- **EN**: This block declares or refines TableGen records such as `GPR128`, `R`, `R`, `R`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `GPR128`, `R`, `R`, `R` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 91-108 / 第 91-108 行
```tablegen
  91:                     DwarfRegNum<[I]>;
  92: }
  93: 
  94: foreach I = [0, 2, 4, 6, 8, 10, 12, 14] in {
  95:   def R#I#Q : GPR128<I, "r"#I, !cast<GPR64>("R"#!add(I, 1)#"D"),
  96:                      !cast<GPR64>("R"#I#"D")>;
  97: }
  98: 
  99: /// zLinux: Allocate the callee-saved R6-R13 backwards. That way they can be
 100: /// saved together with R14 and R15 in one prolog instruction.
 101: /// XPLINK64: Allocate all registers in natural order
 102: defm GR32  : SystemZRegClass<"GR32",  [i32], 32,
 103:                              (add (sequence "R%uL",  0, 5),
 104:                                   (sequence "R%uL", 15, 6)),
 105:                              [(add (sequence "R%uL", 0, 15))]>;
 106: defm GRH32 : SystemZRegClass<"GRH32", [i32], 32,
 107:                              (add (sequence "R%uH",  0, 5),
 108:                                   (sequence "R%uH", 15, 6)),
```
- **EN**: This block declares or refines TableGen records such as `R`, `GR32`, `GRH32`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `R`, `GR32`, `GRH32` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 109-126 / 第 109-126 行
```tablegen
 109:                              [(add (sequence "R%uH", 0, 15))]>;
 110: defm GR64  : SystemZRegClass<"GR64",  [i64], 64,
 111:                              (add (sequence "R%uD",  0, 5),
 112:                                   (sequence "R%uD", 15, 6)),
 113:                              [(add (sequence "R%uD",  0, 15))]>;
 114: 
 115: // Combine the low and high GR32s into a single class.  This can only be
 116: // used for virtual registers if the high-word facility is available.
 117: /// XPLINK64: Allocate all registers in natural order
 118: defm GRX32 : SystemZRegClass<"GRX32", [i32], 32,
 119:                              (add (sequence "R%uL",  0, 5),
 120:                                   (sequence "R%uH",  0, 5),
 121:                                   R15L, R15H, R14L, R14H, R13L, R13H,
 122:                                   R12L, R12H, R11L, R11H, R10L, R10H,
 123:                                   R9L, R9H, R8L, R8H, R7L, R7H, R6L, R6H),
 124:                              [(add
 125:                                R0L, R1L, R2L, R3L, R0H, R1H, R2H, R3H,
 126:                                R4L, R4H, R5L, R5H, R6L, R6H, R7L, R7H,
```
- **EN**: This block declares or refines TableGen records such as `GR64`, `GRX32`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `GR64`, `GRX32` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```tablegen
 127:                                R8L, R8H, R9L, R9H, R10L,R10H,R11L,R11H,
 128:                                R12L,R12H,R13L,R13H,R14L,R14H,R15L,R15H)
 129:                              ]>;
 130: 
 131: // On machines without SIMD support, i128 is not a legal type, so model the
 132: // register pairs as untyped instead.
 133: // XPLINK64: Allocate all registers in natural order
 134: defm GR128 : SystemZRegClass<"GR128", [untyped], 128,
 135:                              (add R0Q, R2Q, R4Q, R12Q, R10Q, R8Q, R6Q, R14Q),
 136:                              [(add R0Q, R2Q, R4Q, R6Q, R8Q, R10Q, R12Q, R14Q)]>;
 137: 
 138: // Base and index registers.  Everything except R0, which in an address
 139: // context evaluates as 0.
 140: // XPLINK64: Allocate all registers in natural order
 141: defm ADDR32 : SystemZRegClass<"ADDR32", [i32], 32, (sub GR32Bit, R0L),
 142:                               [(add (sequence "R%uL",  1, 15))]>;
 143: defm ADDR64 : SystemZRegClass<"ADDR64", [i64], 64, (sub GR64Bit, R0D),
 144:                               [(add (sequence "R%uD",  1, 15))]>;
```
- **EN**: This block declares or refines TableGen records such as `GR128`, `ADDR32`, `ADDR64`.
- **CN**: 该代码块声明或细化了 `GR128`, `ADDR32`, `ADDR64` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: 
 146: // Not used directly, but needs to exist for ADDR32 and ADDR64 subregs
 147: // of a GR128.
 148: // XPLINK64: Allocate all registers in natural order
 149: defm ADDR128 : SystemZRegClass<"ADDR128", [untyped], 128, (sub GR128Bit, R0Q),
 150:                                [(add R2Q, R4Q, R6Q, R8Q, R10Q, R12Q, R14Q)]>;
 151: 
 152: // Any type register. Used for .insn directives when we don't know what the
 153: // register types could be.
 154: defm AnyReg : SystemZRegClass<"AnyReg",
 155:                               [i64, f64, v8i8, v4i16, v2i32, v2f32], 64,
 156:                               (add (sequence "R%uD", 0, 15),
 157:                                    (sequence "F%uD", 0, 15),
 158:                                    (sequence "V%u", 0, 15)),
 159:                               [], 0/*allocatable*/>;
 160: 
 161: //===----------------------------------------------------------------------===//
 162: // Floating-point registers
```
- **EN**: This block declares or refines TableGen records such as `ADDR128`, `AnyReg`.
- **CN**: 该代码块声明或细化了 `ADDR128`, `AnyReg` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163: //===----------------------------------------------------------------------===//
 164: 
 165: // Maps FPR register numbers to their DWARF encoding.
 166: class DwarfMapping<int id> { int Id = id; }
 167: 
 168: def F0Dwarf  : DwarfMapping<16>;
 169: def F2Dwarf  : DwarfMapping<17>;
 170: def F4Dwarf  : DwarfMapping<18>;
 171: def F6Dwarf  : DwarfMapping<19>;
 172: 
 173: def F1Dwarf  : DwarfMapping<20>;
 174: def F3Dwarf  : DwarfMapping<21>;
 175: def F5Dwarf  : DwarfMapping<22>;
 176: def F7Dwarf  : DwarfMapping<23>;
 177: 
 178: def F8Dwarf  : DwarfMapping<24>;
 179: def F10Dwarf : DwarfMapping<25>;
 180: def F12Dwarf : DwarfMapping<26>;
```
- **EN**: This block declares or refines TableGen records such as `DwarfMapping`, `F0Dwarf`, `F2Dwarf`, `F4Dwarf`, `F6Dwarf`, `F1Dwarf`.
- **CN**: 该代码块声明或细化了 `DwarfMapping`, `F0Dwarf`, `F2Dwarf`, `F4Dwarf`, `F6Dwarf`, `F1Dwarf` 等 TableGen 记录。

### Lines 181-198 / 第 181-198 行
```tablegen
 181: def F14Dwarf : DwarfMapping<27>;
 182: 
 183: def F9Dwarf  : DwarfMapping<28>;
 184: def F11Dwarf : DwarfMapping<29>;
 185: def F13Dwarf : DwarfMapping<30>;
 186: def F15Dwarf : DwarfMapping<31>;
 187: 
 188: def F16Dwarf : DwarfMapping<68>;
 189: def F18Dwarf : DwarfMapping<69>;
 190: def F20Dwarf : DwarfMapping<70>;
 191: def F22Dwarf : DwarfMapping<71>;
 192: 
 193: def F17Dwarf : DwarfMapping<72>;
 194: def F19Dwarf : DwarfMapping<73>;
 195: def F21Dwarf : DwarfMapping<74>;
 196: def F23Dwarf : DwarfMapping<75>;
 197: 
 198: def F24Dwarf : DwarfMapping<76>;
```
- **EN**: This block declares or refines TableGen records such as `F14Dwarf`, `F9Dwarf`, `F11Dwarf`, `F13Dwarf`, `F15Dwarf`, `F16Dwarf`.
- **CN**: 该代码块声明或细化了 `F14Dwarf`, `F9Dwarf`, `F11Dwarf`, `F13Dwarf`, `F15Dwarf`, `F16Dwarf` 等 TableGen 记录。

### Lines 199-216 / 第 199-216 行
```tablegen
 199: def F26Dwarf : DwarfMapping<77>;
 200: def F28Dwarf : DwarfMapping<78>;
 201: def F30Dwarf : DwarfMapping<79>;
 202: 
 203: def F25Dwarf : DwarfMapping<80>;
 204: def F27Dwarf : DwarfMapping<81>;
 205: def F29Dwarf : DwarfMapping<82>;
 206: def F31Dwarf : DwarfMapping<83>;
 207: 
 208: // Upper 16 bits of one of the floating-point registers
 209: class FPR16<bits<16> num, string n> : SystemZReg<n> {
 210:   let HWEncoding = num;
 211: }
 212: 
 213: // Upper 32 bits of one of the floating-point registers
 214: class FPR32<bits<16> num, string n, FPR16 high>
 215:   : SystemZRegWithSubregs<n, [high]> {
 216:   let HWEncoding = num;
```
- **EN**: This block declares or refines TableGen records such as `F26Dwarf`, `F28Dwarf`, `F30Dwarf`, `F25Dwarf`, `F27Dwarf`, `F29Dwarf`.
- **CN**: 该代码块声明或细化了 `F26Dwarf`, `F28Dwarf`, `F30Dwarf`, `F25Dwarf`, `F27Dwarf`, `F29Dwarf` 等 TableGen 记录。

### Lines 217-234 / 第 217-234 行
```tablegen
 217:   let SubRegIndices = [subreg_h16];
 218: }
 219: 
 220: // One of the floating-point registers.
 221: class FPR64<bits<16> num, string n, FPR32 high>
 222:  : SystemZRegWithSubregs<n, [high]> {
 223:   let HWEncoding = num;
 224:   let SubRegIndices = [subreg_h32];
 225: }
 226: 
 227: // 8 pairs of FPR64s, with a one-register gap inbetween.
 228: class FPR128<bits<16> num, string n, FPR64 low, FPR64 high>
 229:  : SystemZRegWithSubregs<n, [high, low]> {
 230:   let HWEncoding = num;
 231:   let SubRegIndices = [subreg_h64, subreg_l64];
 232:   let CoveredBySubRegs = 1;
 233: }
 234: 
```
- **EN**: This block declares or refines TableGen records such as `FPR64`, `FPR128`.
- **CN**: 该代码块声明或细化了 `FPR64`, `FPR128` 等 TableGen 记录。

### Lines 235-252 / 第 235-252 行
```tablegen
 235: // Floating-point registers.  Registers 16-31 require the vector facility.
 236: foreach I = 0-15 in {
 237:   def F#I#H : FPR16<I, "f"#I>;
 238:   def F#I#S : FPR32<I, "f"#I, !cast<FPR16>("F"#I#"H")>;
 239:   def F#I#D : FPR64<I, "f"#I, !cast<FPR32>("F"#I#"S")>,
 240:               DwarfRegNum<[!cast<DwarfMapping>("F"#I#"Dwarf").Id]>;
 241: }
 242: foreach I = 16-31 in {
 243:   def F#I#H : FPR16<I, "v"#I>;
 244:   def F#I#S : FPR32<I, "v"#I, !cast<FPR16>("F"#I#"H")>;
 245:   def F#I#D : FPR64<I, "v"#I, !cast<FPR32>("F"#I#"S")>,
 246:               DwarfRegNum<[!cast<DwarfMapping>("F"#I#"Dwarf").Id]>;
 247: }
 248: 
 249: foreach I = [0, 1, 4, 5, 8, 9, 12, 13] in {
 250:   def F#I#Q  : FPR128<I, "f"#I, !cast<FPR64>("F"#!add(I, 2)#"D"),
 251:                      !cast<FPR64>("F"#I#"D")>;
 252: }
```
- **EN**: This block declares or refines TableGen records such as `F`, `F`, `F`, `F`, `F`, `F`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `F`, `F`, `F`, `F`, `F`, `F` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 253-270 / 第 253-270 行
```tablegen
 253: 
 254: // There's no store-multiple instruction for FPRs, so we're not fussy
 255: // about the order in which call-saved registers are allocated.
 256: // Adjust the spill size of f16 to 32 bits in case of no vector support.
 257: def FP16RI : RegInfoByHwMode<[DefaultMode,       NoVecHwMode],
 258:                              [RegInfo<16,16,16>, RegInfo<16,32,32>]>;
 259: defm FP16  : SystemZRegClass<"FP16", [f16], 16, (sequence "F%uH", 0, 15),
 260:                              [(sequence "F%uH", 0, 15)], 1, FP16RI>;
 261: defm FP32  : SystemZRegClass<"FP32", [f32], 32, (sequence "F%uS", 0, 15)>;
 262: defm FP64  : SystemZRegClass<"FP64", [f64], 64, (sequence "F%uD", 0, 15)>;
 263: defm FP128 : SystemZRegClass<"FP128", [f128], 128,
 264:                              (add F0Q, F1Q, F4Q, F5Q, F8Q, F9Q, F12Q, F13Q)>;
 265: 
 266: //===----------------------------------------------------------------------===//
 267: // Vector registers
 268: //===----------------------------------------------------------------------===//
 269: 
 270: // A full 128-bit vector register, with an FPR64 as its high part.
```
- **EN**: This block declares or refines TableGen records such as `FP16RI`, `FP16`, `FP32`, `FP64`, `FP128`.
- **CN**: 该代码块声明或细化了 `FP16RI`, `FP16`, `FP32`, `FP64`, `FP128` 等 TableGen 记录。

### Lines 271-288 / 第 271-288 行
```tablegen
 271: class VR128<bits<16> num, string n, FPR64 high>
 272:   : SystemZRegWithSubregs<n, [high]> {
 273:   let HWEncoding = num;
 274:   let SubRegIndices = [subreg_h64];
 275: }
 276: 
 277: // Full vector registers.
 278: foreach I = 0-31 in {
 279:   def V#I : VR128<I, "v"#I, !cast<FPR64>("F"#I#"D")>,
 280:             DwarfRegNum<[!cast<DwarfMapping>("F"#I#"Dwarf").Id]>;
 281: }
 282: 
 283: // Class used to store 16-bit fp values in the first element of a vector
 284: // register.
 285: defm VR16 : SystemZRegClass<"VR16", [f16], 16,
 286:                             (add (sequence "F%uH", 0, 7),
 287:                                  (sequence "F%uH", 16, 31),
 288:                                  (sequence "F%uH", 8, 15))>;
```
- **EN**: This block declares or refines TableGen records such as `VR128`, `V`, `VR16`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `VR128`, `V`, `VR16` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 289-306 / 第 289-306 行
```tablegen
 289: 
 290: // Class used to store 32-bit values in the first element of a vector
 291: // register.  f32 scalars are used for the WLEDB and WLDEB instructions.
 292: defm VR32 : SystemZRegClass<"VR32", [f32, v4i8, v2i16], 32,
 293:                             (add (sequence "F%uS", 0, 7),
 294:                                  (sequence "F%uS", 16, 31),
 295:                                  (sequence "F%uS", 8, 15))>;
 296: 
 297: // Class used to store 64-bit values in the upper half of a vector register.
 298: // The vector facility also includes scalar f64 instructions that operate
 299: // on the full vector register set.
 300: defm VR64 : SystemZRegClass<"VR64", [f64, v8i8, v4i16, v2i32, v2f32], 64,
 301:                             (add (sequence "F%uD", 0, 7),
 302:                                  (sequence "F%uD", 16, 31),
 303:                                  (sequence "F%uD", 8, 15))>;
 304: 
 305: // The subset of vector registers that can be used for floating-point
 306: // operations too.
```
- **EN**: This block declares or refines TableGen records such as `VR32`, `VR64`.
- **CN**: 该代码块声明或细化了 `VR32`, `VR64` 等 TableGen 记录。

### Lines 307-324 / 第 307-324 行
```tablegen
 307: defm VF128 : SystemZRegClass<"VF128",
 308:                              [v16i8, v8i16, v4i32, v2i64, v8f16, v4f32, v2f64],
 309:                              128, (sequence "V%u", 0, 15)>;
 310: 
 311: // All vector registers.
 312: defm VR128 : SystemZRegClass<"VR128",
 313:                              [v16i8, v8i16, v4i32, v2i64, i128,
 314:                               v8f16, v4f32, v2f64, f128],
 315:                              128, (add (sequence "V%u", 0, 7),
 316:                                        (sequence "V%u", 16, 31),
 317:                                        (sequence "V%u", 8, 15))>;
 318: 
 319: // Attaches a ValueType to a register operand, to make the instruction
 320: // definitions easier.
 321: class TypedReg<ValueType vtin, RegisterOperand opin> {
 322:   ValueType vt = vtin;
 323:   RegisterOperand op = opin;
 324: }
```
- **EN**: This block declares or refines TableGen records such as `VF128`, `VR128`, `TypedReg`.
- **CN**: 该代码块声明或细化了 `VF128`, `VR128`, `TypedReg` 等 TableGen 记录。

### Lines 325-342 / 第 325-342 行
```tablegen
 325: 
 326: def v16hb   : TypedReg<f16,     VR16>;
 327: def v32f    : TypedReg<i32,     VR32>;
 328: def v32sb   : TypedReg<f32,     VR32>;
 329: def v64g    : TypedReg<i64,     VR64>;
 330: def v64db   : TypedReg<f64,     VR64>;
 331: def v128b   : TypedReg<v16i8,   VR128>;
 332: def v128h   : TypedReg<v8i16,   VR128>;
 333: def v128f   : TypedReg<v4i32,   VR128>;
 334: def v128g   : TypedReg<v2i64,   VR128>;
 335: def v128q   : TypedReg<i128,    VR128>;
 336: def v128sb  : TypedReg<v4f32,   VR128>;
 337: def v128db  : TypedReg<v2f64,   VR128>;
 338: def v128xb  : TypedReg<f128,    VR128>;
 339: def v128any : TypedReg<untyped, VR128>;
 340: 
 341: //===----------------------------------------------------------------------===//
 342: // Other registers
```
- **EN**: This block declares or refines TableGen records such as `v16hb`, `v32f`, `v32sb`, `v64g`, `v64db`, `v128b`.
- **CN**: 该代码块声明或细化了 `v16hb`, `v32f`, `v32sb`, `v64g`, `v64db`, `v128b` 等 TableGen 记录。

### Lines 343-360 / 第 343-360 行
```tablegen
 343: //===----------------------------------------------------------------------===//
 344: 
 345: // The 2-bit condition code field of the PSW.  Every register named in an
 346: // inline asm needs a class associated with it.
 347: def CC : SystemZReg<"cc">;
 348: let isAllocatable = 0, CopyCost = -1 in
 349:   def CCR : RegisterClass<"SystemZ", [i32], 32, (add CC)>;
 350: 
 351: // The floating-point control register.
 352: // Note: We only model the current rounding modes and the IEEE masks.
 353: // IEEE flags and DXC are not modeled here.
 354: def FPC : SystemZReg<"fpc">;
 355: let isAllocatable = 0 in
 356:   def FPCRegs : RegisterClass<"SystemZ", [i32], 32, (add FPC)>;
 357: 
 358: // Access registers.
 359: class ACR32<bits<16> num, string n> : SystemZReg<n> {
 360:   let HWEncoding = num;
```
- **EN**: This block declares or refines TableGen records such as `CC`, `CCR`, `FPC`, `FPCRegs`, `ACR32`.
- **CN**: 该代码块声明或细化了 `CC`, `CCR`, `FPC`, `FPCRegs`, `ACR32` 等 TableGen 记录。

### Lines 361-376 / 第 361-376 行
```tablegen
 361: }
 362: foreach I = 0-15 in {
 363:   def A#I : ACR32<I, "a"#I>, DwarfRegNum<[!add(I, 48)]>;
 364: }
 365: defm AR32 : SystemZRegClass<"AR32", [i32], 32,
 366:                             (add (sequence "A%u", 0, 15)), [], 0>;
 367: 
 368: // Control registers.
 369: class CREG64<bits<16> num, string n> : SystemZReg<n> {
 370:   let HWEncoding = num;
 371: }
 372: foreach I = 0-15 in {
 373:   def C#I : CREG64<I, "c"#I>, DwarfRegNum<[!add(I, 32)]>;
 374: }
 375: defm CR64 : SystemZRegClass<"CR64", [i64], 64,
 376:                             (add (sequence "C%u", 0, 15)), [], 0>;
```
- **EN**: This block declares or refines TableGen records such as `A`, `AR32`, `CREG64`, `C`, `CR64`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `A`, `AR32`, `CREG64`, `C`, `CR64` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
