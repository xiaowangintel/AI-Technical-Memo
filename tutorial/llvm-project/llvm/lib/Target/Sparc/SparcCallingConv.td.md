# SparcCallingConv.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcCallingConv.td`
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
//===-- SparcCallingConv.td - Calling Conventions Sparc ----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-12
```tablegen
//
// This describes the calling conventions for the Sparc architectures.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```tablegen
//===----------------------------------------------------------------------===//
// SPARC v8 32-bit.
//===----------------------------------------------------------------------===//

def CC_Sparc32 : CallingConv<[
  // Custom assign SRet to [sp+64].
  CCIfSRet<CCCustom<"CC_Sparc_Assign_SRet">>,
  // f128 arguments are passed indirectly, using i32 pointers.
  // FIXME GCC in soft-float mode passes f128 as if 2xi64 values.
  CCIfType<[f128], CCPassIndirect<i32>>,
```
- **EN**: Declares TableGen records such as `CC_Sparc32` for the backend description.
- **CN**: 为后端描述声明了 `CC_Sparc32` 等 TableGen 记录。

### Lines 23-32
```tablegen
  // i32 f32 arguments get passed in integer registers if there is space.
  CCIfType<[i32, f32], CCAssignToReg<[I0, I1, I2, I3, I4, I5]>>,
  // f64 arguments are split and passed through registers or through stack.
  CCIfType<[f64], CCCustom<"CC_Sparc_Assign_Split_64">>,
  // As are v2i32 arguments (this would be the default behavior for
  // v2i32 if it wasn't allocated to the IntPair register-class)
  CCIfType<[v2i32], CCCustom<"CC_Sparc_Assign_Split_64">>,
  // Alternatively, they are assigned to the stack in 4-byte aligned units.
  CCAssignToStack<4, 4>
]>;
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 33-42
```tablegen


def RetCC_Sparc32 : CallingConv<[
  CCIfType<[i32], CCAssignToReg<[I0, I1, I2, I3, I4, I5]>>,
  CCIfType<[f32], CCAssignToReg<[F0, F1, F2, F3]>>,
  CCIfType<[f64], CCAssignToReg<[D0, D1]>>,
  // FIXME GCC in soft-float mode passes f128 as if 2xi64 values.
  CCIfType<[f128], CCIfInReg<CCAssignToReg<[Q0, Q1]>>>,
  CCIfType<[v2i32], CCCustom<"CC_Sparc_Assign_Ret_Split_64">>
]>;
```
- **EN**: Declares TableGen records such as `RetCC_Sparc32` for the backend description.
- **CN**: 为后端描述声明了 `RetCC_Sparc32` 等 TableGen 记录。

### Lines 43-52
```tablegen

//===----------------------------------------------------------------------===//
// SPARC v9 64-bit.
//===----------------------------------------------------------------------===//
//
// The 64-bit ABI conceptually assigns all function arguments to a parameter
// array starting at [%fp+BIAS+128] in the callee's stack frame. All arguments
// occupy a multiple of 8 bytes in the array. Integer arguments are extended to
// 64 bits by the caller. Floats are right-aligned in their 8-byte slot, the
// first 4 bytes in the slot are undefined.
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 53-62
```tablegen
//
// The integer registers %i0 to %i5 shadow the first 48 bytes of the parameter
// array at fixed offsets. Integer arguments are promoted to registers when
// possible.
//
// The floating point registers %f0 to %f31 shadow the first 128 bytes of the
// parameter array at fixed offsets. Float and double parameters are promoted
// to these registers when possible.
//
// Structs up to 16 bytes in size are passed by value. They are right-aligned
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 63-72
```tablegen
// in one or two 8-byte slots in the parameter array. Struct members are
// promoted to both floating point and integer registers when possible. A
// struct containing two floats would thus be passed in %f0 and %f1, while two
// float function arguments would occupy 8 bytes each, and be passed in %f1 and
// %f3.
//
// When a struct { int, float } is passed by value, the int goes in the high
// bits of an integer register while the float goes in a floating point
// register.
//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 73-82
```tablegen
// The difference is encoded in LLVM IR using the inreg attribute on function
// arguments:
//
//   C:   void f(float, float);
//   IR:  declare void f(float %f1, float %f3)
//
//   C:   void f(struct { float f0, f1; });
//   IR:  declare void f(float inreg %f0, float inreg %f1)
//
//   C:   void f(int, float);
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 83-92
```tablegen
//   IR:  declare void f(int signext %i0, float %f3)
//
//   C:   void f(struct { int i0high; float f1; });
//   IR:  declare void f(i32 inreg %i0high, float inreg %f1)
//
// Two ints in a struct are simply coerced to i64:
//
//   C:   void f(struct { int i0high, i0low; });
//   IR:  declare void f(i64 %i0.coerced)
//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 93-102
```tablegen
// The frontend and backend divide the task of producing ABI compliant code for
// C functions. The C frontend will:
//
//  - Annotate integer arguments with zeroext or signext attributes.
//
//  - Split structs into one or two 64-bit sized chunks, or 32-bit chunks with
//    inreg attributes.
//
//  - Pass structs larger than 16 bytes indirectly with an explicit pointer
//    argument. The byval attribute is not used.
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 103-112
```tablegen
//
// The backend will:
//
//  - Assign all arguments to 64-bit aligned stack slots, 32-bits for inreg.
//
//  - Promote to integer or floating point registers depending on type.
//
// Function return values are passed exactly like function arguments, except a
// struct up to 32 bytes in size can be returned in registers.

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 113-119
```tablegen
// Function arguments AND most return values.
def CC_Sparc64 : CallingConv<[
  // The frontend uses the inreg flag to indicate i32 and float arguments from
  // structs. These arguments are not promoted to 64 bits, but they can still
  // be assigned to integer and float registers.
  CCIfInReg<CCIfType<[i32, f32], CCCustom<"CC_Sparc64_Half">>>,

```
- **EN**: Declares TableGen records such as `CC_Sparc64` for the backend description.
- **CN**: 为后端描述声明了 `CC_Sparc64` 等 TableGen 记录。

### Lines 120-127
```tablegen
  // All integers are promoted to i64 by the caller.
  CCIfType<[i32], CCPromoteToType<i64>>,

  // Custom assignment is required because stack space is reserved for all
  // arguments whether they are passed in registers or not.
  CCCustom<"CC_Sparc64_Full">
]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 128-132
```tablegen
def RetCC_Sparc64 : CallingConv<[
  // A single f32 return value always goes in %f0. The ABI doesn't specify what
  // happens to multiple f32 return values outside a struct.
  CCIfType<[f32], CCCustom<"RetCC_Sparc64_Half">>,

```
- **EN**: Declares TableGen records such as `RetCC_Sparc64` for the backend description.
- **CN**: 为后端描述声明了 `RetCC_Sparc64` 等 TableGen 记录。

### Lines 133-140
```tablegen
  // Otherwise, return values are passed exactly like arguments, except that
  // returns that are too big to fit into the registers is passed as an sret
  // instead.
  CCIfInReg<CCIfType<[i32, f32], CCCustom<"RetCC_Sparc64_Half">>>,
  CCIfType<[i32], CCPromoteToType<i64>>,
  CCCustom<"RetCC_Sparc64_Full">
]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 141-146
```tablegen
// Callee-saved registers are handled by the register window mechanism.
def CSR : CalleeSavedRegs<(add)> {
  let OtherPreserved = (add (sequence "I%u", 0, 7),
                            (sequence "L%u", 0, 7), O6);
}

```
- **EN**: Declares TableGen records such as `CSR`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `CSR` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 147-150
```tablegen
// Callee-saved registers for calls with ReturnsTwice attribute.
def RTCSR : CalleeSavedRegs<(add)> {
  let OtherPreserved = (add I6, I7);
}
```
- **EN**: Declares TableGen records such as `RTCSR`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `RTCSR` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
