# LoongArchBaseInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/MCTargetDesc/LoongArchBaseInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //= LoongArchBaseInfo.cpp - Top level definitions for LoongArch MC -*- C++ -*-//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements helper functions for the LoongArch target useful for the
  10: // compiler back-end and the MC libraries.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "LoongArchBaseInfo.h"
  15: #include "LoongArchMCTargetDesc.h"
  16: #include "llvm/Support/ErrorHandling.h"
  17: #include "llvm/Support/raw_ostream.h"
  18: #include "llvm/TargetParser/Triple.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchBaseInfo.h`, `LoongArchMCTargetDesc.h`, `ErrorHandling.h`, `raw_ostream.h`, `Triple.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchBaseInfo.h`, `LoongArchMCTargetDesc.h`, `ErrorHandling.h`, `raw_ostream.h`, `Triple.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: 
  20: namespace llvm {
  21: 
  22: namespace LoongArchABI {
  23: 
  24: // Check if ABI has been standardized; issue a warning if it hasn't.
  25: // FIXME: Once all ABIs are standardized, this will be removed.
  26: static ABI checkABIStandardized(ABI Abi) {
  27:   StringRef ABIName;
  28:   switch (Abi) {
  29:   case ABI_ILP32F:
  30:     ABIName = "ilp32f";
  31:     break;
  32:   case ABI_LP64F:
  33:     ABIName = "lp64f";
  34:     break;
  35:   case ABI_ILP32S:
  36:   case ABI_ILP32D:
```
- **EN**: The range implements or declares functions including `checkABIStandardized`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `checkABIStandardized` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   case ABI_LP64S:
  38:   case ABI_LP64D:
  39:     return Abi;
  40:   default:
  41:     llvm_unreachable("");
  42:   }
  43:   errs() << "warning: '" << ABIName << "' has not been standardized\n";
  44:   return Abi;
  45: }
  46: 
  47: static ABI getTripleABI(const Triple &TT) {
  48:   bool Is64Bit = TT.isArch64Bit();
  49:   ABI TripleABI;
  50:   switch (TT.getEnvironment()) {
  51:   case llvm::Triple::EnvironmentType::UnknownEnvironment:
  52:     TripleABI = ABI_Unknown;
  53:     break;
  54:   case llvm::Triple::EnvironmentType::GNUSF:
```
- **EN**: The range implements or declares functions including `getTripleABI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getTripleABI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   case llvm::Triple::EnvironmentType::MuslSF:
  56:     TripleABI = Is64Bit ? ABI_LP64S : ABI_ILP32S;
  57:     break;
  58:   case llvm::Triple::EnvironmentType::GNUF32:
  59:   case llvm::Triple::EnvironmentType::MuslF32:
  60:     TripleABI = Is64Bit ? ABI_LP64F : ABI_ILP32F;
  61:     break;
  62:   // Let the fallback case behave like {ILP32,LP64}D.
  63:   case llvm::Triple::EnvironmentType::GNUF64:
  64:   default:
  65:     TripleABI = Is64Bit ? ABI_LP64D : ABI_ILP32D;
  66:     break;
  67:   }
  68:   return TripleABI;
  69: }
  70: 
  71: ABI computeTargetABI(const Triple &TT, const FeatureBitset &FeatureBits,
  72:                      StringRef ABIName) {
```
- **EN**: The range implements or declares functions including `computeTargetABI`.
- **CN**: 这一段实现或声明了 `computeTargetABI` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   bool Is64Bit = TT.isArch64Bit();
  74:   ABI ArgProvidedABI = getTargetABI(ABIName);
  75:   ABI TripleABI = getTripleABI(TT);
  76: 
  77:   auto IsABIValidForFeature = [=](ABI Abi) {
  78:     switch (Abi) {
  79:     default:
  80:       return false;
  81:     case ABI_ILP32S:
  82:       return !Is64Bit;
  83:     case ABI_ILP32F:
  84:       return !Is64Bit && FeatureBits[LoongArch::FeatureBasicF];
  85:     case ABI_ILP32D:
  86:       return !Is64Bit && FeatureBits[LoongArch::FeatureBasicD];
  87:     case ABI_LP64S:
  88:       return Is64Bit;
  89:     case ABI_LP64F:
  90:       return Is64Bit && FeatureBits[LoongArch::FeatureBasicF];
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     case ABI_LP64D:
  92:       return Is64Bit && FeatureBits[LoongArch::FeatureBasicD];
  93:     }
  94:   };
  95: 
  96:   // 1. If the '-target-abi' is valid, use it.
  97:   if (IsABIValidForFeature(ArgProvidedABI)) {
  98:     if (IsABIValidForFeature(TripleABI) && ArgProvidedABI != TripleABI)
  99:       errs()
 100:           << "warning: triple-implied ABI conflicts with provided target-abi '"
 101:           << ABIName << "', using target-abi\n";
 102:     return checkABIStandardized(ArgProvidedABI);
 103:   }
 104: 
 105:   // 2. If the triple-implied ABI is valid, use it.
 106:   if (IsABIValidForFeature(TripleABI)) {
 107:     // If target-abi is not specified, use the valid triple-implied ABI.
 108:     if (ABIName.empty())
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:       return checkABIStandardized(TripleABI);
 110: 
 111:     switch (ArgProvidedABI) {
 112:     case ABI_Unknown:
 113:       // Fallback to the triple-implied ABI if ABI name is specified but
 114:       // invalid.
 115:       errs() << "warning: the '" << ABIName
 116:              << "' is not a recognized ABI for this target, ignoring and "
 117:                 "using triple-implied ABI\n";
 118:       return checkABIStandardized(TripleABI);
 119:     case ABI_ILP32S:
 120:     case ABI_ILP32F:
 121:     case ABI_ILP32D:
 122:       if (Is64Bit) {
 123:         errs() << "warning: 32-bit ABIs are not supported for 64-bit targets, "
 124:                   "ignoring and using triple-implied ABI\n";
 125:         return checkABIStandardized(TripleABI);
 126:       }
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:       break;
 128:     case ABI_LP64S:
 129:     case ABI_LP64F:
 130:     case ABI_LP64D:
 131:       if (!Is64Bit) {
 132:         errs() << "warning: 64-bit ABIs are not supported for 32-bit targets, "
 133:                   "ignoring and using triple-implied ABI\n";
 134:         return checkABIStandardized(TripleABI);
 135:       }
 136:       break;
 137:     }
 138: 
 139:     switch (ArgProvidedABI) {
 140:     case ABI_ILP32F:
 141:     case ABI_LP64F:
 142:       errs() << "warning: the '" << ABIName
 143:              << "' ABI can't be used for a target that doesn't support the 'F' "
 144:                 "instruction set, ignoring and using triple-implied ABI\n";
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:       break;
 146:     case ABI_ILP32D:
 147:     case ABI_LP64D:
 148:       errs() << "warning: the '" << ABIName
 149:              << "' ABI can't be used for a target that doesn't support the 'D' "
 150:                 "instruction set, ignoring and using triple-implied ABI\n";
 151:       break;
 152:     default:
 153:       llvm_unreachable("");
 154:     }
 155:     return checkABIStandardized(TripleABI);
 156:   }
 157: 
 158:   // 3. Parse the 'feature-abi', and use it.
 159:   auto GetFeatureABI = [=]() {
 160:     if (FeatureBits[LoongArch::FeatureBasicD])
 161:       return Is64Bit ? ABI_LP64D : ABI_ILP32D;
 162:     if (FeatureBits[LoongArch::FeatureBasicF])
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 163-180 / 第 163-180 行
```cpp
 163:       return Is64Bit ? ABI_LP64F : ABI_ILP32F;
 164:     return Is64Bit ? ABI_LP64S : ABI_ILP32S;
 165:   };
 166:   if (!ABIName.empty())
 167:     errs() << "warning: both target-abi and the triple-implied ABI are "
 168:               "invalid, ignoring and using feature-implied ABI\n";
 169:   return checkABIStandardized(GetFeatureABI());
 170: }
 171: 
 172: ABI getTargetABI(StringRef ABIName) {
 173:   auto TargetABI = StringSwitch<ABI>(ABIName)
 174:                        .Case("ilp32s", ABI_ILP32S)
 175:                        .Case("ilp32f", ABI_ILP32F)
 176:                        .Case("ilp32d", ABI_ILP32D)
 177:                        .Case("lp64s", ABI_LP64S)
 178:                        .Case("lp64f", ABI_LP64F)
 179:                        .Case("lp64d", ABI_LP64D)
 180:                        .Default(ABI_Unknown);
```
- **EN**: The range implements or declares functions including `getTargetABI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getTargetABI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-191 / 第 181-191 行
```cpp
 181:   return TargetABI;
 182: }
 183: 
 184: // To avoid the BP value clobbered by a function call, we need to choose a
 185: // callee saved register to save the value. The `last` `S` register (s9) is
 186: // used for FP. So we choose the previous (s8) as BP.
 187: MCRegister getBPReg() { return LoongArch::R31; }
 188: 
 189: } // end namespace LoongArchABI
 190: 
 191: } // end namespace llvm
```
- **EN**: The range implements or declares functions including `getBPReg`.
- **CN**: 这一段实现或声明了 `getBPReg` 等函数。

## Key Concepts / 关键概念
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `LoongArchBaseInfo.h`
- `LoongArchMCTargetDesc.h`
- `llvm/Support/ErrorHandling.h`
- `llvm/Support/raw_ostream.h`
- `llvm/TargetParser/Triple.h`
