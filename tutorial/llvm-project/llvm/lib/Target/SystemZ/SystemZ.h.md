# SystemZ.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZ.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //==- SystemZ.h - Top-Level Interface for SystemZ representation -*- C++ -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the entry points for global functions defined in
  10: // the LLVM SystemZ backend.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZ_H
  15: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZ_H
  16: 
  17: #include "MCTargetDesc/SystemZMCTargetDesc.h"
  18: #include "llvm/Support/CodeGen.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMCTargetDesc.h`, `CodeGen.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMCTargetDesc.h`, `CodeGen.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: 
  20: namespace llvm {
  21: class FunctionPass;
  22: class PassRegistry;
  23: class SystemZTargetMachine;
  24: 
  25: namespace SystemZ {
  26: // Condition-code mask values.
  27: const unsigned CCMASK_NONE = 0;
  28: const unsigned CCMASK_0 = 1 << 3;
  29: const unsigned CCMASK_1 = 1 << 2;
  30: const unsigned CCMASK_2 = 1 << 1;
  31: const unsigned CCMASK_3 = 1 << 0;
  32: const unsigned CCMASK_ANY = CCMASK_0 | CCMASK_1 | CCMASK_2 | CCMASK_3;
  33: 
  34: // Condition-code mask assignments for integer and floating-point
  35: // comparisons.
  36: const unsigned CCMASK_CMP_EQ = CCMASK_0;
```
- **EN**: This block declares or refines TableGen records such as `FunctionPass`, `PassRegistry`, `SystemZTargetMachine`.
- **CN**: 该代码块声明或细化了 `FunctionPass`, `PassRegistry`, `SystemZTargetMachine` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```cpp
  37: const unsigned CCMASK_CMP_LT = CCMASK_1;
  38: const unsigned CCMASK_CMP_GT = CCMASK_2;
  39: const unsigned CCMASK_CMP_NE = CCMASK_CMP_LT | CCMASK_CMP_GT;
  40: const unsigned CCMASK_CMP_LE = CCMASK_CMP_EQ | CCMASK_CMP_LT;
  41: const unsigned CCMASK_CMP_GE = CCMASK_CMP_EQ | CCMASK_CMP_GT;
  42: 
  43: // Condition-code mask assignments for floating-point comparisons only.
  44: const unsigned CCMASK_CMP_UO = CCMASK_3;
  45: const unsigned CCMASK_CMP_O  = CCMASK_ANY ^ CCMASK_CMP_UO;
  46: 
  47: // All condition-code values produced by comparisons.
  48: const unsigned CCMASK_ICMP = CCMASK_0 | CCMASK_1 | CCMASK_2;
  49: const unsigned CCMASK_FCMP = CCMASK_0 | CCMASK_1 | CCMASK_2 | CCMASK_3;
  50: 
  51: // Condition-code mask assignments for arithmetical operations.
  52: const unsigned CCMASK_ARITH_EQ       = CCMASK_0;
  53: const unsigned CCMASK_ARITH_LT       = CCMASK_1;
  54: const unsigned CCMASK_ARITH_GT       = CCMASK_2;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55: const unsigned CCMASK_ARITH_OVERFLOW = CCMASK_3;
  56: const unsigned CCMASK_ARITH          = CCMASK_ANY;
  57: 
  58: // Condition-code mask assignments for logical operations.
  59: const unsigned CCMASK_LOGICAL_ZERO     = CCMASK_0 | CCMASK_2;
  60: const unsigned CCMASK_LOGICAL_NONZERO  = CCMASK_1 | CCMASK_3;
  61: const unsigned CCMASK_LOGICAL_CARRY    = CCMASK_2 | CCMASK_3;
  62: const unsigned CCMASK_LOGICAL_NOCARRY  = CCMASK_0 | CCMASK_1;
  63: const unsigned CCMASK_LOGICAL_BORROW   = CCMASK_LOGICAL_NOCARRY;
  64: const unsigned CCMASK_LOGICAL_NOBORROW = CCMASK_LOGICAL_CARRY;
  65: const unsigned CCMASK_LOGICAL          = CCMASK_ANY;
  66: 
  67: // Condition-code mask assignments for CS.
  68: const unsigned CCMASK_CS_EQ = CCMASK_0;
  69: const unsigned CCMASK_CS_NE = CCMASK_1;
  70: const unsigned CCMASK_CS    = CCMASK_0 | CCMASK_1;
  71: 
  72: // Condition-code mask assignments for a completed SRST loop.
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73: const unsigned CCMASK_SRST_FOUND    = CCMASK_1;
  74: const unsigned CCMASK_SRST_NOTFOUND = CCMASK_2;
  75: const unsigned CCMASK_SRST          = CCMASK_1 | CCMASK_2;
  76: 
  77: // Condition-code mask assignments for TEST UNDER MASK.
  78: const unsigned CCMASK_TM_ALL_0       = CCMASK_0;
  79: const unsigned CCMASK_TM_MIXED_MSB_0 = CCMASK_1;
  80: const unsigned CCMASK_TM_MIXED_MSB_1 = CCMASK_2;
  81: const unsigned CCMASK_TM_ALL_1       = CCMASK_3;
  82: const unsigned CCMASK_TM_SOME_0      = CCMASK_TM_ALL_1 ^ CCMASK_ANY;
  83: const unsigned CCMASK_TM_SOME_1      = CCMASK_TM_ALL_0 ^ CCMASK_ANY;
  84: const unsigned CCMASK_TM_MSB_0       = CCMASK_0 | CCMASK_1;
  85: const unsigned CCMASK_TM_MSB_1       = CCMASK_2 | CCMASK_3;
  86: const unsigned CCMASK_TM             = CCMASK_ANY;
  87: 
  88: // Condition-code mask assignments for TRANSACTION_BEGIN.
  89: const unsigned CCMASK_TBEGIN_STARTED       = CCMASK_0;
  90: const unsigned CCMASK_TBEGIN_INDETERMINATE = CCMASK_1;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91: const unsigned CCMASK_TBEGIN_TRANSIENT     = CCMASK_2;
  92: const unsigned CCMASK_TBEGIN_PERSISTENT    = CCMASK_3;
  93: const unsigned CCMASK_TBEGIN               = CCMASK_ANY;
  94: 
  95: // Condition-code mask assignments for TRANSACTION_END.
  96: const unsigned CCMASK_TEND_TX   = CCMASK_0;
  97: const unsigned CCMASK_TEND_NOTX = CCMASK_2;
  98: const unsigned CCMASK_TEND      = CCMASK_TEND_TX | CCMASK_TEND_NOTX;
  99: 
 100: // Condition-code mask assignments for vector comparisons (and similar
 101: // operations).
 102: const unsigned CCMASK_VCMP_ALL       = CCMASK_0;
 103: const unsigned CCMASK_VCMP_MIXED     = CCMASK_1;
 104: const unsigned CCMASK_VCMP_NONE      = CCMASK_3;
 105: const unsigned CCMASK_VCMP           = CCMASK_0 | CCMASK_1 | CCMASK_3;
 106: 
 107: // Condition-code mask assignments for Test Data Class.
 108: const unsigned CCMASK_TDC_NOMATCH   = CCMASK_0;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109: const unsigned CCMASK_TDC_MATCH     = CCMASK_1;
 110: const unsigned CCMASK_TDC           = CCMASK_TDC_NOMATCH | CCMASK_TDC_MATCH;
 111: 
 112: // The position of the low CC bit in an IPM result.
 113: const unsigned IPM_CC = 28;
 114: 
 115: // Mask assignments for PFD.
 116: const unsigned PFD_READ  = 1;
 117: const unsigned PFD_WRITE = 2;
 118: 
 119: // Mask assignments for TDC
 120: const unsigned TDCMASK_ZERO_PLUS       = 0x800;
 121: const unsigned TDCMASK_ZERO_MINUS      = 0x400;
 122: const unsigned TDCMASK_NORMAL_PLUS     = 0x200;
 123: const unsigned TDCMASK_NORMAL_MINUS    = 0x100;
 124: const unsigned TDCMASK_SUBNORMAL_PLUS  = 0x080;
 125: const unsigned TDCMASK_SUBNORMAL_MINUS = 0x040;
 126: const unsigned TDCMASK_INFINITY_PLUS   = 0x020;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127: const unsigned TDCMASK_INFINITY_MINUS  = 0x010;
 128: const unsigned TDCMASK_QNAN_PLUS       = 0x008;
 129: const unsigned TDCMASK_QNAN_MINUS      = 0x004;
 130: const unsigned TDCMASK_SNAN_PLUS       = 0x002;
 131: const unsigned TDCMASK_SNAN_MINUS      = 0x001;
 132: 
 133: const unsigned TDCMASK_ZERO            = TDCMASK_ZERO_PLUS | TDCMASK_ZERO_MINUS;
 134: const unsigned TDCMASK_POSITIVE        = TDCMASK_NORMAL_PLUS |
 135:                                          TDCMASK_SUBNORMAL_PLUS |
 136:                                          TDCMASK_INFINITY_PLUS;
 137: const unsigned TDCMASK_NEGATIVE        = TDCMASK_NORMAL_MINUS |
 138:                                          TDCMASK_SUBNORMAL_MINUS |
 139:                                          TDCMASK_INFINITY_MINUS;
 140: const unsigned TDCMASK_NAN             = TDCMASK_QNAN_PLUS |
 141:                                          TDCMASK_QNAN_MINUS |
 142:                                          TDCMASK_SNAN_PLUS |
 143:                                          TDCMASK_SNAN_MINUS;
 144: const unsigned TDCMASK_PLUS            = TDCMASK_POSITIVE |
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 145-162 / 第 145-162 行
```cpp
 145:                                          TDCMASK_ZERO_PLUS |
 146:                                          TDCMASK_QNAN_PLUS |
 147:                                          TDCMASK_SNAN_PLUS;
 148: const unsigned TDCMASK_MINUS           = TDCMASK_NEGATIVE |
 149:                                          TDCMASK_ZERO_MINUS |
 150:                                          TDCMASK_QNAN_MINUS |
 151:                                          TDCMASK_SNAN_MINUS;
 152: const unsigned TDCMASK_ALL             = TDCMASK_PLUS | TDCMASK_MINUS;
 153: 
 154: // Number of bits in a vector register.
 155: const unsigned VectorBits = 128;
 156: 
 157: // Number of bytes in a vector register (and consequently the number of
 158: // bytes in a general permute vector).
 159: const unsigned VectorBytes = VectorBits / 8;
 160: 
 161: // Return true if Val fits an LLILL operand.
 162: static inline bool isImmLL(uint64_t Val) {
```
- **EN**: The range implements or declares functions including `isImmLL`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isImmLL` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   return (Val & ~0x000000000000ffffULL) == 0;
 164: }
 165: 
 166: // Return true if Val fits an LLILH operand.
 167: static inline bool isImmLH(uint64_t Val) {
 168:   return (Val & ~0x00000000ffff0000ULL) == 0;
 169: }
 170: 
 171: // Return true if Val fits an LLIHL operand.
 172: static inline bool isImmHL(uint64_t Val) {
 173:   return (Val & ~0x00000ffff00000000ULL) == 0;
 174: }
 175: 
 176: // Return true if Val fits an LLIHH operand.
 177: static inline bool isImmHH(uint64_t Val) {
 178:   return (Val & ~0xffff000000000000ULL) == 0;
 179: }
 180: 
```
- **EN**: The range implements or declares functions including `isImmLH`, `isImmHL`, `isImmHH`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isImmLH`, `isImmHL`, `isImmHH` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181: // Return true if Val fits an LLILF operand.
 182: static inline bool isImmLF(uint64_t Val) {
 183:   return (Val & ~0x00000000ffffffffULL) == 0;
 184: }
 185: 
 186: // Return true if Val fits an LLIHF operand.
 187: static inline bool isImmHF(uint64_t Val) {
 188:   return (Val & ~0xffffffff00000000ULL) == 0;
 189: }
 190: } // end namespace SystemZ
 191: 
 192: FunctionPass *createSystemZISelDag(SystemZTargetMachine &TM,
 193:                                    CodeGenOptLevel OptLevel);
 194: FunctionPass *createSystemZElimComparePass(SystemZTargetMachine &TM);
 195: FunctionPass *createSystemZShortenInstPass(SystemZTargetMachine &TM);
 196: FunctionPass *createSystemZLongBranchPass(SystemZTargetMachine &TM);
 197: FunctionPass *createSystemZLDCleanupPass(SystemZTargetMachine &TM);
 198: FunctionPass *createSystemZCopyPhysRegsPass(SystemZTargetMachine &TM);
```
- **EN**: The range implements or declares functions including `isImmLF`, `isImmHF`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isImmLF`, `isImmHF` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199: FunctionPass *createSystemZPostRewritePass(SystemZTargetMachine &TM);
 200: FunctionPass *createSystemZTDCPass();
 201: 
 202: void initializeSystemZAsmPrinterPass(PassRegistry &);
 203: void initializeSystemZCopyPhysRegsPass(PassRegistry &);
 204: void initializeSystemZDAGToDAGISelLegacyPass(PassRegistry &);
 205: void initializeSystemZElimComparePass(PassRegistry &);
 206: void initializeSystemZLDCleanupPass(PassRegistry &);
 207: void initializeSystemZLongBranchPass(PassRegistry &);
 208: void initializeSystemZPostRewritePass(PassRegistry &);
 209: void initializeSystemZShortenInstPass(PassRegistry &);
 210: void initializeSystemZTDCPassPass(PassRegistry &);
 211: 
 212: namespace SYSTEMZAS {
 213: enum : unsigned { PTR32 = 1 };
 214: } // namespace SYSTEMZAS
 215: 
 216: } // end namespace llvm
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 217-218 / 第 217-218 行
```cpp
 217: 
 218: #endif
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `MCTargetDesc/SystemZMCTargetDesc.h`
- `llvm/Support/CodeGen.h`
