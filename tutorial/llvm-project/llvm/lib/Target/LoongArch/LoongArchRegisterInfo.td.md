# LoongArchRegisterInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchRegisterInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target register information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 LoongArch 后端元数据；具体而言，它定义或实现目标寄存器信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //===-- LoongArchRegisterInfo.td - LoongArch Register defs -*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: //  Declarations that describe the LoongArch register files
  11: //===----------------------------------------------------------------------===//
  12: 
  13: let Namespace = "LoongArch" in {
  14: class LoongArchReg<bits<16> Enc, string n, list<string> alt = []>
  15:     : Register<n> {
  16:   let HWEncoding = Enc;
  17:   let AltNames = alt;
  18: }
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `LoongArchReg`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `LoongArchReg` 等 TableGen 记录。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: 
  20: class LoongArchRegWithSubRegs<bits<16> Enc, string n, list<Register> subregs,
  21:                               list<string> alt = []>
  22:     : RegisterWithSubRegs<n, subregs> {
  23:   let HWEncoding = Enc;
  24:   let AltNames = alt;
  25: }
  26: 
  27: class LoongArchReg32<bits<16> Enc, string n, list<string> alt = []>
  28:     : Register<n> {
  29:   let HWEncoding = Enc;
  30:   let AltNames = alt;
  31: }
  32: 
  33: def sub_32 : SubRegIndex<32>;
  34: class LoongArchReg64<LoongArchReg32 subreg>
  35:     : LoongArchRegWithSubRegs<subreg.HWEncoding, subreg.AsmName, [subreg],
  36:                               subreg.AltNames> {
```
- **EN**: This block declares or refines TableGen records such as `LoongArchRegWithSubRegs`, `LoongArchReg32`, `sub_32`, `LoongArchReg64`.
- **CN**: 该代码块声明或细化了 `LoongArchRegWithSubRegs`, `LoongArchReg32`, `sub_32`, `LoongArchReg64` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37:   let SubRegIndices = [sub_32];
  38: }
  39: 
  40: def sub_64 : SubRegIndex<64>;
  41: class LoongArchReg128<LoongArchReg64 subreg, string n>
  42:     : LoongArchRegWithSubRegs<subreg.HWEncoding, n, [subreg]> {
  43:   let SubRegIndices = [sub_64];
  44: }
  45: 
  46: def sub_128 : SubRegIndex<128>;
  47: class LoongArchReg256<LoongArchReg128 subreg, string n>
  48:     : LoongArchRegWithSubRegs<subreg.HWEncoding, n, [subreg]> {
  49:   let SubRegIndices = [sub_128];
  50: }
  51: 
  52: let FallbackRegAltNameIndex = NoRegAltName in
  53: def RegAliasName : RegAltNameIndex;
  54: } // Namespace = "LoongArch"
```
- **EN**: This block declares or refines TableGen records such as `sub_64`, `LoongArchReg128`, `sub_128`, `LoongArchReg256`, `RegAliasName`.
- **CN**: 该代码块声明或细化了 `sub_64`, `LoongArchReg128`, `sub_128`, `LoongArchReg256`, `RegAliasName` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: 
  56: // Integer registers
  57: 
  58: let RegAltNameIndices = [RegAliasName] in {
  59:   let isConstant = true in
  60:   def R0  : LoongArchReg<0,  "r0", ["zero"]>, DwarfRegNum<[0]>;
  61:   def R1  : LoongArchReg<1,  "r1", ["ra"]>, DwarfRegNum<[1]>;
  62:   def R2  : LoongArchReg<2,  "r2", ["tp"]>, DwarfRegNum<[2]>;
  63:   def R3  : LoongArchReg<3,  "r3", ["sp"]>, DwarfRegNum<[3]>;
  64:   def R4  : LoongArchReg<4,  "r4", ["a0"]>, DwarfRegNum<[4]>;
  65:   def R5  : LoongArchReg<5,  "r5", ["a1"]>, DwarfRegNum<[5]>;
  66:   def R6  : LoongArchReg<6,  "r6", ["a2"]>, DwarfRegNum<[6]>;
  67:   def R7  : LoongArchReg<7,  "r7", ["a3"]>, DwarfRegNum<[7]>;
  68:   def R8  : LoongArchReg<8,  "r8", ["a4"]>, DwarfRegNum<[8]>;
  69:   def R9  : LoongArchReg<9,  "r9", ["a5"]>, DwarfRegNum<[9]>;
  70:   def R10 : LoongArchReg<10, "r10", ["a6"]>, DwarfRegNum<[10]>;
  71:   def R11 : LoongArchReg<11, "r11", ["a7"]>, DwarfRegNum<[11]>;
  72:   def R12 : LoongArchReg<12, "r12", ["t0"]>, DwarfRegNum<[12]>;
```
- **EN**: This block declares or refines TableGen records such as `R0`, `R1`, `R2`, `R3`, `R4`, `R5`.
- **CN**: 该代码块声明或细化了 `R0`, `R1`, `R2`, `R3`, `R4`, `R5` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73:   def R13 : LoongArchReg<13, "r13", ["t1"]>, DwarfRegNum<[13]>;
  74:   def R14 : LoongArchReg<14, "r14", ["t2"]>, DwarfRegNum<[14]>;
  75:   def R15 : LoongArchReg<15, "r15", ["t3"]>, DwarfRegNum<[15]>;
  76:   def R16 : LoongArchReg<16, "r16", ["t4"]>, DwarfRegNum<[16]>;
  77:   def R17 : LoongArchReg<17, "r17", ["t5"]>, DwarfRegNum<[17]>;
  78:   def R18 : LoongArchReg<18, "r18", ["t6"]>, DwarfRegNum<[18]>;
  79:   def R19 : LoongArchReg<19, "r19", ["t7"]>, DwarfRegNum<[19]>;
  80:   def R20 : LoongArchReg<20, "r20", ["t8"]>, DwarfRegNum<[20]>;
  81:   def R21 : LoongArchReg<21, "r21", [""]>, DwarfRegNum<[21]>;
  82:   def R22 : LoongArchReg<22, "r22", ["fp", "s9"]>, DwarfRegNum<[22]>;
  83:   def R23 : LoongArchReg<23, "r23", ["s0"]>, DwarfRegNum<[23]>;
  84:   def R24 : LoongArchReg<24, "r24", ["s1"]>, DwarfRegNum<[24]>;
  85:   def R25 : LoongArchReg<25, "r25", ["s2"]>, DwarfRegNum<[25]>;
  86:   def R26 : LoongArchReg<26, "r26", ["s3"]>, DwarfRegNum<[26]>;
  87:   def R27 : LoongArchReg<27, "r27", ["s4"]>, DwarfRegNum<[27]>;
  88:   def R28 : LoongArchReg<28, "r28", ["s5"]>, DwarfRegNum<[28]>;
  89:   def R29 : LoongArchReg<29, "r29", ["s6"]>, DwarfRegNum<[29]>;
  90:   def R30 : LoongArchReg<30, "r30", ["s7"]>, DwarfRegNum<[30]>;
```
- **EN**: This block declares or refines TableGen records such as `R13`, `R14`, `R15`, `R16`, `R17`, `R18`.
- **CN**: 该代码块声明或细化了 `R13`, `R14`, `R15`, `R16`, `R17`, `R18` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91:   def R31 : LoongArchReg<31, "r31", ["s8"]>, DwarfRegNum<[31]>;
  92: } // RegAltNameIndices = [RegAliasName]
  93: 
  94: def GRLenVT : ValueTypeByHwMode<[LA32, LA64],
  95:                                 [i32,  i64]>;
  96: def GRLenRI : RegInfoByHwMode<
  97:       [LA32,              LA64],
  98:       [RegInfo<32,32,32>, RegInfo<64,64,64>]>;
  99: 
 100: class GPRRegisterClass<dag regList>
 101:     : RegisterClass<"LoongArch", [GRLenVT], 32, regList> {
 102:   let RegInfos = GRLenRI;
 103: }
 104: 
 105: // The order of registers represents the preferred allocation sequence.
 106: // Registers are listed in the order caller-save, callee-save, specials.
 107: def GPR : GPRRegisterClass<(add // Argument registers (a0...a7)
 108:                                 (sequence "R%u", 4, 11),
```
- **EN**: This block declares or refines TableGen records such as `R31`, `GRLenVT`, `GRLenRI`, `GPRRegisterClass`, `GPR`.
- **CN**: 该代码块声明或细化了 `R31`, `GRLenVT`, `GRLenRI`, `GPRRegisterClass`, `GPR` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109:                                 // Temporary registers (t0...t8)
 110:                                 (sequence "R%u", 12, 20),
 111:                                 // Static register (s9/fp, s0...s8)
 112:                                 (sequence "R%u", 22, 31),
 113:                                 // Specials (r0, ra, tp, sp)
 114:                                 (sequence "R%u", 0, 3),
 115:                                 // Reserved (Non-allocatable)
 116:                                 R21)>;
 117: 
 118: // GPR for indirect tail calls. We can't use callee-saved registers, as they are
 119: // restored to the saved value before the tail call, which would clobber a call
 120: // address.
 121: def GPRT : GPRRegisterClass<(add // a0...a7, t0...t8
 122:                                  (sequence "R%u", 4, 20))>;
 123: 
 124: // Don't use R1 for JR since that micro-architecture unconditionally treats a
 125: // "jr $ra" as "return from subroutine", hence doing "jr $ra" would interfere
 126: // with both subroutine return prediction and the more general indirect branch
```
- **EN**: This block declares or refines TableGen records such as `GPRT`.
- **CN**: 该代码块声明或细化了 `GPRT` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127: // prediction.
 128: def GPRJR : GPRRegisterClass<(sub GPR, R1)>;
 129: 
 130: // Don't use R0 or R1 for the rj operand of [G]CSRXCHG, because when rj is
 131: // encoded as 0 or 1, the instruction is interpreted as [G]CSRRD or [G]CSRWR,
 132: // respectively, rather than [G]CSRXCHG.
 133: def GPRNoR0R1 : GPRRegisterClass<(sub GPR, R0, R1)>;
 134: 
 135: // Floating point registers
 136: 
 137: let RegAltNameIndices = [RegAliasName] in {
 138:   def F0  : LoongArchReg32<0, "f0", ["fa0"]>, DwarfRegNum<[32]>;
 139:   def F1  : LoongArchReg32<1, "f1", ["fa1"]>, DwarfRegNum<[33]>;
 140:   def F2  : LoongArchReg32<2, "f2", ["fa2"]>, DwarfRegNum<[34]>;
 141:   def F3  : LoongArchReg32<3, "f3", ["fa3"]>, DwarfRegNum<[35]>;
 142:   def F4  : LoongArchReg32<4, "f4", ["fa4"]>, DwarfRegNum<[36]>;
 143:   def F5  : LoongArchReg32<5, "f5", ["fa5"]>, DwarfRegNum<[37]>;
 144:   def F6  : LoongArchReg32<6, "f6", ["fa6"]>, DwarfRegNum<[38]>;
```
- **EN**: This block declares or refines TableGen records such as `GPRJR`, `GPRNoR0R1`, `F0`, `F1`, `F2`, `F3`.
- **CN**: 该代码块声明或细化了 `GPRJR`, `GPRNoR0R1`, `F0`, `F1`, `F2`, `F3` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145:   def F7  : LoongArchReg32<7, "f7", ["fa7"]>, DwarfRegNum<[39]>;
 146:   def F8  : LoongArchReg32<8, "f8", ["ft0"]>, DwarfRegNum<[40]>;
 147:   def F9  : LoongArchReg32<9, "f9", ["ft1"]>, DwarfRegNum<[41]>;
 148:   def F10 : LoongArchReg32<10,"f10", ["ft2"]>, DwarfRegNum<[42]>;
 149:   def F11 : LoongArchReg32<11,"f11", ["ft3"]>, DwarfRegNum<[43]>;
 150:   def F12 : LoongArchReg32<12,"f12", ["ft4"]>, DwarfRegNum<[44]>;
 151:   def F13 : LoongArchReg32<13,"f13", ["ft5"]>, DwarfRegNum<[45]>;
 152:   def F14 : LoongArchReg32<14,"f14", ["ft6"]>, DwarfRegNum<[46]>;
 153:   def F15 : LoongArchReg32<15,"f15", ["ft7"]>, DwarfRegNum<[47]>;
 154:   def F16 : LoongArchReg32<16,"f16", ["ft8"]>, DwarfRegNum<[48]>;
 155:   def F17 : LoongArchReg32<17,"f17", ["ft9"]>, DwarfRegNum<[49]>;
 156:   def F18 : LoongArchReg32<18,"f18", ["ft10"]>, DwarfRegNum<[50]>;
 157:   def F19 : LoongArchReg32<19,"f19", ["ft11"]>, DwarfRegNum<[51]>;
 158:   def F20 : LoongArchReg32<20,"f20", ["ft12"]>, DwarfRegNum<[52]>;
 159:   def F21 : LoongArchReg32<21,"f21", ["ft13"]>, DwarfRegNum<[53]>;
 160:   def F22 : LoongArchReg32<22,"f22", ["ft14"]>, DwarfRegNum<[54]>;
 161:   def F23 : LoongArchReg32<23,"f23", ["ft15"]>, DwarfRegNum<[55]>;
 162:   def F24 : LoongArchReg32<24,"f24", ["fs0"]>, DwarfRegNum<[56]>;
```
- **EN**: This block declares or refines TableGen records such as `F7`, `F8`, `F9`, `F10`, `F11`, `F12`.
- **CN**: 该代码块声明或细化了 `F7`, `F8`, `F9`, `F10`, `F11`, `F12` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163:   def F25 : LoongArchReg32<25,"f25", ["fs1"]>, DwarfRegNum<[57]>;
 164:   def F26 : LoongArchReg32<26,"f26", ["fs2"]>, DwarfRegNum<[58]>;
 165:   def F27 : LoongArchReg32<27,"f27", ["fs3"]>, DwarfRegNum<[59]>;
 166:   def F28 : LoongArchReg32<28,"f28", ["fs4"]>, DwarfRegNum<[60]>;
 167:   def F29 : LoongArchReg32<29,"f29", ["fs5"]>, DwarfRegNum<[61]>;
 168:   def F30 : LoongArchReg32<30,"f30", ["fs6"]>, DwarfRegNum<[62]>;
 169:   def F31 : LoongArchReg32<31,"f31", ["fs7"]>, DwarfRegNum<[63]>;
 170: 
 171:   foreach I = 0-31 in {
 172:     def F#I#_64 : LoongArchReg64<!cast<LoongArchReg32>("F"#I)>,
 173:       DwarfRegNum<[!add(I, 32)]>;
 174:   }
 175: }
 176: 
 177: // The order of registers represents the preferred allocation sequence.
 178: def FPR32 : RegisterClass<"LoongArch", [f32], 32, (sequence "F%u", 0, 31)>;
 179: def FPR64 : RegisterClass<"LoongArch", [f64], 64, (sequence "F%u_64", 0, 31)>;
 180: 
```
- **EN**: This block declares or refines TableGen records such as `F25`, `F26`, `F27`, `F28`, `F29`, `F30`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `F25`, `F26`, `F27`, `F28`, `F29`, `F30` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 181-198 / 第 181-198 行
```tablegen
 181: // Condition flag registers
 182: 
 183: foreach I = 0-7 in
 184: def FCC#I : LoongArchReg<I, "fcc"#I>;
 185: 
 186: def CFR : RegisterClass<"LoongArch", [GRLenVT], 32, (sequence "FCC%u", 0, 7)> {
 187:   let RegInfos = GRLenRI;
 188: }
 189: 
 190: // Control and status registers
 191: 
 192: foreach I = 0-3 in
 193: def FCSR#I : LoongArchReg<I, "fcsr"#I>;
 194: 
 195: let isAllocatable = false in
 196: def FCSR : RegisterClass<"LoongArch", [i32], 32, (sequence "FCSR%u", 0, 3)>;
 197: 
 198: // LSX registers
```
- **EN**: This block declares or refines TableGen records such as `FCC`, `CFR`, `FCSR`, `FCSR`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `FCC`, `CFR`, `FCSR`, `FCSR` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 199-216 / 第 199-216 行
```tablegen
 199: 
 200: foreach I = 0-31 in
 201: def VR#I : LoongArchReg128<!cast<LoongArchReg64>("F"#I#"_64"), "vr"#I>,
 202:            DwarfRegAlias<!cast<LoongArchReg64>("F"#I#"_64")>;
 203: 
 204: def LSX128 : RegisterClass<"LoongArch",
 205:                            [v4f32, v2f64, v16i8, v8i16, v4i32, v2i64],
 206:                            128, (sequence "VR%u", 0, 31)>;
 207: 
 208: // LASX registers
 209: 
 210: foreach I = 0-31 in
 211: def XR#I : LoongArchReg256<!cast<LoongArchReg128>("VR"#I), "xr"#I>,
 212:            DwarfRegAlias<!cast<LoongArchReg128>("VR"#I)>;
 213: 
 214: def LASX256 : RegisterClass<"LoongArch",
 215:                             [v8f32, v4f64, v32i8, v16i16, v8i32, v4i64],
 216:                             256, (sequence "XR%u", 0, 31)>;
```
- **EN**: This block declares or refines TableGen records such as `VR`, `LSX128`, `XR`, `LASX256`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `VR`, `LSX128`, `XR`, `LASX256` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 217-224 / 第 217-224 行
```tablegen
 217: 
 218: // Scratchpad registers
 219: 
 220: foreach I = 0-3 in
 221: def SCR#I : LoongArchReg<I, "scr"#I>;
 222: 
 223: let isAllocatable = false, RegInfos = GRLenRI in
 224: def SCR : RegisterClass<"LoongArch", [GRLenVT], 32, (sequence "SCR%u", 0, 3)>;
```
- **EN**: This block declares or refines TableGen records such as `SCR`, `SCR`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `SCR`, `SCR` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
