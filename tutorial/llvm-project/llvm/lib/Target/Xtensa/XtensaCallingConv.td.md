# XtensaCallingConv.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaCallingConv.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines calling-convention lowering rules and argument/return assignments. This file is written in TableGen DSL rather than C++.
  - **CN**: 定义调用约定降低规则以及参数/返回值分配方式。 该文件使用的是 TableGen DSL，而不是 C++。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===- XtensaCallingConv.td - Xtensa Calling Conventions -*- tablegen ---*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-11
```tablegen
// This describes the calling conventions for the Xtensa ABI.
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 12-19
```tablegen
// Xtensa base calling convention
//===----------------------------------------------------------------------===//
// Xtensa return value
def RetCC_Xtensa : CallingConv<[
  // First two return values go in a2, a3, a4, a5
  CCIfType<[i32], CCAssignToReg<[A2, A3, A4, A5]>>,
  CCIfType<[f32], CCAssignToReg<[A2, A3, A4, A5]>>,
  CCIfType<[i64], CCAssignToRegWithShadow<[A2, A4], [A3, A5]>>
```
- **EN**: Declares TableGen records such as `RetCC_Xtensa` for the backend description.
- **CN**: 为后端描述声明了 `RetCC_Xtensa` 等 TableGen 记录。

### Lines 20-24
```tablegen
]>;

// Callee-saved register lists
def CSR_Xtensa : CalleeSavedRegs<(add A0, A12, A13, A14, A15)>;

```
- **EN**: Declares TableGen records such as `CSR_Xtensa` for the backend description.
- **CN**: 为后端描述声明了 `CSR_Xtensa` 等 TableGen 记录。

### Lines 25-28
```tablegen
//===----------------------------------------------------------------------===//
// Xtensa windowed calling convention. Currently by default implemented
// rotation window by 8 registers.
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 29-35
```tablegen
// Xtensa return value for 8 registers window
def RetCCW8_Xtensa : CallingConv<[
  //First two return values go in a10, a11, a12, a13
  CCIfType<[i32], CCAssignToReg<[A10, A11, A12, A13]>>,
  CCIfType<[i64], CCAssignToRegWithShadow<[A10, A12], [A11, A13]>>
]>;

```
- **EN**: Declares TableGen records such as `RetCCW8_Xtensa` for the backend description.
- **CN**: 为后端描述声明了 `RetCCW8_Xtensa` 等 TableGen 记录。

### Lines 36-39
```tablegen
// Callee-saved register lists for rotation window by 8 registers
def CSRW8_Xtensa : CalleeSavedRegs<(add)> {
  let OtherPreserved = (add A0, SP, A2, A3, A4, A5, A6, A7);
}
```
- **EN**: Declares TableGen records such as `CSRW8_Xtensa`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `CSRW8_Xtensa` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
