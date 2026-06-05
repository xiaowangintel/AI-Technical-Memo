# VECallingConv.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VECallingConv.td`
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
//===-- VECallingConv.td - Calling Conventions VE ----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-15
```tablegen
//
// This describes the calling conventions for the VE architectures.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Aurora VE
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 16-23
```tablegen
def CC_VE_C_Stack: CallingConv<[
  // F128 are assigned to the stack in 16-byte aligned units
  CCIfType<[f128], CCAssignToStackWithShadow<16, 16, [SX7]>>,

  // All of the rest are assigned to the stack in 8-byte aligned units.
  CCAssignToStack<0, 8>
]>;

```
- **EN**: Declares TableGen records such as `CC_VE_C_Stack` for the backend description.
- **CN**: 为后端描述声明了 `CC_VE_C_Stack` 等 TableGen 记录。

### Lines 24-30
```tablegen
///// C Calling Convention (VE ABI v2.1) /////
//
// Reference: https://www.nec.com/en/global/prod/hpc/aurora/document/VE-ABI_v2.1.pdf
//
def CC_VE_C : CallingConv<[
  // All arguments get passed in generic registers if there is space.

```
- **EN**: Declares TableGen records such as `CC_VE_C` for the backend description.
- **CN**: 为后端描述声明了 `CC_VE_C` 等 TableGen 记录。

### Lines 31-40
```tablegen
  // Promote i1/i8/i16/i32 arguments to i64.
  CCIfType<[i1, i8, i16, i32], CCPromoteToType<i64>>,

  // Convert float arguments to i64 with padding.
  //     63     31   0
  //    +------+------+
  //    | float|   0  |
  //    +------+------+
  CCIfType<[f32], CCBitConvertToType<i64>>,

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 41-53
```tablegen
  // bool, char, int, enum, long, long long, float, double
  //     --> generic 64 bit registers
  CCIfType<[i64, f64],
           CCAssignToReg<[SX0, SX1, SX2, SX3, SX4, SX5, SX6, SX7]>>,

  // long double --> pair of generic 64 bit registers
  //
  // NOTE: If Q1 is allocated while SX1 is free, llvm tries to allocate SX1 for
  //       following operands, this masks SX1 to avoid such behavior.
  CCIfType<[f128],
           CCAssignToRegWithShadow<[Q0, Q1, Q2, Q3],
                                   [SX0, SX1, SX3, SX5]>>,

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 54-64
```tablegen
  // Alternatively, they are assigned to the stack in 8-byte aligned units.
  CCDelegateTo<CC_VE_C_Stack>
]>;

///// Standard vararg C Calling Convention (VE ABI v2.1) /////
// All arguments get passed in stack for varargs function or non-prototyped
// function.
def CC_VE2 : CallingConv<[
  // Promote i1/i8/i16/i32 arguments to i64.
  CCIfType<[i1, i8, i16, i32], CCPromoteToType<i64>>,

```
- **EN**: Declares TableGen records such as `CC_VE2` for the backend description.
- **CN**: 为后端描述声明了 `CC_VE2` 等 TableGen 记录。

### Lines 65-71
```tablegen
  // Convert float arguments to i64 with padding.
  //     63     31   0
  //    +------+------+
  //    | float|   0  |
  //    +------+------+
  CCIfType<[f32], CCBitConvertToType<i64>>,

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 72-81
```tablegen
  // F128 are assigned to the stack in 16-byte aligned units
  CCIfType<[f128], CCAssignToStack<16, 16>>,

  CCAssignToStack<0, 8>
]>;

def RetCC_VE_C : CallingConv<[
  // Promote i1/i8/i16/i32 return values to i64.
  CCIfType<[i1, i8, i16, i32], CCPromoteToType<i64>>,

```
- **EN**: Declares TableGen records such as `RetCC_VE_C` for the backend description.
- **CN**: 为后端描述声明了 `RetCC_VE_C` 等 TableGen 记录。

### Lines 82-88
```tablegen
  // Convert float return values to i64 with padding.
  //     63     31   0
  //    +------+------+
  //    | float|   0  |
  //    +------+------+
  CCIfType<[f32], CCBitConvertToType<i64>>,

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 89-99
```tablegen
  // bool, char, int, enum, long, long long, float, double
  //     --> generic 64 bit registers
  CCIfType<[i64, f64],
           CCAssignToReg<[SX0, SX1, SX2, SX3, SX4, SX5, SX6, SX7]>>,

  // long double --> pair of generic 64 bit registers
  CCIfType<[f128],
           CCAssignToRegWithShadow<[Q0, Q1, Q2, Q3],
                                   [SX0, SX1, SX3, SX5]>>,
]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 100-111
```tablegen
///// Custom fastcc /////
//
// This passes vector params and return values in registers.  Scalar values are
// handled conforming to the standard cc.
def CC_VE_Fast : CallingConv<[
  // vector --> generic vector registers
  CCIfType<[v256i32, v256f32, v256i64, v256f64],
           CCAssignToReg<[V0, V1, V2, V3, V4, V5, V6, V7]>>,
  // TODO: make this conditional on packed mode
  CCIfType<[v512i32, v512f32],
           CCAssignToReg<[V0, V1, V2, V3, V4, V5, V6, V7]>>,

```
- **EN**: Declares TableGen records such as `CC_VE_Fast` for the backend description.
- **CN**: 为后端描述声明了 `CC_VE_Fast` 等 TableGen 记录。

### Lines 112-120
```tablegen
  // vector mask --> generic vector mask registers
  CCIfType<[v256i1],
           CCAssignToReg<[VM1, VM2, VM3, VM4, VM5, VM6, VM7]>>,

  // pair of vector mask --> generic vector mask registers
  CCIfType<[v512i1],
           CCAssignToRegWithShadow<[VMP1, VMP2, VMP3],
                                   [VM1, VM3, VM5]>>,

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 121-132
```tablegen
  // Follow the standard C CC for scalars.
  CCDelegateTo<CC_VE_C>
]>;

def RetCC_VE_Fast : CallingConv<[
  // vector --> generic vector registers
  CCIfType<[v256i32, v256f32, v256i64, v256f64],
           CCAssignToReg<[V0, V1, V2, V3, V4, V5, V6, V7]>>,
  // TODO: make this conditional on packed mode
  CCIfType<[v512i32, v512f32],
           CCAssignToReg<[V0, V1, V2, V3, V4, V5, V6, V7]>>,

```
- **EN**: Declares TableGen records such as `RetCC_VE_Fast` for the backend description.
- **CN**: 为后端描述声明了 `RetCC_VE_Fast` 等 TableGen 记录。

### Lines 133-141
```tablegen
  // vector mask --> generic vector mask registers
  CCIfType<[v256i1],
           CCAssignToReg<[VM1, VM2, VM3, VM4, VM5, VM6, VM7]>>,

  // pair of vector mask --> generic vector mask registers
  CCIfType<[v512i1],
           CCAssignToRegWithShadow<[VMP1, VMP2, VMP3],
                                   [VM1, VM3, VM5]>>,

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 142-149
```tablegen
  // Follow the standard C CC for scalars.
  CCDelegateTo<RetCC_VE_C>
]>;

// Callee-saved registers
def CSR : CalleeSavedRegs<(add (sequence "SX%u", 18, 33))>;
def CSR_NoRegs : CalleeSavedRegs<(add)>;

```
- **EN**: Declares TableGen records such as `CSR`, `CSR_NoRegs` for the backend description.
- **CN**: 为后端描述声明了 `CSR`, `CSR_NoRegs` 等 TableGen 记录。

### Lines 150-153
```tablegen
// PreserveAll (clobbers s62,s63) - used for ve_grow_stack
def CSR_preserve_all : CalleeSavedRegs<(add (sequence "SX%u", 0, 61),
                                            (sequence "V%u", 0, 63),
                                            (sequence "VM%u", 1, 15))>;
```
- **EN**: Declares TableGen records such as `CSR_preserve_all` for the backend description.
- **CN**: 为后端描述声明了 `CSR_preserve_all` 等 TableGen 记录。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
