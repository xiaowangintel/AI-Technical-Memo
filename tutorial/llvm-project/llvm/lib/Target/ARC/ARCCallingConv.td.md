# ARCCallingConv.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCCallingConv.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines calling-convention rules in TableGen for argument passing, returns, and special ABI cases.
- 目的（中文）: 使用 TableGen 定义调用约定规则，覆盖参数传递、返回值以及特殊 ABI 情况。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- ARCCallingConv.td - Calling Conventions for ARC -----*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: // This describes the calling conventions for ARC architecture.
   9: //===----------------------------------------------------------------------===//
  10: 
  11: //===----------------------------------------------------------------------===//
  12: // ARC Return Value Calling Convention
  13: //===----------------------------------------------------------------------===//
  14: def RetCC_ARC : CallingConv<[
  15:   // i32 are returned in registers R0, R1, R2, R3
  16:   CCIfType<[i32, i64], CCAssignToReg<[R0, R1, R2, R3]>>,
  17: 
  18:   // Integer values get stored in stack slots that are 4 bytes in
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as RetCC_ARC, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 RetCC_ARC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19:   // size and 4-byte aligned.
  20:   CCIfType<[i64], CCAssignToStack<8, 4>>,
  21:   CCIfType<[i32], CCAssignToStack<4, 4>>
  22: ]>;
  23: 
  24: //===----------------------------------------------------------------------===//
  25: // ARC Argument Calling Conventions
  26: //===----------------------------------------------------------------------===//
  27: def CC_ARC : CallingConv<[
  28:   // Promote i8/i16 arguments to i32.
  29:   CCIfType<[i8, i16], CCPromoteToType<i32>>,
  30: 
  31:   // The first 8 integer arguments are passed in integer registers.
  32:   CCIfType<[i32, i64], CCAssignToReg<[R0, R1, R2, R3, R4, R5, R6, R7]>>,
  33: 
  34:   // Integer values get stored in stack slots that are 4 bytes in
  35:   // size and 4-byte aligned.
  36:   CCIfType<[i64], CCAssignToStack<8, 4>>,
```

- EN: This range defines declarative TableGen records such as CC_ARC, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CC_ARC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-40

```tablegen
  37:   CCIfType<[i32], CCAssignToStack<4, 4>>
  38: ]>;
  39: 
  40: def CSR_ARC : CalleeSavedRegs<(add (sequence "R%u", 13, 25), GP, FP)>;
```

- EN: This range defines declarative TableGen records such as CSR_ARC, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CSR_ARC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- ABI rules / ABI 规则
- Argument assignment / 参数分配
- Calling convention / 调用约定

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
