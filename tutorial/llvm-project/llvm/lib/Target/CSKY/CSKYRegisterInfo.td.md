# CSKYRegisterInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYRegisterInfo.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines target registers, register classes, and register relations in TableGen.
- 目的（中文）: 使用 TableGen 定义目标寄存器、寄存器类以及寄存器之间的关系。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- CSKYRegisterInfo.td - CSKY Register defs -----------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: //  Declarations that describe the CSKY registers.
  11: //===----------------------------------------------------------------------===//
  12: 
  13: let Namespace = "CSKY" in {
  14:   class CSKYReg<bits<6> Enc, string n, list<string> alt = []> : Register<n> {
  15:     let HWEncoding{5 - 0} = Enc;
  16:     let AltNames = alt;
  17:   }
  18: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as CSKYReg, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 CSKYReg 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 19-36

```tablegen
  19:   class CSKYFReg32<bits<5> Enc, string n, list<string> alt = []> : Register<n> {
  20:     let HWEncoding{4 - 0} = Enc;
  21:     let AltNames = alt;
  22:   }
  23: 
  24:   // Because CSKYFReg64 register have AsmName and AltNames that alias with their
  25:   // 32-bit sub-register, CSKYAsmParser will need to coerce a register number
  26:   // from a CSKYFReg32 to the equivalent CSKYFReg64 when appropriate.
  27:   def sub32_0 : SubRegIndex<32, 0>;
  28:   def sub32_32 : SubRegIndex<32, 32>;
  29:   def sub64_0 : SubRegIndex<64, 0>;
  30:   def sub64_64 : SubRegIndex<64,64>;
  31: 
  32:   class CSKYFReg64<CSKYFReg32 subreg> : Register<""> {
  33:     let HWEncoding{4 - 0} = subreg.HWEncoding{4 - 0};
  34:     let SubRegs = [subreg];
  35:     let SubRegIndices = [sub32_0];
  36:     let AsmName = subreg.AsmName;
```

- EN: This range defines declarative TableGen records such as CSKYFReg32, sub32_0, sub32_32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CSKYFReg32, sub32_0, sub32_32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37:     let AltNames = subreg.AltNames;
  38:   }
  39: 
  40:   class CSKYFReg128<CSKYFReg64 subreg> : Register<""> {
  41:     let HWEncoding{4 - 0} = subreg.HWEncoding{4 - 0};
  42:     let SubRegs = [subreg];
  43:     let SubRegIndices = [sub64_0];
  44:     let AsmName = subreg.AsmName;
  45:     let AltNames = subreg.AltNames;
  46:   }
  47: 
  48:   def ABIRegAltName : RegAltNameIndex;
  49: } // Namespace = "CSKY"
  50: 
  51: let RegAltNameIndices = [ABIRegAltName] in {
  52:   def R0 : CSKYReg<0, "r0", ["a0"]>, DwarfRegNum<[0]>;
  53:   def R1 : CSKYReg<1, "r1", ["a1"]>, DwarfRegNum<[1]>;
  54:   def R2 : CSKYReg<2, "r2", ["a2"]>, DwarfRegNum<[2]>;
```

- EN: This range defines declarative TableGen records such as CSKYFReg128, ABIRegAltName, R0, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CSKYFReg128, ABIRegAltName, R0 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 55-72

```tablegen
  55:   def R3 : CSKYReg<3, "r3", ["a3"]>, DwarfRegNum<[3]>;
  56:   def R4 : CSKYReg<4, "r4", ["l0"]>, DwarfRegNum<[4]>;
  57:   def R5 : CSKYReg<5, "r5", ["l1"]>, DwarfRegNum<[5]>;
  58:   def R6 : CSKYReg<6, "r6", ["l2"]>, DwarfRegNum<[6]>;
  59:   def R7 : CSKYReg<7, "r7", ["l3"]>, DwarfRegNum<[7]>;
  60:   def R8 : CSKYReg<8, "r8", ["l4"]>, DwarfRegNum<[8]>;
  61:   def R9 : CSKYReg<9, "r9", ["l5"]>, DwarfRegNum<[9]>;
  62:   def R10 : CSKYReg<10, "r10", ["l6"]>, DwarfRegNum<[10]>;
  63:   def R11 : CSKYReg<11, "r11", ["l7"]>, DwarfRegNum<[11]>;
  64:   def R12 : CSKYReg<12, "r12", ["t0"]>, DwarfRegNum<[12]>;
  65:   def R13 : CSKYReg<13, "r13", ["t1"]>, DwarfRegNum<[13]>;
  66:   def R14 : CSKYReg<14, "r14", ["sp"]>, DwarfRegNum<[14]>;
  67:   def R15 : CSKYReg<15, "r15", ["lr"]>, DwarfRegNum<[15]>;
  68:   def R16 : CSKYReg<16, "r16", ["l8"]>, DwarfRegNum<[16]>;
  69:   def R17 : CSKYReg<17, "r17", ["l9"]>, DwarfRegNum<[17]>;
  70:   def R18 : CSKYReg<18, "r18", ["t2"]>, DwarfRegNum<[18]>;
  71:   def R19 : CSKYReg<19, "r19", ["t3"]>, DwarfRegNum<[19]>;
  72:   def R20 : CSKYReg<20, "r20", ["t4"]>, DwarfRegNum<[20]>;
```

- EN: This range defines declarative TableGen records such as R3, R4, R5, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 R3, R4, R5 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-90

```tablegen
  73:   def R21 : CSKYReg<21, "r21", ["t5"]>, DwarfRegNum<[21]>;
  74:   def R22 : CSKYReg<22, "r22", ["t6"]>, DwarfRegNum<[22]>;
  75:   def R23 : CSKYReg<23, "r23", ["t7"]>, DwarfRegNum<[23]>;
  76:   def R24 : CSKYReg<24, "r24", ["t8"]>, DwarfRegNum<[24]>;
  77:   def R25 : CSKYReg<25, "r25", ["t9"]>, DwarfRegNum<[25]>;
  78:   def R26 : CSKYReg<26, "r26", ["r26"]>, DwarfRegNum<[26]>;
  79:   def R27 : CSKYReg<27, "r27", ["r27"]>, DwarfRegNum<[27]>;
  80:   def R28 : CSKYReg<28, "r28", ["rgb"]>, DwarfRegNum<[28]>;
  81:   def R29 : CSKYReg<29, "r29", ["rtb"]>, DwarfRegNum<[29]>;
  82:   def R30 : CSKYReg<30, "r30", ["svbr"]>, DwarfRegNum<[30]>;
  83:   def R31 : CSKYReg<31, "r31", ["tls"]>, DwarfRegNum<[31]>;
  84: 
  85:   // Faked for GPRTuple
  86:   def R32 : CSKYReg<32, "r32", ["r32"]>, DwarfRegNum<[32]>;
  87: 
  88:   def C : CSKYReg<33, "cr0", ["psr"]>;
  89: 
  90: }
```

- EN: This range defines declarative TableGen records such as R21, R22, R23, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 R21, R22, R23 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 91-108

```tablegen
  91: 
  92: def GPRTuple : RegisterTuples<
  93:           [sub32_0, sub32_32],
  94:           [(add (sequence "R%u", 0, 31)), (add (sequence "R%u", 1, 32))],
  95:           [ "r0",  "r1",  "r2",  "r3",  "r4",  "r5",  "r6",  "r7",
  96:             "r8",  "r9",  "r10", "r11", "r12", "r13", "r14", "r15",
  97:             "r16", "r17", "r18", "r19", "r20", "r21", "r22", "r23",
  98:             "r24", "r25", "r26", "r27", "r28", "r29", "r30", "r31"
  99:           ]>;
 100: 
 101: // Floating point registers
 102: let RegAltNameIndices = [ABIRegAltName] in {
 103:   def F0_32 : CSKYFReg32<0, "fr0", ["vr0"]>, DwarfRegNum<[32]>;
 104:   def F1_32 : CSKYFReg32<1, "fr1", ["vr1"]>, DwarfRegNum<[33]>;
 105:   def F2_32 : CSKYFReg32<2, "fr2", ["vr2"]>, DwarfRegNum<[34]>;
 106:   def F3_32 : CSKYFReg32<3, "fr3", ["vr3"]>, DwarfRegNum<[35]>;
 107:   def F4_32 : CSKYFReg32<4, "fr4", ["vr4"]>, DwarfRegNum<[36]>;
 108:   def F5_32 : CSKYFReg32<5, "fr5", ["vr5"]>, DwarfRegNum<[37]>;
```

- EN: This range defines declarative TableGen records such as GPRTuple, F0_32, F1_32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 GPRTuple, F0_32, F1_32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109:   def F6_32 : CSKYFReg32<6, "fr6", ["vr6"]>, DwarfRegNum<[38]>;
 110:   def F7_32 : CSKYFReg32<7, "fr7", ["vr7"]>, DwarfRegNum<[39]>;
 111:   def F8_32 : CSKYFReg32<8, "fr8", ["vr8"]>, DwarfRegNum<[40]>;
 112:   def F9_32 : CSKYFReg32<9, "fr9", ["vr9"]>, DwarfRegNum<[41]>;
 113:   def F10_32 : CSKYFReg32<10, "fr10", ["vr10"]>, DwarfRegNum<[42]>;
 114:   def F11_32 : CSKYFReg32<11, "fr11", ["vr11"]>, DwarfRegNum<[43]>;
 115:   def F12_32 : CSKYFReg32<12, "fr12", ["vr12"]>, DwarfRegNum<[44]>;
 116:   def F13_32 : CSKYFReg32<13, "fr13", ["vr13"]>, DwarfRegNum<[45]>;
 117:   def F14_32 : CSKYFReg32<14, "fr14", ["vr14"]>, DwarfRegNum<[46]>;
 118:   def F15_32 : CSKYFReg32<15, "fr15", ["vr15"]>, DwarfRegNum<[47]>;
 119:   def F16_32 : CSKYFReg32<16, "fr16", ["vr16"]>, DwarfRegNum<[48]>;
 120:   def F17_32 : CSKYFReg32<17, "fr17", ["vr17"]>, DwarfRegNum<[49]>;
 121:   def F18_32 : CSKYFReg32<18, "fr18", ["vr18"]>, DwarfRegNum<[50]>;
 122:   def F19_32 : CSKYFReg32<19, "fr19", ["vr19"]>, DwarfRegNum<[51]>;
 123:   def F20_32 : CSKYFReg32<20, "fr20", ["vr20"]>, DwarfRegNum<[52]>;
 124:   def F21_32 : CSKYFReg32<21, "fr21", ["vr21"]>, DwarfRegNum<[53]>;
 125:   def F22_32 : CSKYFReg32<22, "fr22", ["vr22"]>, DwarfRegNum<[54]>;
 126:   def F23_32 : CSKYFReg32<23, "fr23", ["vr23"]>, DwarfRegNum<[55]>;
```

- EN: This range defines declarative TableGen records such as F6_32, F7_32, F8_32, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 F6_32, F7_32, F8_32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 127-144

```tablegen
 127:   def F24_32 : CSKYFReg32<24, "fr24", ["vr24"]>, DwarfRegNum<[56]>;
 128:   def F25_32 : CSKYFReg32<25, "fr25", ["vr25"]>, DwarfRegNum<[57]>;
 129:   def F26_32 : CSKYFReg32<26, "fr26", ["vr26"]>, DwarfRegNum<[58]>;
 130:   def F27_32 : CSKYFReg32<27, "fr27", ["vr27"]>, DwarfRegNum<[59]>;
 131:   def F28_32 : CSKYFReg32<28, "fr28", ["vr28"]>, DwarfRegNum<[60]>;
 132:   def F29_32 : CSKYFReg32<29, "fr29", ["vr29"]>, DwarfRegNum<[61]>;
 133:   def F30_32 : CSKYFReg32<30, "fr30", ["vr30"]>, DwarfRegNum<[62]>;
 134:   def F31_32 : CSKYFReg32<31, "fr31", ["vr31"]>, DwarfRegNum<[63]>;
 135: 
 136:   foreach Index = 0 - 31 in {
 137:     def F#Index#_64 : CSKYFReg64<!cast<CSKYFReg32>("F"#Index#"_32")>,
 138:                       DwarfRegNum<[!add(Index, 32)]>;
 139: 
 140:     def F#Index#_128 : CSKYFReg128<!cast<CSKYFReg64>("F"#Index#"_64")>,
 141:                        DwarfRegNum<[!add(Index, 32)]>;
 142:   }
 143: }
 144: 
```

- EN: This range defines declarative TableGen records such as F24_32, F25_32, F26_32, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 F24_32, F25_32, F26_32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 145-162

```tablegen
 145: 
 146: //===----------------------------------------------------------------------===//
 147: // Declarations that describe the CSKY register class.
 148: //===----------------------------------------------------------------------===//
 149: 
 150: // The order of registers represents the preferred allocation sequence.
 151: // Registers are listed in the order caller-save, callee-save, specials.
 152: def GPR : RegisterClass<"CSKY", [i32], 32,
 153:                         (add (sequence "R%u", 0, 3), (sequence "R%u", 12, 13),
 154:                              (sequence "R%u", 18, 25), R15, (sequence "R%u", 4, 11),
 155:                              (sequence "R%u", 16, 17), (sequence "R%u", 26, 27), R28,
 156:                              (sequence "R%u", 29, 30), R14, R31)> {
 157:   let Size = 32;
 158: }
 159: 
 160: // Register class for R0 - R15.
 161: // Some 16-bit integer instructions can only access R0 - R15.
 162: def sGPR : RegisterClass<"CSKY", [i32], 32,
```

- EN: This range defines declarative TableGen records such as GPR, sGPR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 GPR, sGPR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163:                         (add (sequence "R%u", 0, 3), (sequence "R%u", 12, 13), R15,
 164:                         (sequence "R%u", 4, 11), R14)> {
 165:   let Size = 32;
 166: }
 167: 
 168: // Register class for R0 - R7.
 169: // Some 16-bit integer instructions can only access R0 - R7.
 170: def mGPR : RegisterClass<"CSKY", [i32], 32,
 171:                         (add (sequence "R%u", 0, 7))> {
 172:   let Size = 32;
 173: }
 174: 
 175: // Register class for SP only.
 176: def GPRSP : RegisterClass<"CSKY", [i32], 32, (add R14)> {
 177:   let Size = 32;
 178: }
 179: 
 180: def GPRPair : RegisterClass<"CSKY", [untyped], 32, (add GPRTuple)> {
```

- EN: This range defines declarative TableGen records such as mGPR, GPRSP, GPRPair, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 mGPR, GPRSP, GPRPair 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 181-198

```tablegen
 181:   let Size = 64;
 182: }
 183: 
 184: def CARRY : RegisterClass<"CSKY", [i32], 32, (add C)> {
 185:   let Size = 32;
 186:   let CopyCost = -1;
 187: }
 188: 
 189: // The order of registers represents the preferred allocation sequence.
 190: // Registers are listed in the order caller-save, callee-save, specials.
 191: def FPR32 : RegisterClass<"CSKY", [f32], 32,
 192:                          (add (sequence "F%u_32", 0, 31))>;
 193: def sFPR32 : RegisterClass<"CSKY", [f32], 32,
 194:                          (add (sequence "F%u_32", 0, 15))>;
 195: 
 196: def FPR64 : RegisterClass<"CSKY", [f64], 32,
 197:                          (add (sequence "F%u_64", 0, 31))>;
 198: def sFPR64 : RegisterClass<"CSKY", [f64], 32,
```

- EN: This range defines declarative TableGen records such as CARRY, FPR32, sFPR32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CARRY, FPR32, sFPR32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-208

```tablegen
 199:                          (add (sequence "F%u_64", 0, 15))>;
 200: 
 201: def sFPR64_V : RegisterClass<"CSKY", [v2f32], 32, (add sFPR64)>;
 202: 
 203: def FPR128 : RegisterClass<"CSKY",
 204:              [v16i8, v8i16, v4i32, v2i64, v4f32, v2f64, v8f16], 128,
 205:              (add (sequence "F%u_128", 0, 31))>;
 206: def sFPR128 : RegisterClass<"CSKY",
 207:               [v16i8, v8i16, v4i32, v2i64, v4f32, v2f64, v8f16], 128,
 208:               (add (sequence "F%u_128", 0, 15))>;
```

- EN: This range defines declarative TableGen records such as sFPR64_V, FPR128, sFPR128, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 sFPR64_V, FPR128, sFPR128 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Register definitions / 寄存器定义
- Register classes and aliases / 寄存器类与别名
- Register classes / 寄存器类
- Assembly parsing / 汇编解析

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
- Local companions / 本地配套文件: `CSKYRegisterInfo.h`, `CSKYRegisterInfo.cpp`
