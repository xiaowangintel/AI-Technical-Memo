# HexagonBaseInfo.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonBaseInfo.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file contains small standalone helper functions and enum definitions for
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonBaseInfo.h - Top level definitions for Hexagon ----*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file contains small standalone helper functions and enum definitions for
    10: // the Hexagon target useful for the compiler back-end and the MC libraries.
    11: // As such, it deliberately does not include references to LLVM core
    12: // code gen types, passes, etc..
    13: //
    14: //===----------------------------------------------------------------------===//
    15: 
    16: #ifndef LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONBASEINFO_H
    17: #define LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONBASEINFO_H
    18: 
    19: #include <cstdint>
    20: 
    21: #include "HexagonDepITypes.h"
    22: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    23: 
    24: namespace llvm {
    25: 
    26: /// HexagonII - This namespace holds all of the target specific flags that
    27: /// instruction info tracks.
    28: namespace HexagonII {
    29:   unsigned const TypeCVI_FIRST = TypeCVI_4SLOT_MPY;
    30:   unsigned const TypeCVI_LAST = TypeCVI_ZW;
    31: 
    32:   enum AddrMode {
    33:     NoAddrMode     = 0,  // No addressing mode
    34:     Absolute       = 1,  // Absolute addressing mode
    35:     AbsoluteSet    = 2,  // Absolute set addressing mode
    36:     BaseImmOffset  = 3,  // Indirect with offset
    37:     BaseLongOffset = 4,  // Indirect with long offset
    38:     BaseRegOffset  = 5,  // Indirect with register offset
    39:     PostInc        = 6   // Post increment addressing mode
    40:   };
    41: 
    42:   enum MemAccessSize {
    43:     NoMemAccess = 0,
    44:     ByteAccess,
    45:     HalfWordAccess,
    46:     WordAccess,
    47:     DoubleWordAccess,
    48:     HVXVectorAccess
    49:   };
    50: 
```
- EN: It imports headers such as cstdint, HexagonDepITypes.h, MCTargetDesc/HexagonMCTargetDesc.h, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, holds, HexagonII) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as AddrMode, MemAccessSize, which carry the state or API of this component.
- CN: 这里引入了 cstdint, HexagonDepITypes.h, MCTargetDesc/HexagonMCTargetDesc.h 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, holds, HexagonII），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 AddrMode, MemAccessSize 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   enum class RegType : uint8_t {
    52:     Unknown = 0,
    53:     QF32,
    54:     QF16,
    55:   };
    56: 
    57:   struct RegTypeInfo {
    58:     RegType Output = RegType::Unknown;
    59:     RegType Input1 = RegType::Unknown;
    60:     RegType Input2 = RegType::Unknown;
    61:     RegType Input3 = RegType::Unknown;
    62:   };
    63: 
    64:   RegTypeInfo getRegTypeInfo(unsigned Opcode);
    65: 
    66:   inline RegType getOpRegType(unsigned Opcode) {
    67:     return getRegTypeInfo(Opcode).Output;
    68:   }
    69:   inline RegType getInp1RegType(unsigned Opcode) {
    70:     return getRegTypeInfo(Opcode).Input1;
    71:   }
    72:   inline RegType getInp2RegType(unsigned Opcode) {
    73:     return getRegTypeInfo(Opcode).Input2;
    74:   }
    75:   inline RegType getInp3RegType(unsigned Opcode) {
    76:     return getRegTypeInfo(Opcode).Input3;
    77:   }
    78: 
    79:   // MCInstrDesc TSFlags
    80:   // *** Must match HexagonInstrFormat*.td ***
    81:   enum {
    82:     // This 7-bit field describes the insn type.
    83:     TypePos = 0,
    84:     TypeMask = 0x7f,
    85: 
    86:     // Solo instructions.
    87:     SoloPos = 7,
    88:     SoloMask = 0x1,
    89:     // Packed only with A or X-type instructions.
    90:     SoloAXPos = 8,
    91:     SoloAXMask = 0x1,
    92:     // Only A-type instruction in first slot or nothing.
    93:     RestrictSlot1AOKPos = 9,
    94:     RestrictSlot1AOKMask = 0x1,
    95: 
    96:     // Predicated instructions.
    97:     PredicatedPos = 10,
    98:     PredicatedMask = 0x1,
    99:     PredicatedFalsePos = 11,
   100:     PredicatedFalseMask = 0x1,
```
- EN: It declares types such as RegType, RegTypeInfo, which carry the state or API of this component. It declares or implements routines such as getRegTypeInfo, getOpRegType, getInp1RegType, getInp2RegType, ... (5 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonInstrFormat, showing how the code connects to sibling backend components.
- CN: 这里声明了 RegType, RegTypeInfo 等类型，用来承载该组件的状态或接口。 这里声明或实现了 getRegTypeInfo, getOpRegType, getInp1RegType, getInp2RegType, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonInstrFormat，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:     PredicatedNewPos = 12,
   102:     PredicatedNewMask = 0x1,
   103:     PredicateLatePos = 13,
   104:     PredicateLateMask = 0x1,
   105: 
   106:     // New-Value consumer instructions.
   107:     NewValuePos = 14,
   108:     NewValueMask = 0x1,
   109:     // New-Value producer instructions.
   110:     hasNewValuePos = 15,
   111:     hasNewValueMask = 0x1,
   112:     // Which operand consumes or produces a new value.
   113:     NewValueOpPos = 16,
   114:     NewValueOpMask = 0x7,
   115:     // Stores that can become new-value stores.
   116:     mayNVStorePos = 19,
   117:     mayNVStoreMask = 0x1,
   118:     // New-value store instructions.
   119:     NVStorePos = 20,
   120:     NVStoreMask = 0x1,
   121:     // Loads that can become current-value loads.
   122:     mayCVLoadPos = 21,
   123:     mayCVLoadMask = 0x1,
   124:     // Current-value load instructions.
   125:     CVLoadPos = 22,
   126:     CVLoadMask = 0x1,
   127: 
   128:     // Extendable insns.
   129:     ExtendablePos = 23,
   130:     ExtendableMask = 0x1,
   131:     // Insns must be extended.
   132:     ExtendedPos = 24,
   133:     ExtendedMask = 0x1,
   134:     // Which operand may be extended.
   135:     ExtendableOpPos = 25,
   136:     ExtendableOpMask = 0x7,
   137:     // Signed or unsigned range.
   138:     ExtentSignedPos = 28,
   139:     ExtentSignedMask = 0x1,
   140:     // Number of bits of range before extending operand.
   141:     ExtentBitsPos = 29,
   142:     ExtentBitsMask = 0x1f,
   143:     // Alignment power-of-two before extending operand.
   144:     ExtentAlignPos = 34,
   145:     ExtentAlignMask = 0x3,
   146: 
   147:     CofMax1Pos = 36,
   148:     CofMax1Mask = 0x1,
   149:     CofRelax1Pos = 37,
   150:     CofRelax1Mask = 0x1,
```
- EN: This range exposes interface declarations used by other Hexagon backend translation units.
- CN: 这一段暴露了供其他 Hexagon 后端编译单元使用的接口声明。

### Lines 151-200 / 第 151-200 行

```cpp
   151:     CofRelax2Pos = 38,
   152:     CofRelax2Mask = 0x1,
   153: 
   154:     RestrictNoSlot1StorePos = 39,
   155:     RestrictNoSlot1StoreMask = 0x1,
   156: 
   157:     // Addressing mode for load/store instructions.
   158:     AddrModePos = 40,
   159:     AddrModeMask = 0x7,
   160:     // Access size for load/store instructions.
   161:     MemAccessSizePos = 43,
   162:     MemAccesSizeMask = 0xf,
   163: 
   164:     // Branch predicted taken.
   165:     TakenPos = 47,
   166:     TakenMask = 0x1,
   167: 
   168:     // Floating-point instructions.
   169:     FPPos = 48,
   170:     FPMask = 0x1,
   171: 
   172:     // New-Value producer-2 instructions.
   173:     hasNewValuePos2 = 50,
   174:     hasNewValueMask2 = 0x1,
   175:     // Which operand consumes or produces a new value.
   176:     NewValueOpPos2 = 51,
   177:     NewValueOpMask2 = 0x7,
   178: 
   179:     // Accumulator instructions.
   180:     AccumulatorPos = 54,
   181:     AccumulatorMask = 0x1,
   182: 
   183:     // Complex XU, prevent xu competition by preferring slot3
   184:     PrefersSlot3Pos = 55,
   185:     PrefersSlot3Mask = 0x1,
   186: 
   187:     HasHvxTmpPos = 56,
   188:     HasHvxTmpMask = 0x1,
   189: 
   190:     CVINewPos = 58,
   191:     CVINewMask = 0x1,
   192: 
   193:     isCVIPos = 59,
   194:     isCVIMask = 0x1,
   195: 
   196:     isHVXALUPos = 60,
   197:     isHVXALUMask = 0x1,
   198: 
   199:     isHVXALU2SRCPos = 61,
   200:     isHVXALU2SRCMask = 0x1,
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```cpp
   201: 
   202:     hasUnaryRestrictionPos = 62,
   203:     hasUnaryRestrictionMask = 0x1,
   204:   };
   205: 
   206:   // *** The code above must match HexagonInstrFormat*.td *** //
   207: 
   208:   // Hexagon specific MO operand flag mask.
   209:   enum HexagonMOTargetFlagVal {
   210:     // Hexagon-specific MachineOperand target flags.
   211:     //
   212:     // When changing these, make sure to update
   213:     // getSerializableDirectMachineOperandTargetFlags and
   214:     // getSerializableBitmaskMachineOperandTargetFlags if needed.
   215:     MO_NO_FLAG,
   216: 
   217:     /// MO_PCREL - On a symbol operand, indicates a PC-relative relocation
   218:     /// Used for computing a global address for PIC compilations
   219:     MO_PCREL,
   220: 
   221:     /// MO_GOT - Indicates a GOT-relative relocation
   222:     MO_GOT,
   223: 
   224:     // Low or high part of a symbol.
   225:     MO_LO16,
   226:     MO_HI16,
   227: 
   228:     // Offset from the base of the SDA.
   229:     MO_GPREL,
   230: 
   231:     // MO_GDGOT - indicates GOT relative relocation for TLS
   232:     // GeneralDynamic method
   233:     MO_GDGOT,
   234: 
   235:     // MO_GDPLT - indicates PLT relative relocation for TLS
   236:     // GeneralDynamic method
   237:     MO_GDPLT,
   238: 
   239:     // MO_IE - indicates non PIC relocation for TLS
   240:     // Initial Executable method
   241:     MO_IE,
   242: 
   243:     // MO_IEGOT - indicates PIC relocation for TLS
   244:     // Initial Executable method
   245:     MO_IEGOT,
   246: 
   247:     // MO_TPREL - indicates relocation for TLS
   248:     // local Executable method
   249:     MO_TPREL,
   250: 
```
- EN: It declares types such as HexagonMOTargetFlagVal, which carry the state or API of this component. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrFormat, HexagonMOTargetFlagVal, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonMOTargetFlagVal 等类型，用来承载该组件的状态或接口。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrFormat, HexagonMOTargetFlagVal，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:     // HMOTF_ConstExtended
   252:     // Addendum to above, indicates a const extended op
   253:     // Can be used as a mask.
   254:     HMOTF_ConstExtended = 0x80,
   255: 
   256:     // Union of all bitmasks (currently only HMOTF_ConstExtended).
   257:     MO_Bitmasks = HMOTF_ConstExtended
   258:   };
   259: 
   260:   // Hexagon Sub-instruction classes.
   261:   enum SubInstructionGroup {
   262:     HSIG_None = 0,
   263:     HSIG_L1,
   264:     HSIG_L2,
   265:     HSIG_S1,
   266:     HSIG_S2,
   267:     HSIG_A,
   268:     HSIG_Compound
   269:   };
   270: 
   271:   // Hexagon Compound classes.
   272:   enum CompoundGroup {
   273:     HCG_None = 0,
   274:     HCG_A,
   275:     HCG_B,
   276:     HCG_C
   277:   };
   278: 
   279:   enum InstParseBits {
   280:     INST_PARSE_MASK       = 0x0000c000,
   281:     INST_PARSE_PACKET_END = 0x0000c000,
   282:     INST_PARSE_LOOP_END   = 0x00008000,
   283:     INST_PARSE_NOT_END    = 0x00004000,
   284:     INST_PARSE_DUPLEX     = 0x00000000,
   285:     INST_PARSE_EXTENDER   = 0x00000000
   286:   };
   287: 
   288:   enum InstIClassBits : unsigned {
   289:     INST_ICLASS_MASK      = 0xf0000000,
   290:     INST_ICLASS_EXTENDER  = 0x00000000,
   291:     INST_ICLASS_J_1       = 0x10000000,
   292:     INST_ICLASS_J_2       = 0x20000000,
   293:     INST_ICLASS_LD_ST_1   = 0x30000000,
   294:     INST_ICLASS_LD_ST_2   = 0x40000000,
   295:     INST_ICLASS_J_3       = 0x50000000,
   296:     INST_ICLASS_CR        = 0x60000000,
   297:     INST_ICLASS_ALU32_1   = 0x70000000,
   298:     INST_ICLASS_XTYPE_1   = 0x80000000,
   299:     INST_ICLASS_LD        = 0x90000000,
   300:     INST_ICLASS_ST        = 0xa0000000,
```
- EN: It declares types such as SubInstructionGroup, CompoundGroup, InstParseBits, InstIClassBits, which carry the state or API of this component.
- CN: 这里声明了 SubInstructionGroup, CompoundGroup, InstParseBits, InstIClassBits 等类型，用来承载该组件的状态或接口。

### Lines 301-322 / 第 301-322 行

```cpp
   301:     INST_ICLASS_ALU32_2   = 0xb0000000,
   302:     INST_ICLASS_XTYPE_2   = 0xc0000000,
   303:     INST_ICLASS_XTYPE_3   = 0xd0000000,
   304:     INST_ICLASS_XTYPE_4   = 0xe0000000,
   305:     INST_ICLASS_ALU32_3   = 0xf0000000
   306:   };
   307: 
   308:   [[maybe_unused]]
   309:   static unsigned getMemAccessSizeInBytes(MemAccessSize S) {
   310:     switch (S) {
   311:       case ByteAccess:        return 1;
   312:       case HalfWordAccess:    return 2;
   313:       case WordAccess:        return 4;
   314:       case DoubleWordAccess:  return 8;
   315:       default:                return 0;
   316:     }
   317:   }
   318: } // end namespace HexagonII
   319: 
   320: } // end namespace llvm
   321: 
   322: #endif // LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONBASEINFO_H
```
- EN: It opens namespaces (HexagonII, llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as getMemAccessSizeInBytes, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（HexagonII, llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 getMemAccessSizeInBytes 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- generated metadata tables / 生成的元数据表

## Dependencies / 依赖关系

- Direct includes / 直接包含: `cstdint, HexagonDepITypes.h, MCTargetDesc/HexagonMCTargetDesc.h`
- Hexagon symbols / Hexagon 符号: `HexagonBaseInfo, HexagonDepITypes, HexagonMCTargetDesc, HexagonII, HexagonInstrFormat, HexagonMOTargetFlagVal`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
