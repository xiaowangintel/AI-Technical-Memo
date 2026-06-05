# SystemZCallingConv.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZCallingConv.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines calling convention rules and register/stack assignments for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责定义调用约定规则以及寄存器/栈分配。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //===-- SystemZCallingConv.cpp - Calling conventions for SystemZ ----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZCallingConv.h"
  10: 
  11: using namespace llvm;
  12: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZCallingConv.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZCallingConv.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 13-24 / 第 13-24 行
```cpp
  13: const MCPhysReg SystemZ::ELFArgGPRs[SystemZ::ELFNumArgGPRs] = {
  14:   SystemZ::R2D, SystemZ::R3D, SystemZ::R4D, SystemZ::R5D, SystemZ::R6D
  15: };
  16: 
  17: const MCPhysReg SystemZ::ELFArgFPRs[SystemZ::ELFNumArgFPRs] = {
  18:   SystemZ::F0D, SystemZ::F2D, SystemZ::F4D, SystemZ::F6D
  19: };
  20: 
  21: // The XPLINK64 ABI-defined param passing general purpose registers
  22: const MCPhysReg SystemZ::XPLINK64ArgGPRs[SystemZ::XPLINK64NumArgGPRs] = {
  23:     SystemZ::R1D, SystemZ::R2D, SystemZ::R3D
  24: };
```
- **EN**: This span continues the file's main responsibility: this file defines calling convention rules and register/stack assignments for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 25-29 / 第 25-29 行
```cpp
  25: 
  26: // The XPLINK64 ABI-defined param passing floating point registers
  27: const MCPhysReg SystemZ::XPLINK64ArgFPRs[SystemZ::XPLINK64NumArgFPRs] = {
  28:     SystemZ::F0D, SystemZ::F2D, SystemZ::F4D, SystemZ::F6D
  29: };
```
- **EN**: This span continues the file's main responsibility: this file defines calling convention rules and register/stack assignments for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。

## Dependencies / 依赖关系
- `SystemZCallingConv.h`
