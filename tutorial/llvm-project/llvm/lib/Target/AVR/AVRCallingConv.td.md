# AVRCallingConv.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRCallingConv.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines calling-convention rules in TableGen for argument passing, returns, and special ABI cases.
- 目的（中文）: 使用 TableGen 定义调用约定规则，覆盖参数传递、返回值以及特殊 ABI 情况。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- AVRCallingConv.td - Calling Conventions for AVR ----*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: // This describes the calling conventions for AVR architecture.
   9: // Normal functions use a special calling convention, solved in code.
  10: //===----------------------------------------------------------------------===//
  11: 
  12: //===----------------------------------------------------------------------===//
  13: // AVR Return Value Calling Convention
  14: //===----------------------------------------------------------------------===//
  15: 
  16: // Special return value calling convention for runtime functions.
  17: def RetCC_AVR_BUILTIN : CallingConv<[
  18:   CCIfType<[i8], CCAssignToReg<[R24, R25]>>,
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as RetCC_AVR_BUILTIN, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 RetCC_AVR_BUILTIN 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19:   CCIfType<[i16], CCAssignToReg<[R23R22, R25R24]>>
  20: ]>;
  21: 
  22: //===----------------------------------------------------------------------===//
  23: // AVR Argument Calling Conventions
  24: //===----------------------------------------------------------------------===//
  25: 
  26: // The calling conventions are implemented in custom C++ code
  27: 
  28: // Calling convention for variadic functions.
  29: def ArgCC_AVR_Vararg : CallingConv<[
  30:   // i8 are always passed through the stack with a byte slot and byte alignment.
  31:   CCIfType<[i8], CCAssignToStack<1, 1>>,
  32:   // i16 are always passed through the stack with an alignment of 1.
  33:   CCAssignToStack<2, 1>
  34: ]>;
  35: 
  36: //===----------------------------------------------------------------------===//
```

- EN: This range defines declarative TableGen records such as ArgCC_AVR_Vararg, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ArgCC_AVR_Vararg 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-43

```tablegen
  37: // Callee-saved register lists.
  38: //===----------------------------------------------------------------------===//
  39: 
  40: def CSR_Normal : CalleeSavedRegs<(add R29, R28, (sequence "R%u", 17, 2))>;
  41: def CSR_NormalTiny : CalleeSavedRegs<(add R29, R28, R19, R18)>;
  42: def CSR_Interrupts : CalleeSavedRegs<(add(sequence "R%u", 31, 2))>;
  43: def CSR_InterruptsTiny : CalleeSavedRegs<(add(sequence "R%u", 31, 18))>;
```

- EN: This range defines declarative TableGen records such as CSR_Normal, CSR_NormalTiny, CSR_Interrupts, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CSR_Normal, CSR_NormalTiny, CSR_Interrupts 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- ABI rules / ABI 规则
- Argument assignment / 参数分配
- Calling convention / 调用约定

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
