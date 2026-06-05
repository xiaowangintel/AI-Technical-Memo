# AVRRegisterInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRRegisterInfo.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines target registers, register classes, and register relations in TableGen.
- 目的（中文）: 使用 TableGen 定义目标寄存器、寄存器类以及寄存器之间的关系。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- AVRRegisterInfo.td - AVR Register defs -------------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: //  Declarations that describe the AVR register file
  11: //===----------------------------------------------------------------------===//
  12: 
  13: // 8-bit General purpose register definition.
  14: class AVRReg<bits<16> num, string name, list<Register> subregs = [],
  15:              list<string> altNames = []> : RegisterWithSubRegs<name, subregs> {
  16:   field bits<16> Num = num;
  17: 
  18:   let HWEncoding = num;
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as AVRReg, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 AVRReg 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 19-36

```tablegen
  19:   let Namespace = "AVR";
  20:   let SubRegs = subregs;
  21:   let AltNames = altNames;
  22: }
  23: 
  24: // Subregister indices.
  25: let Namespace = "AVR" in {
  26:   def sub_lo : SubRegIndex<8>;
  27:   def sub_hi : SubRegIndex<8, 8>;
  28: }
  29: 
  30: let Namespace = "AVR" in { def ptr : RegAltNameIndex; }
  31: 
  32: //===----------------------------------------------------------------------===//
  33: //  8-bit general purpose registers
  34: //===----------------------------------------------------------------------===//
  35: 
  36: def R0 : AVRReg<0, "r0">, DwarfRegNum<[0]>;
```

- EN: This range defines declarative TableGen records such as sub_lo, sub_hi, R0, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 sub_lo, sub_hi, R0 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37: def R1 : AVRReg<1, "r1">, DwarfRegNum<[1]>;
  38: def R2 : AVRReg<2, "r2">, DwarfRegNum<[2]>;
  39: def R3 : AVRReg<3, "r3">, DwarfRegNum<[3]>;
  40: def R4 : AVRReg<4, "r4">, DwarfRegNum<[4]>;
  41: def R5 : AVRReg<5, "r5">, DwarfRegNum<[5]>;
  42: def R6 : AVRReg<6, "r6">, DwarfRegNum<[6]>;
  43: def R7 : AVRReg<7, "r7">, DwarfRegNum<[7]>;
  44: def R8 : AVRReg<8, "r8">, DwarfRegNum<[8]>;
  45: def R9 : AVRReg<9, "r9">, DwarfRegNum<[9]>;
  46: def R10 : AVRReg<10, "r10">, DwarfRegNum<[10]>;
  47: def R11 : AVRReg<11, "r11">, DwarfRegNum<[11]>;
  48: def R12 : AVRReg<12, "r12">, DwarfRegNum<[12]>;
  49: def R13 : AVRReg<13, "r13">, DwarfRegNum<[13]>;
  50: def R14 : AVRReg<14, "r14">, DwarfRegNum<[14]>;
  51: def R15 : AVRReg<15, "r15">, DwarfRegNum<[15]>;
  52: def R16 : AVRReg<16, "r16">, DwarfRegNum<[16]>;
  53: def R17 : AVRReg<17, "r17">, DwarfRegNum<[17]>;
  54: def R18 : AVRReg<18, "r18">, DwarfRegNum<[18]>;
```

- EN: This range defines declarative TableGen records such as R1, R2, R3, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 R1, R2, R3 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55: def R19 : AVRReg<19, "r19">, DwarfRegNum<[19]>;
  56: def R20 : AVRReg<20, "r20">, DwarfRegNum<[20]>;
  57: def R21 : AVRReg<21, "r21">, DwarfRegNum<[21]>;
  58: def R22 : AVRReg<22, "r22">, DwarfRegNum<[22]>;
  59: def R23 : AVRReg<23, "r23">, DwarfRegNum<[23]>;
  60: def R24 : AVRReg<24, "r24">, DwarfRegNum<[24]>;
  61: def R25 : AVRReg<25, "r25">, DwarfRegNum<[25]>;
  62: def R26 : AVRReg<26, "r26", [], ["xl"]>, DwarfRegNum<[26]>;
  63: def R27 : AVRReg<27, "r27", [], ["xh"]>, DwarfRegNum<[27]>;
  64: def R28 : AVRReg<28, "r28", [], ["yl"]>, DwarfRegNum<[28]>;
  65: def R29 : AVRReg<29, "r29", [], ["yh"]>, DwarfRegNum<[29]>;
  66: def R30 : AVRReg<30, "r30", [], ["zl"]>, DwarfRegNum<[30]>;
  67: def R31 : AVRReg<31, "r31", [], ["zh"]>, DwarfRegNum<[31]>;
  68: def SPL : AVRReg<32, "SPL">, DwarfRegNum<[32]>;
  69: def SPH : AVRReg<33, "SPH">, DwarfRegNum<[33]>;
  70: 
  71: // 16 bit GPR pairs.
  72: let SubRegIndices = [sub_lo, sub_hi], CoveredBySubRegs = 1 in {
```

- EN: This range defines declarative TableGen records such as R19, R20, R21, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R19, R20, R21 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 73-90

```tablegen
  73:   // The value 16 for the encoding is arbitrary. SP register is not encoded
  74:   // into instructions, they use it implicitly depending on the opcode.
  75:   def SP : AVRReg<16, "SP", [SPL, SPH]>, DwarfRegNum<[32]>;
  76: 
  77:   // The pointer registers (X,Y,Z) are a special case because they
  78:   // are printed as a `high:low` pair when a DREG is expected,
  79:   // but printed using `X`, `Y`, `Z` when a pointer register is expected.
  80:   // DREG registers are only used in ADIW, SBIW and MOVW instructions.
  81:   let RegAltNameIndices = [ptr] in {
  82:     def R31R30 : AVRReg<15, "r31:r30", [R30, R31], ["Z"]>, DwarfRegNum<[30]>;
  83:     def R29R28 : AVRReg<14, "r29:r28", [R28, R29], ["Y"]>, DwarfRegNum<[28]>;
  84:     def R27R26 : AVRReg<13, "r27:r26", [R26, R27], ["X"]>, DwarfRegNum<[26]>;
  85:   }
  86:   def R25R24 : AVRReg<12, "r25:r24", [R24, R25]>, DwarfRegNum<[24]>;
  87:   def R23R22 : AVRReg<11, "r23:r22", [R22, R23]>, DwarfRegNum<[22]>;
  88:   def R21R20 : AVRReg<10, "r21:r20", [R20, R21]>, DwarfRegNum<[20]>;
  89:   def R19R18 : AVRReg<9, "r19:r18", [R18, R19]>, DwarfRegNum<[18]>;
  90:   def R17R16 : AVRReg<8, "r17:r16", [R16, R17]>, DwarfRegNum<[16]>;
```

- EN: This range defines declarative TableGen records such as SP, R31R30, R29R28, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SP, R31R30, R29R28 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91:   def R15R14 : AVRReg<7, "r15:r14", [R14, R15]>, DwarfRegNum<[14]>;
  92:   def R13R12 : AVRReg<6, "r13:r12", [R12, R13]>, DwarfRegNum<[12]>;
  93:   def R11R10 : AVRReg<5, "r11:r10", [R10, R11]>, DwarfRegNum<[10]>;
  94:   def R9R8 : AVRReg<4, "r9:r8", [R8, R9]>, DwarfRegNum<[8]>;
  95:   def R7R6 : AVRReg<3, "r7:r6", [R6, R7]>, DwarfRegNum<[6]>;
  96:   def R5R4 : AVRReg<2, "r5:r4", [R4, R5]>, DwarfRegNum<[4]>;
  97:   def R3R2 : AVRReg<1, "r3:r2", [R2, R3]>, DwarfRegNum<[2]>;
  98:   def R1R0 : AVRReg<0, "r1:r0", [R0, R1]>, DwarfRegNum<[0]>;
  99: 
 100:   // Pseudo registers for unaligned i16. These are only used in pseudo
 101:   // instructions, so encoding values are arbitrary.
 102:   def R26R25 : AVRReg<25, "r26:r25", [R25, R26]>, DwarfRegNum<[25]>;
 103:   def R24R23 : AVRReg<23, "r24:r23", [R23, R24]>, DwarfRegNum<[23]>;
 104:   def R22R21 : AVRReg<21, "r22:r21", [R21, R22]>, DwarfRegNum<[21]>;
 105:   def R20R19 : AVRReg<19, "r20:r19", [R19, R20]>, DwarfRegNum<[19]>;
 106:   def R18R17 : AVRReg<17, "r18:r17", [R17, R18]>, DwarfRegNum<[17]>;
 107:   def R16R15 : AVRReg<15, "r16:r15", [R15, R16]>, DwarfRegNum<[15]>;
 108:   def R14R13 : AVRReg<13, "r14:r13", [R13, R14]>, DwarfRegNum<[13]>;
```

- EN: This range defines declarative TableGen records such as R15R14, R13R12, R11R10, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 R15R14, R13R12, R11R10 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 109-126

```tablegen
 109:   def R12R11 : AVRReg<11, "r12:r11", [R11, R12]>, DwarfRegNum<[11]>;
 110:   def R10R9 : AVRReg<9, "r10:r9", [R9, R10]>, DwarfRegNum<[9]>;
 111: }
 112: 
 113: //===----------------------------------------------------------------------===//
 114: // Register Classes
 115: //===----------------------------------------------------------------------===//
 116: 
 117: // Main 8-bit register class.
 118: def GPR8 : RegisterClass<"AVR", [i8], 8,
 119:                          (// Return value and argument registers.
 120:                           add R24, R25, R18, R19, R20, R21, R22, R23,
 121:                           // Scratch registers.
 122:                           R30, R31, R26, R27,
 123:                           // Callee saved registers.
 124:                           R28, R29, R17, R16, R15, R14, R13, R12, R11, R10,
 125:                           R9, R8, R7, R6, R5, R4, R3, R2, R0, R1)>;
 126: 
```

- EN: This range defines declarative TableGen records such as R12R11, R10R9, GPR8, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 R12R11, R10R9, GPR8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 127-144

```tablegen
 127: // Simple lower registers r0..r15
 128: def GPR8lo : RegisterClass<"AVR", [i8], 8,
 129:                            (add R15, R14, R13, R12, R11, R10, R9, R8, R7, R6,
 130:                             R5, R4, R3, R2, R0, R1)>;
 131: 
 132: // 8-bit register class for instructions which take immediates.
 133: def LD8 : RegisterClass<"AVR", [i8], 8,
 134:                         (// Return value and arguments.
 135:                          add R24, R25, R18, R19, R20, R21, R22, R23,
 136:                          // Scratch registers.
 137:                          R30, R31, R26, R27,
 138:                          // Callee saved registers.
 139:                          R28, R29, R17, R16)>;
 140: 
 141: // Simple lower registers r16..r23
 142: def LD8lo : RegisterClass<"AVR", [i8], 8,
 143:                           (add R23, R22, R21, R20, R19, R18, R17, R16)>;
 144: 
```

- EN: This range defines declarative TableGen records such as GPR8lo, LD8, LD8lo, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 GPR8lo, LD8, LD8lo 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 145-162

```tablegen
 145: // Main 16-bit pair register class.
 146: def DREGS : RegisterClass<"AVR", [i16], 8,
 147:                           (// Return value and arguments.
 148:                            add R25R24, R19R18, R21R20, R23R22,
 149:                            // Scratch registers.
 150:                            R31R30, R27R26,
 151:                            // Callee saved registers.
 152:                            R29R28, R17R16, R15R14, R13R12, R11R10, R9R8,
 153:                            R7R6, R5R4, R3R2, R1R0,
 154:                            // Pseudo regs for unaligned 16-bits
 155:                            R26R25, R24R23, R22R21, R20R19, R18R17, R16R15,
 156:                            R14R13, R12R11, R10R9)>;
 157: 
 158: // Lower 16-bit pair registers in R0..R15, only used in inline assembly.
 159: def DREGSlo
 160:     : RegisterClass<"AVR", [i16], 8,
 161:                     (add R15R14, R13R12, R11R10, R9R8, R7R6, R5R4, R3R2, R1R0)>;
 162: 
```

- EN: This range defines declarative TableGen records such as DREGS, DREGSlo, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 DREGS, DREGSlo 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 163-180

```tablegen
 163: // Lower 16-bit pair registers in r16..r23, only used in inline assembly.
 164: def DREGSLD8lo : RegisterClass<"AVR", [i16], 8,
 165:                                (// Return value and arguments.
 166:                                 add R19R18, R21R20, R23R22,
 167:                                 // Callee saved registers.
 168:                                 R17R16)>;
 169: 
 170: // 16-bit pair register class for movw
 171: def DREGSMOVW : RegisterClass<"AVR", [i16], 8,
 172:                               (// Return value and arguments.
 173:                                add R25R24, R19R18, R21R20, R23R22,
 174:                                // Scratch registers.
 175:                                R31R30, R27R26,
 176:                                // Callee saved registers.
 177:                                R29R28, R17R16, R15R14, R13R12, R11R10, R9R8,
 178:                                R7R6, R5R4, R3R2, R1R0)>;
 179: 
 180: // 16-bit register class for immediate instructions.
```

- EN: This range defines declarative TableGen records such as DREGSLD8lo, DREGSMOVW, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 DREGSLD8lo, DREGSMOVW 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 181-198

```tablegen
 181: def DLDREGS : RegisterClass<"AVR", [i16], 8,
 182:                             (// Return value and arguments.
 183:                              add R25R24, R19R18, R21R20, R23R22,
 184:                              // Scratch registers.
 185:                              R31R30, R27R26,
 186:                              // Callee saved registers.
 187:                              R29R28, R17R16)>;
 188: 
 189: // 16-bit register class for the adiw/sbiw instructions.
 190: def IWREGS : RegisterClass<"AVR", [i16], 8,
 191:                            (// Return value and arguments.
 192:                             add R25R24,
 193:                             // Scratch registers.
 194:                             R31R30, R27R26,
 195:                             // Callee saved registers.
 196:                             R29R28)>;
 197: 
 198: // 16-bit register class for the ld and st instructions.
```

- EN: This range defines declarative TableGen records such as DLDREGS, IWREGS, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 DLDREGS, IWREGS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 199-216

```tablegen
 199: // AKA X,Y, and Z
 200: def PTRREGS : RegisterClass<"AVR", [i16], 8,
 201:                             (add R27R26, // X
 202:                              R29R28,     // Y
 203:                              R31R30),    // Z
 204:                              ptr>;
 205: 
 206: // 16-bit register class for the ldd and std instructions.
 207: // AKA Y and Z.
 208: def PTRDISPREGS : RegisterClass<"AVR", [i16], 8, (add R31R30, R29R28), ptr>;
 209: 
 210: // We have a bunch of instructions with an explicit Z register argument. We
 211: // model this using a register class containing only the Z register.
 212: def ZREG : RegisterClass<"AVR", [i16], 8, (add R31R30)>;
 213: 
 214: // general registers excluding Z register lo/hi, these are the only
 215: // registers that are always safe for STDSPQr instructions
 216: def GPR8NOZ : RegisterClass<"AVR", [i8], 8,
```

- EN: This range defines declarative TableGen records such as PTRREGS, PTRDISPREGS, ZREG, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 PTRREGS, PTRDISPREGS, ZREG 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 217-234

```tablegen
 217:                          (// Return value and argument registers.
 218:                           add R24, R25, R18, R19, R20, R21, R22, R23,
 219:                           // Scratch registers.
 220:                           R26, R27,
 221:                           // Callee saved registers.
 222:                           R28, R29, R17, R16, R15, R14, R13, R12, R11, R10,
 223:                           R9, R8, R7, R6, R5, R4, R3, R2, R0, R1)>;
 224: 
 225: // 16-bit pair register class excluding Z register lo/hi, these are the only
 226: // registers that are always safe for STDWSPQr instructions
 227: def DREGSNOZ : RegisterClass<"AVR", [i16], 8,
 228:                           (// Return value and arguments.
 229:                            add R25R24, R19R18, R21R20, R23R22,
 230:                            // Scratch registers.
 231:                            R27R26,
 232:                            // Callee saved registers.
 233:                            R29R28, R17R16, R15R14, R13R12, R11R10, R9R8,
 234:                            R7R6, R5R4, R3R2, R1R0,
```

- EN: This range defines declarative TableGen records such as DREGSNOZ, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 DREGSNOZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 235-246

```tablegen
 235:                            // Pseudo regs for unaligned 16-bits
 236:                            R26R25, R24R23, R22R21, R20R19, R18R17, R16R15,
 237:                            R14R13, R12R11, R10R9)>;
 238: 
 239: // Register class used for the stack read pseudo instruction.
 240: def GPRSP : RegisterClass<"AVR", [i16], 8, (add SP)>;
 241: 
 242: // Status register.
 243: def SREG : AVRReg<14, "FLAGS">, DwarfRegNum<[88]>;
 244: def CCR : RegisterClass<"AVR", [i8], 8, (add SREG)> {
 245:   let CopyCost = -1; // Don't allow copying of status registers
 246: }
```

- EN: This range defines declarative TableGen records such as GPRSP, SREG, CCR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 GPRSP, SREG, CCR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Register definitions / 寄存器定义
- Register classes and aliases / 寄存器类与别名
- Register classes / 寄存器类
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
- Local companions / 本地配套文件: `AVRRegisterInfo.h`, `AVRRegisterInfo.cpp`
