# AVRDevices.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRDevices.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines target metadata in TableGen; this is declarative DSL code rather than ordinary C++ implementation.
- 目的（中文）: 使用 TableGen 定义目标元数据；这是声明式 DSL 代码，而不是普通 C++ 实现。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===---------------------------------------------------------------------===//
   2: // AVR Device Definitions
   3: //===---------------------------------------------------------------------===//
   4: 
   5: // :TODO: Implement the skip errata, see `gcc/config/avr/avr-arch.h` for details
   6: // :TODO: We define all devices with SRAM to have all variants of LD/ST/LDD/STD.
   7: //        In reality, avr1 (no SRAM) has one variant each of `LD` and `ST`.
   8: //        avr2 (with SRAM) adds the rest of the variants.
   9: 
  10: // A feature set aggregates features, grouping them. We don't want to create a
  11: // new member in AVRSubtarget (to store a value) for each set because we do not
  12: // care if the set is supported, only the subfeatures inside the set. We fix
  13: // this by simply setting the same dummy member for all feature sets, which is
  14: // then ignored.
  15: class FeatureSet<string name, string desc, list<SubtargetFeature> i>
  16:     : SubtargetFeature<name, "HasFeatureSet"#NAME, "true", desc, i>;
  17: 
  18: // A family of microcontrollers, defining a set of supported features.
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as FeatureSet, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 FeatureSet 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19: class Family<string name, list<SubtargetFeature> i>
  20:     : FeatureSet<
  21:           name, !strconcat("The device is a part of the ", name, " family"), i>;
  22: 
  23: // The device has SRAM, and supports the bare minimum of
  24: // SRAM-relevant instructions.
  25: //
  26: // These are:
  27: // LD - all 9 variants
  28: // ST - all 9 variants
  29: // LDD - two variants for Y and Z
  30: // STD - two variants for Y and Z
  31: // `LDS Rd, K`
  32: // `STS k, Rr`
  33: // `PUSH`/`POP`
  34: def FeatureSRAM : SubtargetFeature<"sram", "HasSRAM", "true",
  35:                                    "The device has random access memory">;
  36: 
```

- EN: This range defines declarative TableGen records such as Family, FeatureSRAM, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Family, FeatureSRAM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-54

```tablegen
  37: // The device supports the `JMP k` and `CALL k` instructions.
  38: def FeatureJMPCALL : SubtargetFeature<"jmpcall", "HasJMPCALL", "true",
  39:                                       "The device supports the `JMP` and "
  40:                                       "`CALL` instructions">;
  41: 
  42: // The device supports the indirect branches `IJMP` and `ICALL`.
  43: def FeatureIJMPCALL : SubtargetFeature<"ijmpcall", "HasSRAMIJMPCALL", "true",
  44:                                        "The device supports `IJMP`/`ICALL`"
  45:                                        "instructions">;
  46: 
  47: // The device supports the extended indirect branches `EIJMP` and `EICALL`.
  48: def FeatureEIJMPCALL : SubtargetFeature<"eijmpcall", "HasEIJMPCALL", "true",
  49:                                         "The device supports the "
  50:                                         "`EIJMP`/`EICALL` instructions">;
  51: 
  52: // The device supports `ADDI Rd, K`, `SUBI Rd, K`.
  53: def FeatureADDSUBIW : SubtargetFeature<"addsubiw", "HasADDSUBIW", "true",
  54:                                        "Enable 16-bit register-immediate "
```

- EN: This range defines declarative TableGen records such as FeatureJMPCALL, FeatureIJMPCALL, FeatureEIJMPCALL, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureJMPCALL, FeatureIJMPCALL, FeatureEIJMPCALL 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55:                                        "addition and subtraction instructions">;
  56: 
  57: // The device has an 8-bit stack pointer (SP) register.
  58: def FeatureSmallStack
  59:     : SubtargetFeature<"smallstack", "HasSmallStack", "true",
  60:                        "The device has an 8-bit "
  61:                        "stack pointer">;
  62: 
  63: // The device potentially requires emitting rjmp that wraps across the flash
  64: // boundary.
  65: //
  66: // We enable this for devices that have exactly 8 kB of flash memory and don't
  67: // support the `jmp` instruction - with this feature enabled, we try to convert
  68: // out-of-bounds relative jumps into in-bounds by wrapping the offset, e.g.
  69: // `rjmp +5000` becomes `rjmp -3192`.
  70: def FeatureWrappingRjmp
  71:     : SubtargetFeature<"wrappingrjmp", "HasWrappingRjmp", "true",
  72:                        "The device potentially requires emitting rjmp that "
```

- EN: This range defines declarative TableGen records such as FeatureSmallStack, FeatureWrappingRjmp, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureSmallStack, FeatureWrappingRjmp 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-90

```tablegen
  73:                        "wraps across the flash boundary">;
  74: 
  75: // The device supports the 16-bit GPR pair MOVW instruction.
  76: def FeatureMOVW : SubtargetFeature<"movw", "HasMOVW", "true",
  77:                                    "The device supports the 16-bit MOVW "
  78:                                    "instruction">;
  79: 
  80: // The device supports the `LPM` instruction, with implied destination being r0.
  81: def FeatureLPM : SubtargetFeature<"lpm", "HasLPM", "true",
  82:                                   "The device supports the `LPM` instruction">;
  83: 
  84: // The device supports the `LPM Rd, Z[+] instruction.
  85: def FeatureLPMX : SubtargetFeature<"lpmx", "HasLPMX", "true",
  86:                                    "The device supports the `LPM Rd, Z[+]` "
  87:                                    "instruction">;
  88: 
  89: // The device supports the `ELPM` instruction.
  90: def FeatureELPM : SubtargetFeature<"elpm", "HasELPM", "true",
```

- EN: This range defines declarative TableGen records such as FeatureMOVW, FeatureLPM, FeatureLPMX, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureMOVW, FeatureLPM, FeatureLPMX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 91-108

```tablegen
  91:                                    "The device supports the ELPM instruction">;
  92: 
  93: // The device supports the `ELPM Rd, Z[+]` instructions.
  94: def FeatureELPMX : SubtargetFeature<"elpmx", "HasELPMX", "true",
  95:                                     "The device supports the `ELPM Rd, Z[+]` "
  96:                                     "instructions">;
  97: 
  98: // The device supports the `SPM` instruction.
  99: def FeatureSPM : SubtargetFeature<"spm", "HasSPM", "true",
 100:                                   "The device supports the `SPM` instruction">;
 101: 
 102: // The device supports the `SPM Z+` instruction.
 103: def FeatureSPMX : SubtargetFeature<"spmx", "HasSPMX", "true",
 104:                                    "The device supports the `SPM Z+` "
 105:                                    "instruction">;
 106: 
 107: // The device supports the `DES k` instruction.
 108: def FeatureDES : SubtargetFeature<"des", "HasDES", "true",
```

- EN: This range defines declarative TableGen records such as FeatureELPMX, FeatureSPM, FeatureSPMX, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureELPMX, FeatureSPM, FeatureSPMX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 109-126

```tablegen
 109:                                   "The device supports the `DES k` encryption "
 110:                                   "instruction">;
 111: 
 112: // The device supports the Read-Write-Modify instructions
 113: // XCH, LAS, LAC, and LAT.
 114: def FeatureRMW : SubtargetFeature<"rmw", "SupportsRMW", "true",
 115:                                   "The device supports the read-write-modify "
 116:                                   "instructions: XCH, LAS, LAC, LAT">;
 117: 
 118: // The device supports the `[F]MUL[S][U]` family of instructions.
 119: def FeatureMultiplication
 120:     : SubtargetFeature<"mul", "SupportsMultiplication", "true",
 121:                        "The device supports the "
 122:                        "multiplication instructions">;
 123: 
 124: // The device supports the `BREAK` instruction.
 125: def FeatureBREAK : SubtargetFeature<"break", "HasBREAK", "true",
 126:                                     "The device supports the `BREAK` debugging "
```

- EN: This range defines declarative TableGen records such as FeatureRMW, FeatureMultiplication, FeatureBREAK, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureRMW, FeatureMultiplication, FeatureBREAK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 127-144

```tablegen
 127:                                     "instruction">;
 128: 
 129: // The device has instruction encodings specific to the Tiny core.
 130: def FeatureTinyEncoding
 131:     : SubtargetFeature<"tinyencoding", "HasTinyEncoding", "true",
 132:                        "The device has Tiny core specific "
 133:                        "instruction encodings">;
 134: 
 135: // When writing a 16-bit port or storing a 16-bit word, do the low byte first.
 136: def FeatureLowByteFirst
 137:     : SubtargetFeature<"lowbytefirst", "HasLowByteFirst", "true",
 138:                        "Do the low byte first when writing a 16-bit port or "
 139:                        "storing a 16-bit word">;
 140: 
 141: // The device has CPU registers mapped in data address space
 142: def FeatureMMR : SubtargetFeature<"memmappedregs", "HasMemMappedGPR", "true",
 143:                                   "The device has CPU registers "
 144:                                   "mapped in data address space">;
```

- EN: This range defines declarative TableGen records such as FeatureTinyEncoding, FeatureLowByteFirst, FeatureMMR, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureTinyEncoding, FeatureLowByteFirst, FeatureMMR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 145-162

```tablegen
 145: 
 146: class ELFArch<string name>
 147:     : SubtargetFeature<"", "ELFArch", !strconcat("ELF::", name), "">;
 148: 
 149: // ELF e_flags architecture values
 150: def ELFArchAVR1 : ELFArch<"EF_AVR_ARCH_AVR1">;
 151: def ELFArchAVR2 : ELFArch<"EF_AVR_ARCH_AVR2">;
 152: def ELFArchAVR25 : ELFArch<"EF_AVR_ARCH_AVR25">;
 153: def ELFArchAVR3 : ELFArch<"EF_AVR_ARCH_AVR3">;
 154: def ELFArchAVR31 : ELFArch<"EF_AVR_ARCH_AVR31">;
 155: def ELFArchAVR35 : ELFArch<"EF_AVR_ARCH_AVR35">;
 156: def ELFArchAVR4 : ELFArch<"EF_AVR_ARCH_AVR4">;
 157: def ELFArchAVR5 : ELFArch<"EF_AVR_ARCH_AVR5">;
 158: def ELFArchAVR51 : ELFArch<"EF_AVR_ARCH_AVR51">;
 159: def ELFArchAVR6 : ELFArch<"EF_AVR_ARCH_AVR6">;
 160: def ELFArchTiny : ELFArch<"EF_AVR_ARCH_AVRTINY">;
 161: def ELFArchXMEGA1 : ELFArch<"EF_AVR_ARCH_XMEGA1">;
 162: def ELFArchXMEGA2 : ELFArch<"EF_AVR_ARCH_XMEGA2">;
```

- EN: This range defines declarative TableGen records such as ELFArch, ELFArchAVR1, ELFArchAVR2, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ELFArch, ELFArchAVR1, ELFArchAVR2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 163-180

```tablegen
 163: def ELFArchXMEGA3 : ELFArch<"EF_AVR_ARCH_XMEGA3">;
 164: def ELFArchXMEGA4 : ELFArch<"EF_AVR_ARCH_XMEGA4">;
 165: def ELFArchXMEGA5 : ELFArch<"EF_AVR_ARCH_XMEGA5">;
 166: def ELFArchXMEGA6 : ELFArch<"EF_AVR_ARCH_XMEGA6">;
 167: def ELFArchXMEGA7 : ELFArch<"EF_AVR_ARCH_XMEGA7">;
 168: 
 169: //===---------------------------------------------------------------------===//
 170: // AVR Families
 171: //===---------------------------------------------------------------------===//
 172: 
 173: // The device has at least the bare minimum that **every** single AVR
 174: // device should have.
 175: def FamilyAVR0 : Family<"avr0", []>;
 176: 
 177: def FamilyAVR1 : Family<"avr1", [FamilyAVR0, FeatureLPM, FeatureMMR]>;
 178: 
 179: def FamilyAVR2
 180:     : Family<"avr2",
```

- EN: This range defines declarative TableGen records such as ELFArchXMEGA3, ELFArchXMEGA4, ELFArchXMEGA5, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ELFArchXMEGA3, ELFArchXMEGA4, ELFArchXMEGA5 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 181-198

```tablegen
 181:              [FamilyAVR1, FeatureIJMPCALL, FeatureADDSUBIW, FeatureSRAM]>;
 182: 
 183: def FamilyAVR25
 184:     : Family<"avr25",
 185:              [FamilyAVR2, FeatureMOVW, FeatureLPMX, FeatureSPM, FeatureBREAK]>;
 186: 
 187: def FamilyAVR3 : Family<"avr3", [FamilyAVR2, FeatureJMPCALL]>;
 188: 
 189: def FamilyAVR31 : Family<"avr31", [FamilyAVR3, FeatureELPM]>;
 190: 
 191: def FamilyAVR35
 192:     : Family<"avr35",
 193:              [FamilyAVR3, FeatureMOVW, FeatureLPMX, FeatureSPM, FeatureBREAK]>;
 194: 
 195: def FamilyAVR4 : Family<"avr4",
 196:                         [FamilyAVR2, FeatureMultiplication, FeatureMOVW,
 197:                          FeatureLPMX, FeatureSPM, FeatureBREAK]>;
 198: 
```

- EN: This range defines declarative TableGen records such as FamilyAVR25, FamilyAVR3, FamilyAVR31, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FamilyAVR25, FamilyAVR3, FamilyAVR31 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 199-216

```tablegen
 199: def FamilyAVR5 : Family<"avr5",
 200:                         [FamilyAVR3, FeatureMultiplication, FeatureMOVW,
 201:                          FeatureLPMX, FeatureSPM, FeatureBREAK]>;
 202: 
 203: def FamilyAVR51 : Family<"avr51", [FamilyAVR5, FeatureELPM, FeatureELPMX]>;
 204: 
 205: def FamilyAVR6 : Family<"avr6", [FamilyAVR51, FeatureEIJMPCALL]>;
 206: 
 207: def FamilyTiny
 208:     : Family<"avrtiny",
 209:              [FamilyAVR0, FeatureBREAK, FeatureSRAM, FeatureTinyEncoding,
 210:               FeatureSmallStack]>;
 211: 
 212: def FamilyXMEGA2 : Family<"xmega2",
 213:                           [FamilyAVR0, FeatureLPM, FeatureIJMPCALL,
 214:                            FeatureADDSUBIW, FeatureSRAM, FeatureJMPCALL,
 215:                            FeatureMultiplication, FeatureMOVW, FeatureLPMX,
 216:                            FeatureSPM, FeatureSPMX,
```

- EN: This range defines declarative TableGen records such as FamilyAVR5, FamilyAVR51, FamilyAVR6, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FamilyAVR5, FamilyAVR51, FamilyAVR6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 217-234

```tablegen
 217:                            FeatureBREAK, FeatureLowByteFirst]>;
 218: 
 219: def FamilyXMEGA3 : Family<"xmega3",
 220:                           [FamilyAVR0, FeatureLPM, FeatureIJMPCALL,
 221:                            FeatureADDSUBIW, FeatureSRAM, FeatureJMPCALL,
 222:                            FeatureMultiplication, FeatureMOVW, FeatureLPMX,
 223:                            FeatureBREAK, FeatureLowByteFirst]>;
 224: 
 225: def FamilyXMEGA4 : Family<"xmega4",
 226:                           [FamilyAVR0, FeatureLPM, FeatureIJMPCALL,
 227:                            FeatureADDSUBIW, FeatureSRAM, FeatureJMPCALL,
 228:                            FeatureMultiplication, FeatureMOVW, FeatureLPMX,
 229:                            FeatureELPM, FeatureELPMX,
 230:                            FeatureSPM, FeatureSPMX,
 231:                            FeatureBREAK, FeatureLowByteFirst]>;
 232: 
 233: def FamilyXMEGA : Family<"xmega",
 234:                          [FamilyAVR0, FeatureLPM, FeatureIJMPCALL,
```

- EN: This range defines declarative TableGen records such as FamilyXMEGA3, FamilyXMEGA4, FamilyXMEGA, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FamilyXMEGA3, FamilyXMEGA4, FamilyXMEGA 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 235-252

```tablegen
 235:                           FeatureADDSUBIW, FeatureSRAM, FeatureJMPCALL,
 236:                           FeatureMultiplication, FeatureMOVW, FeatureLPMX,
 237:                           FeatureSPM, FeatureBREAK, FeatureEIJMPCALL,
 238:                           FeatureSPMX, FeatureDES, FeatureELPM, FeatureELPMX,
 239:                           FeatureLowByteFirst]>;
 240: 
 241: def FamilyXMEGAU : Family<"xmegau", [FamilyXMEGA, FeatureRMW]>;
 242: 
 243: def FeatureSetSpecial
 244:     : FeatureSet<"special",
 245:                  "Enable use of the entire instruction "
 246:                  "set - used for debugging",
 247:                  [
 248:                    FeatureSRAM, FeatureJMPCALL, FeatureIJMPCALL,
 249:                    FeatureEIJMPCALL, FeatureADDSUBIW, FeatureMOVW, FeatureLPM,
 250:                    FeatureLPMX, FeatureELPM, FeatureELPMX, FeatureSPM,
 251:                    FeatureSPMX, FeatureDES, FeatureRMW, FeatureMultiplication,
 252:                    FeatureBREAK, FeatureMMR
```

- EN: This range defines declarative TableGen records such as FamilyXMEGAU, FeatureSetSpecial, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FamilyXMEGAU, FeatureSetSpecial 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 253-270

```tablegen
 253:                  ]>;
 254: 
 255: //===---------------------------------------------------------------------===//
 256: // AVR microcontrollers supported.
 257: //===---------------------------------------------------------------------===//
 258: 
 259: class Device<string Name, Family Fam, ELFArch Arch,
 260:              list<SubtargetFeature> ExtraFeatures = []>
 261:     : Processor<Name, NoItineraries, !listconcat([Fam, Arch], ExtraFeatures)>;
 262: 
 263: // Generic MCUs
 264: // Note that several versions of GCC has strange ELF architecture
 265: // settings for backwards compatibility - see `gas/config/tc-avr.c`
 266: // in AVR binutils. We do not replicate this.
 267: def : Device<"avr1", FamilyAVR1, ELFArchAVR1>;
 268: def : Device<"avr2", FamilyAVR2, ELFArchAVR2>;
 269: def : Device<"avr25", FamilyAVR25, ELFArchAVR25>;
 270: def : Device<"avr3", FamilyAVR3, ELFArchAVR3>;
```

- EN: This range defines declarative TableGen records such as Device, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Device 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 271-288

```tablegen
 271: def : Device<"avr31", FamilyAVR31, ELFArchAVR31>;
 272: def : Device<"avr35", FamilyAVR35, ELFArchAVR35>;
 273: def : Device<"avr4", FamilyAVR4, ELFArchAVR4>;
 274: def : Device<"avr5", FamilyAVR5, ELFArchAVR5>;
 275: def : Device<"avr51", FamilyAVR51, ELFArchAVR51>;
 276: def : Device<"avr6", FamilyAVR6, ELFArchAVR6>;
 277: def : Device<"avrxmega1", FamilyXMEGA, ELFArchXMEGA1>;
 278: def : Device<"avrxmega2", FamilyXMEGA, ELFArchXMEGA2>;
 279: def : Device<"avrxmega3", FamilyXMEGA3, ELFArchXMEGA3>;
 280: def : Device<"avrxmega4", FamilyXMEGA, ELFArchXMEGA4>;
 281: def : Device<"avrxmega5", FamilyXMEGA, ELFArchXMEGA5>;
 282: def : Device<"avrxmega6", FamilyXMEGA, ELFArchXMEGA6>;
 283: def : Device<"avrxmega7", FamilyXMEGA, ELFArchXMEGA7>;
 284: def : Device<"avrtiny", FamilyTiny, ELFArchTiny>;
 285: 
 286: // Specific MCUs
 287: // NOTE: This list has been synchronized with gcc-avr 5.4.0 and avr-libc 2.0.0.
 288: def : Device<"at90s1200", FamilyAVR0, ELFArchAVR1, [FeatureSmallStack]>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 289-306

```tablegen
 289: def : Device<"attiny11", FamilyAVR1, ELFArchAVR1, [FeatureSmallStack]>;
 290: def : Device<"attiny12", FamilyAVR1, ELFArchAVR1, [FeatureSmallStack]>;
 291: def : Device<"attiny15", FamilyAVR1, ELFArchAVR1, [FeatureSmallStack]>;
 292: def : Device<"attiny28", FamilyAVR1, ELFArchAVR1, [FeatureSmallStack]>;
 293: def : Device<"at90s2313", FamilyAVR2, ELFArchAVR2, [FeatureSmallStack]>;
 294: def : Device<"at90s2323", FamilyAVR2, ELFArchAVR2, [FeatureSmallStack]>;
 295: def : Device<"at90s2333", FamilyAVR2, ELFArchAVR2, [FeatureSmallStack]>;
 296: def : Device<"at90s2343", FamilyAVR2, ELFArchAVR2, [FeatureSmallStack]>;
 297: def : Device<"attiny22", FamilyAVR2, ELFArchAVR2, [FeatureSmallStack]>;
 298: def : Device<"attiny26", FamilyAVR2, ELFArchAVR2,
 299:              [FeatureLPMX, FeatureSmallStack]>;
 300: def : Device<"at86rf401", FamilyAVR2, ELFArchAVR25, [FeatureMOVW, FeatureLPMX]>;
 301: def : Device<"at90s4414", FamilyAVR2, ELFArchAVR2, [FeatureSmallStack]>;
 302: def : Device<"at90s4433", FamilyAVR2, ELFArchAVR2, [FeatureSmallStack]>;
 303: def : Device<"at90s4434", FamilyAVR2, ELFArchAVR2, [FeatureSmallStack]>;
 304: def : Device<"at90s8515", FamilyAVR2, ELFArchAVR2, [FeatureWrappingRjmp]>;
 305: def : Device<"at90c8534", FamilyAVR2, ELFArchAVR2, [FeatureWrappingRjmp]>;
 306: def : Device<"at90s8535", FamilyAVR2, ELFArchAVR2, [FeatureWrappingRjmp]>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 307-324

```tablegen
 307: def : Device<"ata5272", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 308: def : Device<"ata6616c", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 309: def : Device<"attiny13", FamilyAVR25, ELFArchAVR25, [FeatureSmallStack]>;
 310: def : Device<"attiny13a", FamilyAVR25, ELFArchAVR25, [FeatureSmallStack]>;
 311: def : Device<"attiny2313", FamilyAVR25, ELFArchAVR25, [FeatureSmallStack]>;
 312: def : Device<"attiny2313a", FamilyAVR25, ELFArchAVR25, [FeatureSmallStack]>;
 313: def : Device<"attiny24", FamilyAVR25, ELFArchAVR25, [FeatureSmallStack]>;
 314: def : Device<"attiny24a", FamilyAVR25, ELFArchAVR25, [FeatureSmallStack]>;
 315: def : Device<"attiny4313", FamilyAVR25, ELFArchAVR25>;
 316: def : Device<"attiny44", FamilyAVR25, ELFArchAVR25>;
 317: def : Device<"attiny44a", FamilyAVR25, ELFArchAVR25>;
 318: def : Device<"attiny84", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 319: def : Device<"attiny84a", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 320: def : Device<"attiny25", FamilyAVR25, ELFArchAVR25, [FeatureSmallStack]>;
 321: def : Device<"attiny45", FamilyAVR25, ELFArchAVR25>;
 322: def : Device<"attiny85", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 323: def : Device<"attiny261", FamilyAVR25, ELFArchAVR25, [FeatureSmallStack]>;
 324: def : Device<"attiny261a", FamilyAVR25, ELFArchAVR25, [FeatureSmallStack]>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 325-342

```tablegen
 325: def : Device<"attiny441", FamilyAVR25, ELFArchAVR25>;
 326: def : Device<"attiny461", FamilyAVR25, ELFArchAVR25>;
 327: def : Device<"attiny461a", FamilyAVR25, ELFArchAVR25>;
 328: def : Device<"attiny841", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 329: def : Device<"attiny861", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 330: def : Device<"attiny861a", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 331: def : Device<"attiny87", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 332: def : Device<"attiny43u", FamilyAVR25, ELFArchAVR25>;
 333: def : Device<"attiny48", FamilyAVR25, ELFArchAVR25>;
 334: def : Device<"attiny88", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 335: def : Device<"attiny828", FamilyAVR25, ELFArchAVR25, [FeatureWrappingRjmp]>;
 336: def : Device<"at43usb355", FamilyAVR3, ELFArchAVR3>;
 337: def : Device<"at76c711", FamilyAVR3, ELFArchAVR3>;
 338: def : Device<"atmega103", FamilyAVR31, ELFArchAVR31>;
 339: def : Device<"at43usb320", FamilyAVR31, ELFArchAVR31>;
 340: def : Device<"attiny167", FamilyAVR35, ELFArchAVR35>;
 341: def : Device<"at90usb82", FamilyAVR35, ELFArchAVR35>;
 342: def : Device<"at90usb162", FamilyAVR35, ELFArchAVR35>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 343-360

```tablegen
 343: def : Device<"ata5505", FamilyAVR35, ELFArchAVR35>;
 344: def : Device<"ata6617c", FamilyAVR35, ELFArchAVR35>;
 345: def : Device<"ata664251", FamilyAVR35, ELFArchAVR35>;
 346: def : Device<"atmega8u2", FamilyAVR35, ELFArchAVR35>;
 347: def : Device<"atmega16u2", FamilyAVR35, ELFArchAVR35>;
 348: def : Device<"atmega32u2", FamilyAVR35, ELFArchAVR35>;
 349: def : Device<"attiny1634", FamilyAVR35, ELFArchAVR35>;
 350: def : Device<"atmega8", FamilyAVR2, ELFArchAVR4,
 351:              [FeatureMultiplication, FeatureMOVW, FeatureLPMX, FeatureSPM, FeatureWrappingRjmp]>;
 352: def : Device<"ata6289", FamilyAVR4, ELFArchAVR4>;
 353: def : Device<"atmega8a", FamilyAVR2, ELFArchAVR4,
 354:              [FeatureMultiplication, FeatureMOVW, FeatureLPMX, FeatureSPM, FeatureWrappingRjmp]>;
 355: def : Device<"ata6285", FamilyAVR4, ELFArchAVR4, [FeatureWrappingRjmp]>;
 356: def : Device<"ata6286", FamilyAVR4, ELFArchAVR4>;
 357: def : Device<"ata6612c", FamilyAVR4, ELFArchAVR4>;
 358: def : Device<"atmega48", FamilyAVR4, ELFArchAVR4>;
 359: def : Device<"atmega48a", FamilyAVR4, ELFArchAVR4>;
 360: def : Device<"atmega48pa", FamilyAVR4, ELFArchAVR4>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 361-378

```tablegen
 361: def : Device<"atmega48pb", FamilyAVR4, ELFArchAVR4>;
 362: def : Device<"atmega48p", FamilyAVR4, ELFArchAVR4>;
 363: def : Device<"atmega88", FamilyAVR4, ELFArchAVR4>;
 364: def : Device<"atmega88a", FamilyAVR4, ELFArchAVR4>;
 365: def : Device<"atmega88p", FamilyAVR4, ELFArchAVR4>;
 366: def : Device<"atmega88pa", FamilyAVR4, ELFArchAVR4>;
 367: def : Device<"atmega88pb", FamilyAVR4, ELFArchAVR4>;
 368: def : Device<"atmega8515", FamilyAVR2, ELFArchAVR4,
 369:              [FeatureMultiplication, FeatureMOVW, FeatureLPMX, FeatureSPM, FeatureWrappingRjmp]>;
 370: def : Device<"atmega8535", FamilyAVR2, ELFArchAVR4,
 371:              [FeatureMultiplication, FeatureMOVW, FeatureLPMX, FeatureSPM, FeatureWrappingRjmp]>;
 372: def : Device<"atmega8hva", FamilyAVR4, ELFArchAVR4>;
 373: def : Device<"at90pwm1", FamilyAVR4, ELFArchAVR4>;
 374: def : Device<"at90pwm2", FamilyAVR4, ELFArchAVR4>;
 375: def : Device<"at90pwm2b", FamilyAVR4, ELFArchAVR4>;
 376: def : Device<"at90pwm3", FamilyAVR4, ELFArchAVR4>;
 377: def : Device<"at90pwm3b", FamilyAVR4, ELFArchAVR4>;
 378: def : Device<"at90pwm81", FamilyAVR4, ELFArchAVR4>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 379-396

```tablegen
 379: def : Device<"ata5702m322", FamilyAVR5, ELFArchAVR5>;
 380: def : Device<"ata5782", FamilyAVR5, ELFArchAVR5>;
 381: def : Device<"ata5790", FamilyAVR5, ELFArchAVR5>;
 382: def : Device<"ata5790n", FamilyAVR5, ELFArchAVR5>;
 383: def : Device<"ata5791", FamilyAVR5, ELFArchAVR5>;
 384: def : Device<"ata5795", FamilyAVR5, ELFArchAVR5>;
 385: def : Device<"ata5831", FamilyAVR5, ELFArchAVR5>;
 386: def : Device<"ata6613c", FamilyAVR5, ELFArchAVR5>;
 387: def : Device<"ata6614q", FamilyAVR5, ELFArchAVR5>;
 388: def : Device<"ata8210", FamilyAVR5, ELFArchAVR5>;
 389: def : Device<"ata8510", FamilyAVR5, ELFArchAVR5>;
 390: def : Device<"atmega16", FamilyAVR5, ELFArchAVR5>;
 391: def : Device<"atmega16a", FamilyAVR5, ELFArchAVR5>;
 392: def : Device<"atmega161", FamilyAVR3, ELFArchAVR5,
 393:              [FeatureMultiplication, FeatureMOVW, FeatureLPMX, FeatureSPM]>;
 394: def : Device<"atmega162", FamilyAVR5, ELFArchAVR5>;
 395: def : Device<"atmega163", FamilyAVR3, ELFArchAVR5,
 396:              [FeatureMultiplication, FeatureMOVW, FeatureLPMX, FeatureSPM]>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 397-414

```tablegen
 397: def : Device<"atmega164a", FamilyAVR5, ELFArchAVR5>;
 398: def : Device<"atmega164p", FamilyAVR5, ELFArchAVR5>;
 399: def : Device<"atmega164pa", FamilyAVR5, ELFArchAVR5>;
 400: def : Device<"atmega165", FamilyAVR5, ELFArchAVR5>;
 401: def : Device<"atmega165a", FamilyAVR5, ELFArchAVR5>;
 402: def : Device<"atmega165p", FamilyAVR5, ELFArchAVR5>;
 403: def : Device<"atmega165pa", FamilyAVR5, ELFArchAVR5>;
 404: def : Device<"atmega168", FamilyAVR5, ELFArchAVR5>;
 405: def : Device<"atmega168a", FamilyAVR5, ELFArchAVR5>;
 406: def : Device<"atmega168p", FamilyAVR5, ELFArchAVR5>;
 407: def : Device<"atmega168pa", FamilyAVR5, ELFArchAVR5>;
 408: def : Device<"atmega168pb", FamilyAVR5, ELFArchAVR5>;
 409: def : Device<"atmega169", FamilyAVR5, ELFArchAVR5>;
 410: def : Device<"atmega169a", FamilyAVR5, ELFArchAVR5>;
 411: def : Device<"atmega169p", FamilyAVR5, ELFArchAVR5>;
 412: def : Device<"atmega169pa", FamilyAVR5, ELFArchAVR5>;
 413: def : Device<"atmega32", FamilyAVR5, ELFArchAVR5>;
 414: def : Device<"atmega32a", FamilyAVR5, ELFArchAVR5>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 415-432

```tablegen
 415: def : Device<"atmega323", FamilyAVR5, ELFArchAVR5>;
 416: def : Device<"atmega324a", FamilyAVR5, ELFArchAVR5>;
 417: def : Device<"atmega324p", FamilyAVR5, ELFArchAVR5>;
 418: def : Device<"atmega324pa", FamilyAVR5, ELFArchAVR5>;
 419: def : Device<"atmega324pb", FamilyAVR5, ELFArchAVR5>;
 420: def : Device<"atmega325", FamilyAVR5, ELFArchAVR5>;
 421: def : Device<"atmega325a", FamilyAVR5, ELFArchAVR5>;
 422: def : Device<"atmega325p", FamilyAVR5, ELFArchAVR5>;
 423: def : Device<"atmega325pa", FamilyAVR5, ELFArchAVR5>;
 424: def : Device<"atmega3250", FamilyAVR5, ELFArchAVR5>;
 425: def : Device<"atmega3250a", FamilyAVR5, ELFArchAVR5>;
 426: def : Device<"atmega3250p", FamilyAVR5, ELFArchAVR5>;
 427: def : Device<"atmega3250pa", FamilyAVR5, ELFArchAVR5>;
 428: def : Device<"atmega328", FamilyAVR5, ELFArchAVR5>;
 429: def : Device<"atmega328p", FamilyAVR5, ELFArchAVR5>;
 430: def : Device<"atmega328pb", FamilyAVR5, ELFArchAVR5>;
 431: def : Device<"atmega329", FamilyAVR5, ELFArchAVR5>;
 432: def : Device<"atmega329a", FamilyAVR5, ELFArchAVR5>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 433-450

```tablegen
 433: def : Device<"atmega329p", FamilyAVR5, ELFArchAVR5>;
 434: def : Device<"atmega329pa", FamilyAVR5, ELFArchAVR5>;
 435: def : Device<"atmega3290", FamilyAVR5, ELFArchAVR5>;
 436: def : Device<"atmega3290a", FamilyAVR5, ELFArchAVR5>;
 437: def : Device<"atmega3290p", FamilyAVR5, ELFArchAVR5>;
 438: def : Device<"atmega3290pa", FamilyAVR5, ELFArchAVR5>;
 439: def : Device<"atmega406", FamilyAVR5, ELFArchAVR5>;
 440: def : Device<"atmega64", FamilyAVR5, ELFArchAVR5>;
 441: def : Device<"atmega64a", FamilyAVR5, ELFArchAVR5>;
 442: def : Device<"atmega640", FamilyAVR5, ELFArchAVR5>;
 443: def : Device<"atmega644", FamilyAVR5, ELFArchAVR5>;
 444: def : Device<"atmega644a", FamilyAVR5, ELFArchAVR5>;
 445: def : Device<"atmega644p", FamilyAVR5, ELFArchAVR5>;
 446: def : Device<"atmega644pa", FamilyAVR5, ELFArchAVR5>;
 447: def : Device<"atmega645", FamilyAVR5, ELFArchAVR5>;
 448: def : Device<"atmega645a", FamilyAVR5, ELFArchAVR5>;
 449: def : Device<"atmega645p", FamilyAVR5, ELFArchAVR5>;
 450: def : Device<"atmega649", FamilyAVR5, ELFArchAVR5>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 451-468

```tablegen
 451: def : Device<"atmega649a", FamilyAVR5, ELFArchAVR5>;
 452: def : Device<"atmega649p", FamilyAVR5, ELFArchAVR5>;
 453: def : Device<"atmega6450", FamilyAVR5, ELFArchAVR5>;
 454: def : Device<"atmega6450a", FamilyAVR5, ELFArchAVR5>;
 455: def : Device<"atmega6450p", FamilyAVR5, ELFArchAVR5>;
 456: def : Device<"atmega6490", FamilyAVR5, ELFArchAVR5>;
 457: def : Device<"atmega6490a", FamilyAVR5, ELFArchAVR5>;
 458: def : Device<"atmega6490p", FamilyAVR5, ELFArchAVR5>;
 459: def : Device<"atmega64rfr2", FamilyAVR5, ELFArchAVR5>;
 460: def : Device<"atmega644rfr2", FamilyAVR5, ELFArchAVR5>;
 461: def : Device<"atmega16hva", FamilyAVR5, ELFArchAVR5>;
 462: def : Device<"atmega16hva2", FamilyAVR5, ELFArchAVR5>;
 463: def : Device<"atmega16hvb", FamilyAVR5, ELFArchAVR5>;
 464: def : Device<"atmega16hvbrevb", FamilyAVR5, ELFArchAVR5>;
 465: def : Device<"atmega32hvb", FamilyAVR5, ELFArchAVR5>;
 466: def : Device<"atmega32hvbrevb", FamilyAVR5, ELFArchAVR5>;
 467: def : Device<"atmega64hve", FamilyAVR5, ELFArchAVR5>;
 468: def : Device<"atmega64hve2", FamilyAVR5, ELFArchAVR5>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 469-486

```tablegen
 469: def : Device<"at90can32", FamilyAVR5, ELFArchAVR5>;
 470: def : Device<"at90can64", FamilyAVR5, ELFArchAVR5>;
 471: def : Device<"at90pwm161", FamilyAVR5, ELFArchAVR5>;
 472: def : Device<"at90pwm216", FamilyAVR5, ELFArchAVR5>;
 473: def : Device<"at90pwm316", FamilyAVR5, ELFArchAVR5>;
 474: def : Device<"atmega32c1", FamilyAVR5, ELFArchAVR5>;
 475: def : Device<"atmega64c1", FamilyAVR5, ELFArchAVR5>;
 476: def : Device<"atmega16m1", FamilyAVR5, ELFArchAVR5>;
 477: def : Device<"atmega32m1", FamilyAVR5, ELFArchAVR5>;
 478: def : Device<"atmega64m1", FamilyAVR5, ELFArchAVR5>;
 479: def : Device<"atmega16u4", FamilyAVR5, ELFArchAVR5>;
 480: def : Device<"atmega32u4", FamilyAVR5, ELFArchAVR5>;
 481: def : Device<"atmega32u6", FamilyAVR5, ELFArchAVR5>;
 482: def : Device<"at90usb646", FamilyAVR5, ELFArchAVR5>;
 483: def : Device<"at90usb647", FamilyAVR5, ELFArchAVR5>;
 484: def : Device<"at90scr100", FamilyAVR5, ELFArchAVR5>;
 485: def : Device<"at94k", FamilyAVR3, ELFArchAVR5,
 486:              [FeatureMultiplication, FeatureMOVW, FeatureLPMX]>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 487-504

```tablegen
 487: def : Device<"m3000", FamilyAVR5, ELFArchAVR5>;
 488: def : Device<"atmega128", FamilyAVR51, ELFArchAVR51>;
 489: def : Device<"atmega128a", FamilyAVR51, ELFArchAVR51>;
 490: def : Device<"atmega1280", FamilyAVR51, ELFArchAVR51>;
 491: def : Device<"atmega1281", FamilyAVR51, ELFArchAVR51>;
 492: def : Device<"atmega1284", FamilyAVR51, ELFArchAVR51>;
 493: def : Device<"atmega1284p", FamilyAVR51, ELFArchAVR51>;
 494: def : Device<"atmega128rfa1", FamilyAVR51, ELFArchAVR51>;
 495: def : Device<"atmega128rfr2", FamilyAVR51, ELFArchAVR51>;
 496: def : Device<"atmega1284rfr2", FamilyAVR51, ELFArchAVR51>;
 497: def : Device<"at90can128", FamilyAVR51, ELFArchAVR51>;
 498: def : Device<"at90usb1286", FamilyAVR51, ELFArchAVR51>;
 499: def : Device<"at90usb1287", FamilyAVR51, ELFArchAVR51>;
 500: def : Device<"atmega2560", FamilyAVR6, ELFArchAVR6>;
 501: def : Device<"atmega2561", FamilyAVR6, ELFArchAVR6>;
 502: def : Device<"atmega256rfr2", FamilyAVR6, ELFArchAVR6>;
 503: def : Device<"atmega2564rfr2", FamilyAVR6, ELFArchAVR6>;
 504: def : Device<"atxmega16a4", FamilyXMEGA, ELFArchXMEGA2>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 505-522

```tablegen
 505: def : Device<"atxmega16a4u", FamilyXMEGAU, ELFArchXMEGA2>;
 506: def : Device<"atxmega16c4", FamilyXMEGAU, ELFArchXMEGA2>;
 507: def : Device<"atxmega16d4", FamilyXMEGA, ELFArchXMEGA2>;
 508: def : Device<"atxmega32a4", FamilyXMEGA, ELFArchXMEGA2>;
 509: def : Device<"atxmega32a4u", FamilyXMEGAU, ELFArchXMEGA2>;
 510: def : Device<"atxmega32c3", FamilyXMEGAU, ELFArchXMEGA2>;
 511: def : Device<"atxmega32c4", FamilyXMEGAU, ELFArchXMEGA2>;
 512: def : Device<"atxmega32d3", FamilyXMEGA, ELFArchXMEGA2>;
 513: def : Device<"atxmega32d4", FamilyXMEGA, ELFArchXMEGA2>;
 514: def : Device<"atxmega32e5", FamilyXMEGAU, ELFArchXMEGA2>;
 515: def : Device<"atxmega16e5", FamilyXMEGAU, ELFArchXMEGA2>;
 516: def : Device<"atxmega8e5", FamilyXMEGAU, ELFArchXMEGA2>;
 517: def : Device<"atxmega64a3", FamilyXMEGA, ELFArchXMEGA4>;
 518: def : Device<"atxmega64a3u", FamilyXMEGAU, ELFArchXMEGA4>;
 519: def : Device<"atxmega64a4u", FamilyXMEGAU, ELFArchXMEGA4>;
 520: def : Device<"atxmega64b1", FamilyXMEGAU, ELFArchXMEGA4>;
 521: def : Device<"atxmega64b3", FamilyXMEGAU, ELFArchXMEGA4>;
 522: def : Device<"atxmega64c3", FamilyXMEGAU, ELFArchXMEGA4>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 523-540

```tablegen
 523: def : Device<"atxmega64d3", FamilyXMEGA, ELFArchXMEGA4>;
 524: def : Device<"atxmega64d4", FamilyXMEGA, ELFArchXMEGA4>;
 525: def : Device<"atxmega64a1", FamilyXMEGA, ELFArchXMEGA5>;
 526: def : Device<"atxmega64a1u", FamilyXMEGAU, ELFArchXMEGA5>;
 527: def : Device<"atxmega128a3", FamilyXMEGA, ELFArchXMEGA6>;
 528: def : Device<"atxmega128a3u", FamilyXMEGAU, ELFArchXMEGA6>;
 529: def : Device<"atxmega128b1", FamilyXMEGAU, ELFArchXMEGA6>;
 530: def : Device<"atxmega128b3", FamilyXMEGAU, ELFArchXMEGA6>;
 531: def : Device<"atxmega128c3", FamilyXMEGAU, ELFArchXMEGA6>;
 532: def : Device<"atxmega128d3", FamilyXMEGA, ELFArchXMEGA6>;
 533: def : Device<"atxmega128d4", FamilyXMEGA, ELFArchXMEGA6>;
 534: def : Device<"atxmega192a3", FamilyXMEGA, ELFArchXMEGA6>;
 535: def : Device<"atxmega192a3u", FamilyXMEGAU, ELFArchXMEGA6>;
 536: def : Device<"atxmega192c3", FamilyXMEGAU, ELFArchXMEGA6>;
 537: def : Device<"atxmega192d3", FamilyXMEGA, ELFArchXMEGA6>;
 538: def : Device<"atxmega256a3", FamilyXMEGA, ELFArchXMEGA6>;
 539: def : Device<"atxmega256a3u", FamilyXMEGAU, ELFArchXMEGA6>;
 540: def : Device<"atxmega256a3b", FamilyXMEGA, ELFArchXMEGA6>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 541-558

```tablegen
 541: def : Device<"atxmega256a3bu", FamilyXMEGAU, ELFArchXMEGA6>;
 542: def : Device<"atxmega256c3", FamilyXMEGAU, ELFArchXMEGA6>;
 543: def : Device<"atxmega256d3", FamilyXMEGA, ELFArchXMEGA6>;
 544: def : Device<"atxmega384c3", FamilyXMEGAU, ELFArchXMEGA6>;
 545: def : Device<"atxmega384d3", FamilyXMEGA, ELFArchXMEGA6>;
 546: def : Device<"atxmega128a1", FamilyXMEGA, ELFArchXMEGA7>;
 547: def : Device<"atxmega128a1u", FamilyXMEGAU, ELFArchXMEGA7>;
 548: def : Device<"atxmega128a4u", FamilyXMEGAU, ELFArchXMEGA7>;
 549: def : Device<"attiny4", FamilyTiny, ELFArchTiny>;
 550: def : Device<"attiny5", FamilyTiny, ELFArchTiny>;
 551: def : Device<"attiny9", FamilyTiny, ELFArchTiny>;
 552: def : Device<"attiny10", FamilyTiny, ELFArchTiny>;
 553: def : Device<"attiny20", FamilyTiny, ELFArchTiny>;
 554: def : Device<"attiny40", FamilyTiny, ELFArchTiny>;
 555: def : Device<"attiny102", FamilyTiny, ELFArchTiny>;
 556: def : Device<"attiny104", FamilyTiny, ELFArchTiny>;
 557: def : Device<"attiny202", FamilyXMEGA3, ELFArchXMEGA3>;
 558: def : Device<"attiny402", FamilyXMEGA3, ELFArchXMEGA3>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 559-576

```tablegen
 559: def : Device<"attiny204", FamilyXMEGA3, ELFArchXMEGA3>;
 560: def : Device<"attiny404", FamilyXMEGA3, ELFArchXMEGA3>;
 561: def : Device<"attiny804", FamilyXMEGA3, ELFArchXMEGA3>;
 562: def : Device<"attiny1604", FamilyXMEGA3, ELFArchXMEGA3>;
 563: def : Device<"attiny406", FamilyXMEGA3, ELFArchXMEGA3>;
 564: def : Device<"attiny806", FamilyXMEGA3, ELFArchXMEGA3>;
 565: def : Device<"attiny1606", FamilyXMEGA3, ELFArchXMEGA3>;
 566: def : Device<"attiny807", FamilyXMEGA3, ELFArchXMEGA3>;
 567: def : Device<"attiny1607", FamilyXMEGA3, ELFArchXMEGA3>;
 568: def : Device<"attiny212", FamilyXMEGA3, ELFArchXMEGA3>;
 569: def : Device<"attiny412", FamilyXMEGA3, ELFArchXMEGA3>;
 570: def : Device<"attiny214", FamilyXMEGA3, ELFArchXMEGA3>;
 571: def : Device<"attiny414", FamilyXMEGA3, ELFArchXMEGA3>;
 572: def : Device<"attiny814", FamilyXMEGA3, ELFArchXMEGA3>;
 573: def : Device<"attiny1614", FamilyXMEGA3, ELFArchXMEGA3>;
 574: def : Device<"attiny416", FamilyXMEGA3, ELFArchXMEGA3>;
 575: def : Device<"attiny816", FamilyXMEGA3, ELFArchXMEGA3>;
 576: def : Device<"attiny1616", FamilyXMEGA3, ELFArchXMEGA3>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 577-594

```tablegen
 577: def : Device<"attiny3216", FamilyXMEGA3, ELFArchXMEGA3>;
 578: def : Device<"attiny417", FamilyXMEGA3, ELFArchXMEGA3>;
 579: def : Device<"attiny817", FamilyXMEGA3, ELFArchXMEGA3>;
 580: def : Device<"attiny1617", FamilyXMEGA3, ELFArchXMEGA3>;
 581: def : Device<"attiny3217", FamilyXMEGA3, ELFArchXMEGA3>;
 582: def : Device<"attiny1624", FamilyXMEGA3, ELFArchXMEGA3>;
 583: def : Device<"attiny1626", FamilyXMEGA3, ELFArchXMEGA3>;
 584: def : Device<"attiny1627", FamilyXMEGA3, ELFArchXMEGA3>;
 585: def : Device<"attiny3224", FamilyXMEGA3, ELFArchXMEGA3>;
 586: def : Device<"attiny3226", FamilyXMEGA3, ELFArchXMEGA3>;
 587: def : Device<"attiny3227", FamilyXMEGA3, ELFArchXMEGA3>;
 588: def : Device<"atmega808", FamilyXMEGA3, ELFArchXMEGA3>;
 589: def : Device<"atmega809", FamilyXMEGA3, ELFArchXMEGA3>;
 590: def : Device<"atmega1608", FamilyXMEGA3, ELFArchXMEGA3>;
 591: def : Device<"atmega1609", FamilyXMEGA3, ELFArchXMEGA3>;
 592: def : Device<"atmega3208", FamilyXMEGA3, ELFArchXMEGA3>;
 593: def : Device<"atmega3209", FamilyXMEGA3, ELFArchXMEGA3>;
 594: def : Device<"atmega4808", FamilyXMEGA3, ELFArchXMEGA3>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 595-612

```tablegen
 595: def : Device<"atmega4809", FamilyXMEGA3, ELFArchXMEGA3>;
 596: 
 597: // Additions from gcc 14:
 598: 
 599: def : Device<"avr64da28", FamilyXMEGA2, ELFArchXMEGA2>;
 600: def : Device<"avr64da32", FamilyXMEGA2, ELFArchXMEGA2>;
 601: def : Device<"avr64da48", FamilyXMEGA2, ELFArchXMEGA2>;
 602: def : Device<"avr64da64", FamilyXMEGA2, ELFArchXMEGA2>;
 603: def : Device<"avr64db28", FamilyXMEGA2, ELFArchXMEGA2>;
 604: def : Device<"avr64db32", FamilyXMEGA2, ELFArchXMEGA2>;
 605: def : Device<"avr64db48", FamilyXMEGA2, ELFArchXMEGA2>;
 606: def : Device<"avr64db64", FamilyXMEGA2, ELFArchXMEGA2>;
 607: def : Device<"avr64dd14", FamilyXMEGA2, ELFArchXMEGA2>;
 608: def : Device<"avr64dd20", FamilyXMEGA2, ELFArchXMEGA2>;
 609: def : Device<"avr64dd28", FamilyXMEGA2, ELFArchXMEGA2>;
 610: def : Device<"avr64dd32", FamilyXMEGA2, ELFArchXMEGA2>;
 611: def : Device<"avr64du28", FamilyXMEGA2, ELFArchXMEGA2>;
 612: def : Device<"avr64du32", FamilyXMEGA2, ELFArchXMEGA2>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 613-630

```tablegen
 613: def : Device<"avr64ea28", FamilyXMEGA2, ELFArchXMEGA2>;
 614: def : Device<"avr64ea32", FamilyXMEGA2, ELFArchXMEGA2>;
 615: def : Device<"avr64ea48", FamilyXMEGA2, ELFArchXMEGA2>;
 616: def : Device<"avr64sd28", FamilyXMEGA2, ELFArchXMEGA2>;
 617: def : Device<"avr64sd32", FamilyXMEGA2, ELFArchXMEGA2>;
 618: def : Device<"avr64sd48", FamilyXMEGA2, ELFArchXMEGA2>;
 619: 
 620: def : Device<"avr16dd20", FamilyXMEGA3, ELFArchXMEGA3>;
 621: def : Device<"avr16dd28", FamilyXMEGA3, ELFArchXMEGA3>;
 622: def : Device<"avr16dd32", FamilyXMEGA3, ELFArchXMEGA3>;
 623: def : Device<"avr16du14", FamilyXMEGA3, ELFArchXMEGA3>;
 624: def : Device<"avr16du20", FamilyXMEGA3, ELFArchXMEGA3>;
 625: def : Device<"avr16du28", FamilyXMEGA3, ELFArchXMEGA3>;
 626: def : Device<"avr16du32", FamilyXMEGA3, ELFArchXMEGA3>;
 627: def : Device<"avr32da28", FamilyXMEGA3, ELFArchXMEGA3>;
 628: def : Device<"avr32da32", FamilyXMEGA3, ELFArchXMEGA3>;
 629: def : Device<"avr32da48", FamilyXMEGA3, ELFArchXMEGA3>;
 630: def : Device<"avr32db28", FamilyXMEGA3, ELFArchXMEGA3>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 631-648

```tablegen
 631: def : Device<"avr32db32", FamilyXMEGA3, ELFArchXMEGA3>;
 632: def : Device<"avr32db48", FamilyXMEGA3, ELFArchXMEGA3>;
 633: def : Device<"avr32dd14", FamilyXMEGA3, ELFArchXMEGA3>;
 634: def : Device<"avr32dd20", FamilyXMEGA3, ELFArchXMEGA3>;
 635: def : Device<"avr32dd28", FamilyXMEGA3, ELFArchXMEGA3>;
 636: def : Device<"avr32dd32", FamilyXMEGA3, ELFArchXMEGA3>;
 637: def : Device<"avr32du14", FamilyXMEGA3, ELFArchXMEGA3>;
 638: def : Device<"avr32du20", FamilyXMEGA3, ELFArchXMEGA3>;
 639: def : Device<"avr32du28", FamilyXMEGA3, ELFArchXMEGA3>;
 640: def : Device<"avr32du32", FamilyXMEGA3, ELFArchXMEGA3>;
 641: def : Device<"avr16eb14", FamilyXMEGA3, ELFArchXMEGA3>;
 642: def : Device<"avr16eb20", FamilyXMEGA3, ELFArchXMEGA3>;
 643: def : Device<"avr16eb28", FamilyXMEGA3, ELFArchXMEGA3>;
 644: def : Device<"avr16eb32", FamilyXMEGA3, ELFArchXMEGA3>;
 645: def : Device<"avr16ea28", FamilyXMEGA3, ELFArchXMEGA3>;
 646: def : Device<"avr16ea32", FamilyXMEGA3, ELFArchXMEGA3>;
 647: def : Device<"avr16ea48", FamilyXMEGA3, ELFArchXMEGA3>;
 648: def : Device<"avr32ea28", FamilyXMEGA3, ELFArchXMEGA3>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 649-661

```tablegen
 649: def : Device<"avr32ea32", FamilyXMEGA3, ELFArchXMEGA3>;
 650: def : Device<"avr32ea48", FamilyXMEGA3, ELFArchXMEGA3>;
 651: def : Device<"avr32sd20", FamilyXMEGA3, ELFArchXMEGA3>;
 652: def : Device<"avr32sd28", FamilyXMEGA3, ELFArchXMEGA3>;
 653: def : Device<"avr32sd32", FamilyXMEGA3, ELFArchXMEGA3>;
 654: def : Device<"avr128da28", FamilyXMEGA4, ELFArchXMEGA4>;
 655: def : Device<"avr128da32", FamilyXMEGA4, ELFArchXMEGA4>;
 656: def : Device<"avr128da48", FamilyXMEGA4, ELFArchXMEGA4>;
 657: def : Device<"avr128da64", FamilyXMEGA4, ELFArchXMEGA4>;
 658: def : Device<"avr128db28", FamilyXMEGA4, ELFArchXMEGA4>;
 659: def : Device<"avr128db32", FamilyXMEGA4, ELFArchXMEGA4>;
 660: def : Device<"avr128db48", FamilyXMEGA4, ELFArchXMEGA4>;
 661: def : Device<"avr128db64", FamilyXMEGA4, ELFArchXMEGA4>;
```

- EN: This range continues the implementation of the backend component described by AVRDevices.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- CPU feature modelling / CPU 特性建模
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
