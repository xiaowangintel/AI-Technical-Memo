# CSKYCallingConv.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYCallingConv.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the custom routines for the CSKY Calling Convention that aren't done by tablegen.
- 目的（中文）: 实现降级与寄存器分配过程中使用的目标调用约定辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //=== CSKYCallingConv.h - CSKY Custom Calling Convention Routines -*-C++-*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the custom routines for the CSKY Calling Convention that
  10: // aren't done by tablegen.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_CSKY_CSKYCALLINGCONV_H
  15: #define LLVM_LIB_TARGET_CSKY_CSKYCALLINGCONV_H
  16: 
  17: #include "CSKY.h"
  18: #include "CSKYSubtarget.h"
  19: #include "llvm/CodeGen/CallingConvLower.h"
  20: #include "llvm/CodeGen/TargetInstrInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/IR/CallingConv.h"
  22: 
  23: namespace llvm {
  24: 
  25: static bool CC_CSKY_ABIV2_SOFT_64(unsigned &ValNo, MVT &ValVT, MVT &LocVT,
  26:                                   CCValAssign::LocInfo &LocInfo,
  27:                                   ISD::ArgFlagsTy &ArgFlags, CCState &State) {
  28: 
  29:   static const MCPhysReg ArgGPRs[] = {CSKY::R0, CSKY::R1, CSKY::R2, CSKY::R3};
  30:   Register Reg = State.AllocateReg(ArgGPRs);
  31:   LocVT = MVT::i32;
  32:   if (!Reg) {
  33:     unsigned StackOffset = State.AllocateStack(8, Align(4));
  34:     State.addLoc(
  35:         CCValAssign::getMem(ValNo, ValVT, StackOffset, LocVT, LocInfo));
  36:     return true;
  37:   }
  38:   if (!State.AllocateReg(ArgGPRs))
  39:     State.AllocateStack(4, Align(4));
  40:   State.addLoc(CCValAssign::getReg(ValNo, ValVT, Reg, LocVT, LocInfo));
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as CC_CSKY_ABIV2_SOFT_64 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 CC_CSKY_ABIV2_SOFT_64 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 41-60

```cpp
  41:   return true;
  42: }
  43: 
  44: static bool Ret_CSKY_ABIV2_SOFT_64(unsigned &ValNo, MVT &ValVT, MVT &LocVT,
  45:                                    CCValAssign::LocInfo &LocInfo,
  46:                                    ISD::ArgFlagsTy &ArgFlags, CCState &State) {
  47: 
  48:   static const MCPhysReg ArgGPRs[] = {CSKY::R0, CSKY::R1};
  49:   Register Reg = State.AllocateReg(ArgGPRs);
  50:   LocVT = MVT::i32;
  51:   if (!Reg)
  52:     return false;
  53: 
  54:   if (!State.AllocateReg(ArgGPRs))
  55:     return false;
  56: 
  57:   State.addLoc(CCValAssign::getReg(ValNo, ValVT, Reg, LocVT, LocInfo));
  58:   return true;
  59: }
  60: 
```

- EN: Function bodies or method definitions such as Ret_CSKY_ABIV2_SOFT_64 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: Ret_CSKY_ABIV2_SOFT_64 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 61-63

```cpp
  61: } // namespace llvm
  62: 
  63: #endif
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

## Key Concepts / 关键概念

- Instruction semantics helpers / 指令语义辅助逻辑
- CPU feature modelling / CPU 特性建模
- Calling convention / 调用约定

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKY.h`, `CSKYSubtarget.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/IR/CallingConv.h`
- Local companions / 本地配套文件: `CSKYCallingConv.td`
