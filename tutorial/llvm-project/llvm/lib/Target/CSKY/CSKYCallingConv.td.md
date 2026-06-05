# CSKYCallingConv.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYCallingConv.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines calling-convention rules in TableGen for argument passing, returns, and special ABI cases.
- 目的（中文）: 使用 TableGen 定义调用约定规则，覆盖参数传递、返回值以及特殊 ABI 情况。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- CSKYCallingConv.td - Calling Conventions CSKY ----*- tablegen -*---===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This describes the calling conventions for the CSKY architecture.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: def CSR_I32 : CalleeSavedRegs<(add R8, R15, (sequence "R%u", 4, 7),
  14:   (sequence "R%u", 9, 11), (sequence "R%u", 16, 17), R28)>;
  15: def CSR_GPR_FPR32 : CalleeSavedRegs<(add CSR_I32, (sequence "F%u_32", 8, 15))>;
  16: def CSR_GPR_FPR64 : CalleeSavedRegs<(add CSR_I32,
  17:                                     (sequence "F%u_64", 8, 15))>;
  18: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as CSR_I32, CSR_GPR_FPR32, CSR_GPR_FPR64, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 CSR_I32, CSR_GPR_FPR32, CSR_GPR_FPR64 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19: // Interrupt handler needs to save/restore all registers that are used,
  20: // both Caller and Callee saved registers.
  21: def CSR_GPR_ISR : CalleeSavedRegs<(add R8, R15,
  22:     (sequence "R%u", 0, 3),
  23:     (sequence "R%u", 4, 7),
  24:     (sequence "R%u", 9, 13),
  25:     (sequence "R%u", 16, 31))>;
  26: 
  27: def CSR_GPR_FPR32_ISR: CalleeSavedRegs<(add CSR_GPR_ISR,
  28:     (sequence "F%u_32", 0, 15))>;
  29: def CSR_GPR_FPR64_ISR: CalleeSavedRegs<(add CSR_GPR_ISR,
  30:     (sequence "F%u_64", 0, 15))>;
  31: 
  32: def CSR_GPR_FPR32v3_ISR: CalleeSavedRegs<(add CSR_GPR_FPR32_ISR,
  33:     (sequence "F%u_32", 16, 31))>;
  34: def CSR_GPR_FPR64v3_ISR: CalleeSavedRegs<(add CSR_GPR_FPR64_ISR,
  35:     (sequence "F%u_64", 16, 31))>;
  36: 
```

- EN: This range defines declarative TableGen records such as CSR_GPR_ISR, CSR_GPR_FPR32_ISR, CSR_GPR_FPR64_ISR, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CSR_GPR_ISR, CSR_GPR_FPR32_ISR, CSR_GPR_FPR64_ISR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-54

```tablegen
  37: // Needed for implementation of CSKYRegisterInfo::getNoPreservedMask()
  38: def CSR_NoRegs : CalleeSavedRegs<(add)>;
  39: 
  40: def CC_CSKY_ABIV2_SOFT : CallingConv<[
  41:   // DSP types
  42:   CCIfType<[v2i16, v4i8], CCAssignToReg<[R0, R1, R2, R3]>>,
  43:   CCIfType<[v2i16, v4i8], CCAssignToStack<4, 4>>,
  44:   CCIfType<[i8, i16], CCPromoteToType<i32>>,
  45:   CCIfType<[f32], CCAssignToReg<[R0, R1, R2, R3]>>,
  46:   CCIfType<[f32], CCAssignToStack<4, 4>>,
  47:   CCIfType<[i32], CCAssignToReg<[R0, R1, R2, R3]>>,
  48:   CCIfType<[i32], CCAssignToStack<4, 4>>,
  49:   CCIfType<[f64], CCCustom<"CC_CSKY_ABIV2_SOFT_64">>,
  50:   CCIfType<[f64], CCAssignToStack<8, 4>>
  51: ]>;
  52: 
  53: def RetCC_CSKY_ABIV2_SOFT : CallingConv<[
  54:   // DSP types
```

- EN: This range defines declarative TableGen records such as CSR_NoRegs, CC_CSKY_ABIV2_SOFT, RetCC_CSKY_ABIV2_SOFT, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CSR_NoRegs, CC_CSKY_ABIV2_SOFT, RetCC_CSKY_ABIV2_SOFT 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55:   CCIfType<[v2i16, v4i8], CCAssignToReg<[R0, R1]>>,
  56:   CCIfType<[i8, i16], CCPromoteToType<i32>>,
  57:   CCIfType<[f32], CCBitConvertToType<i32>>,
  58:   CCIfType<[i32], CCAssignToReg<[R0, R1]>>,
  59:   CCIfType<[f64], CCCustom<"Ret_CSKY_ABIV2_SOFT_64">>
  60: ]>;
  61: 
  62: def CC_CSKY_ABIV2_FP : CallingConv<[
  63:   // DSP types
  64:   CCIfType<[v2i16, v4i8], CCAssignToReg<[R0, R1, R2, R3]>>,
  65:   CCIfType<[v2i16, v4i8], CCAssignToStack<4, 4>>,
  66:   CCIfType<[i8, i16], CCPromoteToType<i32>>,
  67:   CCIfType<[i32], CCAssignToReg<[R0, R1, R2, R3]>>,
  68:   CCIfType<[i32], CCAssignToStack<4, 4>>,
  69:   CCIfType<[f32], CCAssignToReg<[F0_32, F1_32, F2_32, F3_32]>>,
  70:   CCIfType<[f32], CCAssignToStack<4, 4>>,
  71:   CCIfType<[f64], CCAssignToReg<[F0_64, F1_64, F2_64, F3_64]>>,
  72:   CCIfType<[f64], CCAssignToStack<8, 4>>
```

- EN: This range defines declarative TableGen records such as CC_CSKY_ABIV2_FP, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CC_CSKY_ABIV2_FP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-82

```tablegen
  73: ]>;
  74: 
  75: def RetCC_CSKY_ABIV2_FP : CallingConv<[
  76:   // DSP types
  77:   CCIfType<[v2i16, v4i8], CCAssignToReg<[R0, R1]>>,
  78:   CCIfType<[i8, i16], CCPromoteToType<i32>>,
  79:   CCIfType<[i32], CCAssignToReg<[R0, R1]>>,
  80:   CCIfType<[f32], CCAssignToReg<[F0_32]>>,
  81:   CCIfType<[f64], CCAssignToReg<[F0_64]>>
  82: ]>;
```

- EN: This range defines declarative TableGen records such as RetCC_CSKY_ABIV2_FP, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 RetCC_CSKY_ABIV2_FP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- ABI rules / ABI 规则
- Argument assignment / 参数分配
- Calling convention / 调用约定

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
- Local companions / 本地配套文件: `CSKYCallingConv.h`
