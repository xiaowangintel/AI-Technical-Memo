# LoongArchMatInt.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/MCTargetDesc/LoongArchMatInt.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===- LoongArchMatInt.cpp - Immediate materialisation ---------*- C++ -*--===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "LoongArchMatInt.h"
  10: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  11: #include "llvm/Support/MathExtras.h"
  12: 
  13: using namespace llvm;
  14: 
  15: LoongArchMatInt::InstSeq LoongArchMatInt::generateInstSeq(int64_t Val) {
  16:   // Val:
  17:   // |            hi32              |              lo32            |
  18:   // +-----------+------------------+------------------+-----------+
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchMatInt.h`, `LoongArchMCTargetDesc.h`, `MathExtras.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `LoongArchMatInt::generateInstSeq`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchMatInt.h`, `LoongArchMCTargetDesc.h`, `MathExtras.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `LoongArchMatInt::generateInstSeq` 等函数。

### Lines 19-36 / 第 19-36 行
```cpp
  19:   // | Highest12 |    Higher20      |       Hi20       |    Lo12   |
  20:   // +-----------+------------------+------------------+-----------+
  21:   // 63        52 51              32 31              12 11         0
  22:   //
  23:   const int64_t Highest12 = Val >> 52 & 0xFFF;
  24:   const int64_t Higher20 = Val >> 32 & 0xFFFFF;
  25:   const int64_t Hi20 = Val >> 12 & 0xFFFFF;
  26:   const int64_t Lo12 = Val & 0xFFF;
  27:   InstSeq Insts;
  28: 
  29:   // LU52I_D used for: Bits[63:52] | Bits[51:0].
  30:   if (Highest12 != 0 && SignExtend64<52>(Val) == 0) {
  31:     Insts.push_back(Inst(LoongArch::LU52I_D, SignExtend64<12>(Highest12)));
  32:     return Insts;
  33:   }
  34: 
  35:   // lo32
  36:   if (Hi20 == 0)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 37-54 / 第 37-54 行
```cpp
  37:     Insts.push_back(Inst(LoongArch::ORI, Lo12));
  38:   else if (SignExtend32<1>(Lo12 >> 11) == SignExtend32<20>(Hi20))
  39:     Insts.push_back(Inst(LoongArch::ADDI_W, SignExtend64<12>(Lo12)));
  40:   else {
  41:     Insts.push_back(Inst(LoongArch::LU12I_W, SignExtend64<20>(Hi20)));
  42:     if (Lo12 != 0)
  43:       Insts.push_back(Inst(LoongArch::ORI, Lo12));
  44:   }
  45: 
  46:   // hi32
  47:   // Higher20
  48:   if (SignExtend32<1>(Hi20 >> 19) != SignExtend32<20>(Higher20))
  49:     Insts.push_back(Inst(LoongArch::LU32I_D, SignExtend64<20>(Higher20)));
  50: 
  51:   // Highest12
  52:   if (SignExtend32<1>(Higher20 >> 19) != SignExtend32<12>(Highest12))
  53:     Insts.push_back(Inst(LoongArch::LU52I_D, SignExtend64<12>(Highest12)));
  54: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   size_t N = Insts.size();
  56:   if (N < 3)
  57:     return Insts;
  58: 
  59:   // When the number of instruction sequences is greater than 2, we have the
  60:   // opportunity to optimize using the BSTRINS_D instruction. The scenario is as
  61:   // follows:
  62:   //
  63:   // N of Insts = 3
  64:   // 1. ORI + LU32I_D + LU52I_D     =>     ORI + BSTRINS_D, TmpVal = ORI
  65:   // 2. ADDI_W + LU32I_D + LU52I_D  =>  ADDI_W + BSTRINS_D, TmpVal = ADDI_W
  66:   // 3. LU12I_W + ORI + LU32I_D     =>     ORI + BSTRINS_D, TmpVal = ORI
  67:   // 4. LU12I_W + LU32I_D + LU52I_D => LU12I_W + BSTRINS_D, TmpVal = LU12I_W
  68:   //
  69:   // N of Insts = 4
  70:   // 5. LU12I_W + ORI + LU32I_D + LU52I_D => LU12I_W + ORI + BSTRINS_D
  71:   //                                      => ORI + LU52I_D + BSTRINS_D
  72:   //    TmpVal = (LU12I_W | ORI) or (ORI | LU52I_D)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   // The BSTRINS_D instruction will use the `TmpVal` to construct the `Val`.
  74:   uint64_t TmpVal1 = 0;
  75:   uint64_t TmpVal2 = 0;
  76:   switch (Insts[0].Opc) {
  77:   default:
  78:     llvm_unreachable("unexpected opcode");
  79:     break;
  80:   case LoongArch::LU12I_W:
  81:     if (Insts[1].Opc == LoongArch::ORI) {
  82:       TmpVal1 = Insts[1].Imm;
  83:       if (N == 3)
  84:         break;
  85:       TmpVal2 = static_cast<uint64_t>(Insts[3].Imm) << 52 | TmpVal1;
  86:     }
  87:     TmpVal1 |= static_cast<uint64_t>(Insts[0].Imm) << 12;
  88:     break;
  89:   case LoongArch::ORI:
  90:   case LoongArch::ADDI_W:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     TmpVal1 = Insts[0].Imm;
  92:     break;
  93:   }
  94: 
  95:   uint64_t Msb = 32;
  96:   uint64_t HighMask = ~((1ULL << (Msb + 1)) - 1);
  97:   for (; Msb < 64; ++Msb, HighMask = HighMask << 1) {
  98:     for (uint64_t Lsb = Msb; Lsb > 0; --Lsb) {
  99:       uint64_t LowMask = (1ULL << Lsb) - 1;
 100:       uint64_t Mask = HighMask | LowMask;
 101:       uint64_t LsbToZero = TmpVal1 & ((1ULL << (Msb - Lsb + 1)) - 1);
 102:       uint64_t MsbToLsb = LsbToZero << Lsb;
 103:       if ((MsbToLsb | (TmpVal1 & Mask)) == (uint64_t)Val) {
 104:         if (Insts[1].Opc == LoongArch::ORI && N == 3)
 105:           Insts[0] = Insts[1];
 106:         Insts.pop_back_n(2);
 107:         Insts.push_back(Inst(LoongArch::BSTRINS_D, Msb << 32 | Lsb));
 108:         return Insts;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 109-125 / 第 109-125 行
```cpp
 109:       }
 110:       if (TmpVal2 != 0) {
 111:         LsbToZero = TmpVal2 & ((1ULL << (Msb - Lsb + 1)) - 1);
 112:         MsbToLsb = LsbToZero << Lsb;
 113:         if ((MsbToLsb | (TmpVal2 & Mask)) == (uint64_t)Val) {
 114:           Insts[0] = Insts[1];
 115:           Insts[1] = Insts[3];
 116:           Insts.pop_back_n(2);
 117:           Insts.push_back(Inst(LoongArch::BSTRINS_D, Msb << 32 | Lsb));
 118:           return Insts;
 119:         }
 120:       }
 121:     }
 122:   }
 123: 
 124:   return Insts;
 125: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **Target backend structure**: Shows how LLVM splits a backend into MC, CodeGen, and target-specific layers. / 展示 LLVM 如何将后端拆分为 MC、CodeGen 和目标相关层。

## Dependencies / 依赖关系
- `LoongArchMatInt.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `llvm/Support/MathExtras.h`
