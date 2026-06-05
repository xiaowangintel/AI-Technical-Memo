# M68kLegalizerInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/GISel/M68kLegalizerInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file describes legalization rules for GlobalISel for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责描述 GlobalISel 的合法化规则。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //===-- M68kLegalizerInfo.cpp -----------------------------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: /// \file
   9: /// This file implements the targeting of the Machinelegalizer class for M68k.
  10: //===----------------------------------------------------------------------===//
  11: 
  12: #include "M68kLegalizerInfo.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kLegalizerInfo.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kLegalizerInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 13-24 / 第 13-24 行
```cpp
  13: #include "llvm/CodeGen/GlobalISel/LegalizerHelper.h"
  14: #include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
  15: #include "llvm/CodeGen/TargetOpcodes.h"
  16: #include "llvm/CodeGen/ValueTypes.h"
  17: #include "llvm/IR/DerivedTypes.h"
  18: #include "llvm/IR/Type.h"
  19: 
  20: using namespace llvm;
  21: 
  22: M68kLegalizerInfo::M68kLegalizerInfo(const M68kSubtarget &ST) {
  23:   using namespace TargetOpcode;
  24:   const LLT s8 = LLT::scalar(8);
```
- **EN**: It imports dependencies such as `LegalizerHelper.h`, `LegalizerInfo.h`, `TargetOpcodes.h`, `ValueTypes.h`, `DerivedTypes.h`, `Type.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 它引入了 `LegalizerHelper.h`, `LegalizerInfo.h`, `TargetOpcodes.h`, `ValueTypes.h`, `DerivedTypes.h`, `Type.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 25-36 / 第 25-36 行
```cpp
  25:   const LLT s16 = LLT::scalar(16);
  26:   const LLT s32 = LLT::scalar(32);
  27:   const LLT p0 = LLT::pointer(0, 32);
  28: 
  29:   getActionDefinitionsBuilder({G_ADD, G_SUB, G_MUL, G_UDIV, G_AND})
  30:       .legalFor({s8, s16, s32})
  31:       .clampScalar(0, s8, s32)
  32:       .widenScalarToNextPow2(0, 8);
  33: 
  34:   getActionDefinitionsBuilder(G_CONSTANT)
  35:       .legalFor({s32, p0})
  36:       .clampScalar(0, s32, s32);
```
- **EN**: The range implements or declares functions including `getActionDefinitionsBuilder`, `getActionDefinitionsBuilder`.
- **CN**: 这一段实现或声明了 `getActionDefinitionsBuilder`, `getActionDefinitionsBuilder` 等函数。

### Lines 37-48 / 第 37-48 行
```cpp
  37: 
  38:   getActionDefinitionsBuilder({G_FRAME_INDEX, G_GLOBAL_VALUE}).legalFor({p0});
  39: 
  40:   getActionDefinitionsBuilder({G_STORE, G_LOAD})
  41:       .legalForTypesWithMemDesc({{s32, p0, s32, 4},
  42:                                  {s32, p0, s16, 4},
  43:                                  {s32, p0, s8, 4},
  44:                                  {s16, p0, s16, 2},
  45:                                  {s8, p0, s8, 1},
  46:                                  {p0, p0, s32, 4}})
  47:       .clampScalar(0, s8, s32);
  48: 
```
- **EN**: The range implements or declares functions including `getActionDefinitionsBuilder`.
- **CN**: 这一段实现或声明了 `getActionDefinitionsBuilder` 等函数。

### Lines 49-52 / 第 49-52 行
```cpp
  49:   getActionDefinitionsBuilder(G_PTR_ADD).legalFor({{p0, s32}});
  50: 
  51:   getLegacyLegalizerInfo().computeTables();
  52: }
```
- **EN**: This span continues the file's main responsibility: this file describes legalization rules for GlobalISel for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `M68kLegalizerInfo.h`
- `llvm/CodeGen/GlobalISel/LegalizerHelper.h`
- `llvm/CodeGen/GlobalISel/LegalizerInfo.h`
- `llvm/CodeGen/TargetOpcodes.h`
- `llvm/CodeGen/ValueTypes.h`
- `llvm/IR/DerivedTypes.h`
- `llvm/IR/Type.h`
