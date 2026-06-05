# M68kCallingConv.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kCallingConv.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines M68k backend metadata using LLVM TableGen DSL; specifically it defines calling convention rules and register/stack assignments.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 M68k 后端元数据；具体而言，它定义调用约定规则以及寄存器/栈分配。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```tablegen
   1: //===-- M68kCallingConv.td - Calling Conventions for M68k --*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This describes the calling conventions for the M68k architectures. These
  11: /// conventions assume Int to be 4 bytes and 4 byte aligned.
  12: ///
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```tablegen
  13: //===----------------------------------------------------------------------===//
  14: 
  15: // TODO Verify C convention follows SysV M68K ABI
  16: 
  17: class CCIfSubtarget<string F, CCAction A>
  18:     : CCIf<!strconcat("State.getMachineFunction()."
  19:                       "getSubtarget<M68kSubtarget>().", F), A>;
  20: 
  21: //===----------------------------------------------------------------------===//
  22: // Return Value Calling Conventions
  23: //===----------------------------------------------------------------------===//
  24: 
```
- **EN**: This block declares or refines TableGen records such as `CCIfSubtarget`.
- **CN**: 该代码块声明或细化了 `CCIfSubtarget` 等 TableGen 记录。

### Lines 25-36 / 第 25-36 行
```tablegen
  25: /// M68k C return convention.
  26: def RetCC_M68k_C : CallingConv<[
  27:   CCIfPtr<CCAssignToReg<[A0]>>,
  28:   CCIfType<[i1],   CCPromoteToType<i8>>,
  29:   CCIfType<[i8],   CCAssignToReg<[BD0, BD1]>>,
  30:   CCIfType<[i16],  CCAssignToReg<[WD0, WD1]>>,
  31:   CCIfType<[i32],  CCAssignToReg<[D0, D1]>>,
  32: ]>;
  33: 
  34: /// M68k fastcc return convention.
  35: /// This convention allows to return up to 16 bytes in registers which can be
  36: /// split among 16 1-byte values or used for a single 16-byte value.
```
- **EN**: This block declares or refines TableGen records such as `RetCC_M68k_C`.
- **CN**: 该代码块声明或细化了 `RetCC_M68k_C` 等 TableGen 记录。

### Lines 37-48 / 第 37-48 行
```tablegen
  37: /// TODO: Verify its functionality and write tests
  38: def RetCC_M68k_Fast : CallingConv<[
  39:   CCIfPtr<CCAssignToReg<[A0]>>,
  40:   CCIfType<[i1],   CCPromoteToType<i8>>,
  41:   CCIfType<[i8],   CCAssignToReg<[BD0, BD1]>>,
  42:   CCIfType<[i16],  CCAssignToReg<[WD0, WD1, WA0, WA1]>>,
  43:   CCIfType<[i32],  CCAssignToReg<[D0, D1, A0, A1]>>,
  44: ]>;
  45: 
  46: /// This is the root return-value convention for the M68k backend.
  47: def RetCC_M68k : CallingConv<[
  48:   CCIfCC<"CallingConv::Fast", CCDelegateTo<RetCC_M68k_Fast>>,
```
- **EN**: This block declares or refines TableGen records such as `RetCC_M68k_Fast`, `RetCC_M68k`.
- **CN**: 该代码块声明或细化了 `RetCC_M68k_Fast`, `RetCC_M68k` 等 TableGen 记录。

### Lines 49-60 / 第 49-60 行
```tablegen
  49:   CCDelegateTo<RetCC_M68k_C>
  50: ]>;
  51: 
  52: //===----------------------------------------------------------------------===//
  53: // M68k C Calling Convention
  54: //===----------------------------------------------------------------------===//
  55: 
  56: /// CC_M68k_Common - In all M68k calling conventions, extra integers and FP
  57: /// values are spilled on the stack.
  58: def CC_M68k_Common : CallingConv<[
  59:   /// Handles byval parameters.
  60:   CCIfByVal<CCPassByVal<4, 4>>,
```
- **EN**: This block declares or refines TableGen records such as `CC_M68k_Common`.
- **CN**: 该代码块声明或细化了 `CC_M68k_Common` 等 TableGen 记录。

### Lines 61-72 / 第 61-72 行
```tablegen
  61: 
  62:   /// Integer values get stored in stack slots that are 4 bytes in
  63:   /// size and 4-byte aligned.
  64:   CCIfType<[i32],  CCAssignToStack<4, 4>>
  65: ]>;
  66: 
  67: def CC_M68k_Fast : CallingConv<[
  68:   /// Promote i1/i8/i16 arguments to i32.
  69:   CCIfType<[i1, i8, i16], CCPromoteToType<i32>>,
  70: 
  71:   /// The 'nest' parameter, if any, is passed in A1.
  72:   CCIfNest<CCAssignToReg<[A1]>>, // FIXME verify if this is correct
```
- **EN**: This block declares or refines TableGen records such as `CC_M68k_Fast`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `CC_M68k_Fast` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-84 / 第 73-84 行
```tablegen
  73: 
  74:   /// Since M68k uses %An for pointers and we want them be passed in regs
  75:   /// too we have to use custom function.
  76:   CCIfType<[i32], CCCustom<"CC_M68k_Any_AssignToReg">>,
  77: 
  78:   /// Otherwise, same as everything else.
  79:   CCDelegateTo<CC_M68k_Common>
  80: ]>;
  81: 
  82: def CC_M68k_C : CallingConv<[
  83:   /// Promote i1/i8/i16 arguments to i32.
  84:   CCIfType<[i1, i8, i16], CCPromoteToType<i32>>,
```
- **EN**: This block declares or refines TableGen records such as `CC_M68k_C`.
- **CN**: 该代码块声明或细化了 `CC_M68k_C` 等 TableGen 记录。

### Lines 85-96 / 第 85-96 行
```tablegen
  85: 
  86:   /// The 'nest' parameter, if any, is passed in A1.
  87:   CCIfNest<CCAssignToReg<[A1]>>, // FIXME verify if this is correct
  88: 
  89:   /// Use registers only if 'inreg' used and the call is not vararg
  90:   CCIfNotVarArg<CCIfInReg<CCIfType<[i32], CCAssignToReg<[D0, D1]>>>>,
  91: 
  92:   // TODO: Support for 'sret'
  93: 
  94:   /// Otherwise, same as everything else.
  95:   CCDelegateTo<CC_M68k_Common>
  96: ]>;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 97-108 / 第 97-108 行
```tablegen
  97: 
  98: /// This is the root argument convention for the M68k backend.
  99: def CC_M68k : CallingConv<[
 100:   CCIfCC<"CallingConv::Fast", CCDelegateTo<CC_M68k_Fast>>,
 101:   CCDelegateTo<CC_M68k_C>
 102: ]>;
 103: 
 104: //===----------------------------------------------------------------------===//
 105: // Callee-saved Registers.
 106: //===----------------------------------------------------------------------===//
 107: 
 108: def CSR_NoRegs : CalleeSavedRegs<(add)>;
```
- **EN**: This block declares or refines TableGen records such as `CC_M68k`, `CSR_NoRegs`.
- **CN**: 该代码块声明或细化了 `CC_M68k`, `CSR_NoRegs` 等 TableGen 记录。

### Lines 109-114 / 第 109-114 行
```tablegen
 109: 
 110: // A5 - BP
 111: // A6 - FP
 112: def CSR_STD : CalleeSavedRegs<(add D2, D3, D4, D5, D6, D7,
 113:                                    A2, A3, A4, A5, A6)>;
 114: 
```
- **EN**: This block declares or refines TableGen records such as `CSR_STD`.
- **CN**: 该代码块声明或细化了 `CSR_STD` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
