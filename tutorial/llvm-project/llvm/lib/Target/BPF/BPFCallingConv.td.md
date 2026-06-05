# BPFCallingConv.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFCallingConv.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines calling-convention rules in TableGen for argument passing, returns, and special ABI cases.
- 目的（中文）: 使用 TableGen 定义调用约定规则，覆盖参数传递、返回值以及特殊 ABI 情况。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- BPFCallingConv.td - Calling Conventions BPF --------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This describes the calling conventions for the BPF architecture.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: // BPF 64-bit C return-value convention.
  14: def RetCC_BPF64 : CallingConv<[CCIfType<[i64], CCAssignToReg<[R0, R2]>>]>;
  15: 
  16: // BPF 64-bit C Calling convention.
  17: def CC_BPF64 : CallingConv<[
  18:   // Promote i8/i16/i32 args to i64
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as RetCC_BPF64, CC_BPF64, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 RetCC_BPF64, CC_BPF64 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19:   CCIfType<[ i8, i16, i32 ], CCPromoteToType<i64>>,
  20: 
  21:   // All arguments get passed in integer registers if there is space.
  22:   CCIfType<[i64], CCAssignToReg<[ R1, R2, R3, R4, R5 ]>>,
  23: 
  24:   // Could be assigned to the stack in 8-byte aligned units, but unsupported
  25:   CCAssignToStack<8, 8>
  26: ]>;
  27: 
  28: // Return-value convention when -mattr=+alu32 enabled
  29: def RetCC_BPF32 : CallingConv<[
  30:   CCIfType<[i32], CCAssignToRegWithShadow<[W0, W2], [R0, R2]>>,
  31:   CCIfType<[i64], CCAssignToRegWithShadow<[R0, R2], [W0, W2]>>
  32: ]>;
  33: 
  34: // Calling convention when -mattr=+alu32 enabled
  35: def CC_BPF32 : CallingConv<[
  36:   // Promote i8/i16/i32 args to i64
```

- EN: This range defines declarative TableGen records such as RetCC_BPF32, CC_BPF32, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 RetCC_BPF32, CC_BPF32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-49

```tablegen
  37:   CCIfType<[i32], CCAssignToRegWithShadow<[W1, W2, W3, W4, W5],
  38:                                           [R1, R2, R3, R4, R5]>>,
  39: 
  40:   // All arguments get passed in integer registers if there is space.
  41:   CCIfType<[i64], CCAssignToRegWithShadow<[R1, R2, R3, R4, R5],
  42:                                           [W1, W2, W3, W4, W5]>>,
  43: 
  44:   // Could be assigned to the stack in 8-byte aligned units, but unsupported
  45:   CCAssignToStack<8, 8>
  46: ]>;
  47: 
  48: def CSR : CalleeSavedRegs<(add R6, R7, R8, R9, R10)>;
  49: def CSR_PreserveAll : CalleeSavedRegs<(add R0, R1, R2, R3, R4, R5, R6, R7, R8, R9, R10)>;
```

- EN: This range defines declarative TableGen records such as CSR, CSR_PreserveAll, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CSR, CSR_PreserveAll 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- ABI rules / ABI 规则
- Argument assignment / 参数分配
- Calling convention / 调用约定

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
