# LanaiCallingConv.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiCallingConv.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines calling-convention rules in TableGen for argument passing, returns, and special ABI cases.
- 目的（中文）: 使用 TableGen 定义调用约定规则，覆盖参数传递、返回值以及特殊 ABI 情况。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- LanaiCallingConv.td - Calling Conventions Lanai -------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This describes the calling conventions for the Lanai architectures.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: //===----------------------------------------------------------------------===//
  14: // Return Value Calling Conventions
  15: //===----------------------------------------------------------------------===//
  16: 
  17: // Lanai 32-bit C Calling convention.
  18: def CC_Lanai32 : CallingConv<[
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as CC_Lanai32, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 CC_Lanai32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19:   // Promote i8/i16 args to i32
  20:   CCIfType<[i8, i16], CCPromoteToType<i32>>,
  21: 
  22:   // Put argument in registers if marked 'inreg' and not a vararg call.
  23:   CCIfNotVarArg<CCIfInReg<CCIfType<[i32],
  24:                                    CCAssignToReg<[R6, R7, R18, R19]>>>>,
  25: 
  26:   // Otherwise they are assigned to the stack in 4-byte aligned units.
  27:   CCAssignToStack<4, 4>
  28: ]>;
  29: 
  30: // Lanai 32-bit Fast Calling convention.
  31: def CC_Lanai32_Fast : CallingConv<[
  32:   // Promote i8/i16 args to i32
  33:   CCIfType<[ i8, i16 ], CCPromoteToType<i32>>,
  34: 
  35:   // Put arguments in registers.
  36:   CCIfNotVarArg<CCIfType<[i32], CCAssignToReg<[ R6, R7, R18, R19 ]>>>,
```

- EN: This range defines declarative TableGen records such as CC_Lanai32_Fast, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CC_Lanai32_Fast 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-49

```tablegen
  37: 
  38:   // Otherwise they are assigned to the stack in 4-byte aligned units.
  39:   CCAssignToStack<4, 4>
  40: ]>;
  41: 
  42: // Lanai 32-bit C return-value convention.
  43: def RetCC_Lanai32 : CallingConv<[
  44:   // Specify two registers to allow returning 64-bit results that have already
  45:   // been lowered to 2 32-bit values.
  46:   CCIfType<[i32], CCAssignToReg<[RV, R9]>>
  47: ]>;
  48: 
  49: def CSR: CalleeSavedRegs<(add)>;
```

- EN: This range defines declarative TableGen records such as RetCC_Lanai32, CSR, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 RetCC_Lanai32, CSR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- ABI rules / ABI 规则
- Argument assignment / 参数分配
- Calling convention / 调用约定

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
