# riscv_vector_common.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/riscv_vector_common.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: RISC-V V-ext builtin class.
- **Purpose (CN)**: 声明与 `riscv_vector_common` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 849

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//==------ riscv_vector_common.td - RISC-V V-ext builtin class ------------===//
//
//  Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
//  See https://llvm.org/LICENSE.txt for license information.
//  SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines RVV builtin base class for RISC-V V-extension.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Instruction definitions
//===----------------------------------------------------------------------===//
// Each record of the class RVVBuiltin defines a collection of builtins (i.e.
// "def vadd : RVVBuiltin" will be used to define things like "vadd_vv_i32m1",
// "vadd_vv_i32m2", etc).
//
// The elements of this collection are defined by an instantiation process the
// range of which is specified by the cross product of the LMUL attribute and
// every element in the attribute TypeRange. By default builtins have LMUL = [1,
// 2, 4, 8, 1/2, 1/4, 1/8] so the process is repeated 7 times. In tablegen we
// use the Log2LMUL [0, 1, 2, 3, -1, -2, -3] to represent the LMUL.
````
- **L1 EN**: Comment explains nearby logic, constraints, or intent: `riscv_vector_common.td - RISC-V V-ext builtin class`.
  **L1 CN**: 注释解释附近代码的逻辑、约束或设计意图：`riscv_vector_common.td - RISC-V V-ext builtin class`。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, constraints, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, constraints, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释解释附近代码的逻辑、约束或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, constraints, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines RVV builtin base class for RISC-V V-extension.`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines RVV builtin base class for RISC-V V-extension.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Comment explains nearby logic, constraints, or intent: `Instruction definitions`.
  **L14 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Instruction definitions`。
- **L15 EN**: Banner comment marking a file or section boundary.
  **L15 CN**: 横幅注释，用于标记文件或章节边界。
- **L16 EN**: Comment explains nearby logic, constraints, or intent: `Each record of the class RVVBuiltin defines a collection of builtins (i.e.`.
  **L16 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each record of the class RVVBuiltin defines a collection of builtins (i.e.`。
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `"def vadd : RVVBuiltin" will be used to define things like "vadd_vv_i32m1",`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`"def vadd : RVVBuiltin" will be used to define things like "vadd_vv_i32m1",`。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `"vadd_vv_i32m2", etc).`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`"vadd_vv_i32m2", etc).`。
- **L19 EN**: Separator comment used for visual grouping.
  **L19 CN**: 用于视觉分组的分隔注释。
- **L20 EN**: Comment explains nearby logic, constraints, or intent: `The elements of this collection are defined by an instantiation process the`.
  **L20 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The elements of this collection are defined by an instantiation process the`。
- **L21 EN**: Comment explains nearby logic, constraints, or intent: `range of which is specified by the cross product of the LMUL attribute and`.
  **L21 CN**: 注释解释附近代码的逻辑、约束或设计意图：`range of which is specified by the cross product of the LMUL attribute and`。
- **L22 EN**: Comment explains nearby logic, constraints, or intent: `every element in the attribute TypeRange. By default builtins have LMUL [1,`.
  **L22 CN**: 注释解释附近代码的逻辑、约束或设计意图：`every element in the attribute TypeRange. By default builtins have LMUL [1,`。
- **L23 EN**: Comment explains nearby logic, constraints, or intent: `2, 4, 8, 1/2, 1/4, 1/8] so the process is repeated 7 times. In tablegen we`.
  **L23 CN**: 注释解释附近代码的逻辑、约束或设计意图：`2, 4, 8, 1/2, 1/4, 1/8] so the process is repeated 7 times. In tablegen we`。
- **L24 EN**: Comment explains nearby logic, constraints, or intent: `use the Log2LMUL [0, 1, 2, 3, -1, -2, -3] to represent the LMUL.`.
  **L24 CN**: 注释解释附近代码的逻辑、约束或设计意图：`use the Log2LMUL [0, 1, 2, 3, -1, -2, -3] to represent the LMUL.`。

### Lines 25-48

````tablegen
//
// LMUL represents the fact that the types of values used by that builtin are
// values generated by instructions that are executed under that LMUL. However,
// this does not mean the builtin is necessarily lowered into an instruction
// that executes under the specified LMUL. An example where this happens are
// loads and stores of masks. A mask like `vbool8_t` can be generated, for
// instance, by comparing two `__rvv_int8m1_t` (this is LMUL=1) or comparing two
// `__rvv_int16m2_t` (this is LMUL=2). The actual load or store, however, will
// be performed under LMUL=1 because mask registers are not grouped.
//
// TypeRange is a non-empty sequence of basic types:
//
//   c: int8_t (i8)
//   s: int16_t (i16)
//   i: int32_t (i32)
//   l: int64_t (i64)
//   x: float16_t (half)
//   f: float32_t (float)
//   d: float64_t (double)
//   y: bfloat16_t (bfloat16)
//
// This way, given an LMUL, a record with a TypeRange "sil" will cause the
// definition of 3 builtins. Each type "t" in the TypeRange (in this example
// they are int16_t, int32_t, int64_t) is used as a parameter that drives the
````
- **L25 EN**: Separator comment used for visual grouping.
  **L25 CN**: 用于视觉分组的分隔注释。
- **L26 EN**: Comment explains nearby logic, constraints, or intent: `LMUL represents the fact that the types of values used by that builtin are`.
  **L26 CN**: 注释解释附近代码的逻辑、约束或设计意图：`LMUL represents the fact that the types of values used by that builtin are`。
- **L27 EN**: Comment explains nearby logic, constraints, or intent: `values generated by instructions that are executed under that LMUL. However,`.
  **L27 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values generated by instructions that are executed under that LMUL. However,`。
- **L28 EN**: Comment explains nearby logic, constraints, or intent: `this does not mean the builtin is necessarily lowered into an instruction`.
  **L28 CN**: 注释解释附近代码的逻辑、约束或设计意图：`this does not mean the builtin is necessarily lowered into an instruction`。
- **L29 EN**: Comment explains nearby logic, constraints, or intent: `that executes under the specified LMUL. An example where this happens are`.
  **L29 CN**: 注释解释附近代码的逻辑、约束或设计意图：`that executes under the specified LMUL. An example where this happens are`。
- **L30 EN**: Comment explains nearby logic, constraints, or intent: `loads and stores of masks. A mask like `vbool8_t` can be generated, for`.
  **L30 CN**: 注释解释附近代码的逻辑、约束或设计意图：`loads and stores of masks. A mask like `vbool8_t` can be generated, for`。
- **L31 EN**: Comment explains nearby logic, constraints, or intent: `instance, by comparing two `__rvv_int8m1_t` (this is LMUL 1) or comparing two`.
  **L31 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instance, by comparing two `__rvv_int8m1_t` (this is LMUL 1) or comparing two`。
- **L32 EN**: Comment explains nearby logic, constraints, or intent: ``__rvv_int16m2_t` (this is LMUL 2). The actual load or store, however, will`.
  **L32 CN**: 注释解释附近代码的逻辑、约束或设计意图：``__rvv_int16m2_t` (this is LMUL 2). The actual load or store, however, will`。
- **L33 EN**: Comment explains nearby logic, constraints, or intent: `be performed under LMUL 1 because mask registers are not grouped.`.
  **L33 CN**: 注释解释附近代码的逻辑、约束或设计意图：`be performed under LMUL 1 because mask registers are not grouped.`。
- **L34 EN**: Separator comment used for visual grouping.
  **L34 CN**: 用于视觉分组的分隔注释。
- **L35 EN**: Comment explains nearby logic, constraints, or intent: `TypeRange is a non-empty sequence of basic types:`.
  **L35 CN**: 注释解释附近代码的逻辑、约束或设计意图：`TypeRange is a non-empty sequence of basic types:`。
- **L36 EN**: Separator comment used for visual grouping.
  **L36 CN**: 用于视觉分组的分隔注释。
- **L37 EN**: Comment explains nearby logic, constraints, or intent: `c: int8_t (i8)`.
  **L37 CN**: 注释解释附近代码的逻辑、约束或设计意图：`c: int8_t (i8)`。
- **L38 EN**: Comment explains nearby logic, constraints, or intent: `s: int16_t (i16)`.
  **L38 CN**: 注释解释附近代码的逻辑、约束或设计意图：`s: int16_t (i16)`。
- **L39 EN**: Comment explains nearby logic, constraints, or intent: `i: int32_t (i32)`.
  **L39 CN**: 注释解释附近代码的逻辑、约束或设计意图：`i: int32_t (i32)`。
- **L40 EN**: Comment explains nearby logic, constraints, or intent: `l: int64_t (i64)`.
  **L40 CN**: 注释解释附近代码的逻辑、约束或设计意图：`l: int64_t (i64)`。
- **L41 EN**: Comment explains nearby logic, constraints, or intent: `x: float16_t (half)`.
  **L41 CN**: 注释解释附近代码的逻辑、约束或设计意图：`x: float16_t (half)`。
- **L42 EN**: Comment explains nearby logic, constraints, or intent: `f: float32_t (float)`.
  **L42 CN**: 注释解释附近代码的逻辑、约束或设计意图：`f: float32_t (float)`。
- **L43 EN**: Comment explains nearby logic, constraints, or intent: `d: float64_t (double)`.
  **L43 CN**: 注释解释附近代码的逻辑、约束或设计意图：`d: float64_t (double)`。
- **L44 EN**: Comment explains nearby logic, constraints, or intent: `y: bfloat16_t (bfloat16)`.
  **L44 CN**: 注释解释附近代码的逻辑、约束或设计意图：`y: bfloat16_t (bfloat16)`。
- **L45 EN**: Separator comment used for visual grouping.
  **L45 CN**: 用于视觉分组的分隔注释。
- **L46 EN**: Comment explains nearby logic, constraints, or intent: `This way, given an LMUL, a record with a TypeRange "sil" will cause the`.
  **L46 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This way, given an LMUL, a record with a TypeRange "sil" will cause the`。
- **L47 EN**: Comment explains nearby logic, constraints, or intent: `definition of 3 builtins. Each type "t" in the TypeRange (in this example`.
  **L47 CN**: 注释解释附近代码的逻辑、约束或设计意图：`definition of 3 builtins. Each type "t" in the TypeRange (in this example`。
- **L48 EN**: Comment explains nearby logic, constraints, or intent: `they are int16_t, int32_t, int64_t) is used as a parameter that drives the`.
  **L48 CN**: 注释解释附近代码的逻辑、约束或设计意图：`they are int16_t, int32_t, int64_t) is used as a parameter that drives the`。

### Lines 49-72

````tablegen
// definition of that particular builtin (for the given LMUL).
//
// During the instantiation, types can be transformed or modified using type
// transformers. Given a type "t" the following primitive type transformers can
// be applied to it to yield another type.
//
//   e: type of "t" as is (identity)
//   v: computes a vector type whose element type is "t" for the current LMUL
//   w: computes a vector type identical to what 'v' computes except for the
//      element type which is twice as wide as the element type of 'v'
//   q: computes a vector type identical to what 'v' computes except for the
//      element type which is four times as wide as the element type of 'v'
//   o: computes a vector type identical to what 'v' computes except for the
//      element type which is eight times as wide as the element type of 'v'
//   d: computes a vector type identical to what 'v' computes except for the
//      LMUL which is twice as large as the LMUL of 'v'
//   m: computes a vector type identical to what 'v' computes except for the
//      element type which is bool
//   0: void type, ignores "t"
//   z: size_t, ignores "t"
//   t: ptrdiff_t, ignores "t"
//   u: unsigned long, ignores "t"
//   l: long, ignores "t"
//   f: float32, ignores "t"
````
- **L49 EN**: Comment explains nearby logic, constraints, or intent: `definition of that particular builtin (for the given LMUL).`.
  **L49 CN**: 注释解释附近代码的逻辑、约束或设计意图：`definition of that particular builtin (for the given LMUL).`。
- **L50 EN**: Separator comment used for visual grouping.
  **L50 CN**: 用于视觉分组的分隔注释。
- **L51 EN**: Comment explains nearby logic, constraints, or intent: `During the instantiation, types can be transformed or modified using type`.
  **L51 CN**: 注释解释附近代码的逻辑、约束或设计意图：`During the instantiation, types can be transformed or modified using type`。
- **L52 EN**: Comment explains nearby logic, constraints, or intent: `transformers. Given a type "t" the following primitive type transformers can`.
  **L52 CN**: 注释解释附近代码的逻辑、约束或设计意图：`transformers. Given a type "t" the following primitive type transformers can`。
- **L53 EN**: Comment explains nearby logic, constraints, or intent: `be applied to it to yield another type.`.
  **L53 CN**: 注释解释附近代码的逻辑、约束或设计意图：`be applied to it to yield another type.`。
- **L54 EN**: Separator comment used for visual grouping.
  **L54 CN**: 用于视觉分组的分隔注释。
- **L55 EN**: Comment explains nearby logic, constraints, or intent: `e: type of "t" as is (identity)`.
  **L55 CN**: 注释解释附近代码的逻辑、约束或设计意图：`e: type of "t" as is (identity)`。
- **L56 EN**: Comment explains nearby logic, constraints, or intent: `v: computes a vector type whose element type is "t" for the current LMUL`.
  **L56 CN**: 注释解释附近代码的逻辑、约束或设计意图：`v: computes a vector type whose element type is "t" for the current LMUL`。
- **L57 EN**: Comment explains nearby logic, constraints, or intent: `w: computes a vector type identical to what 'v' computes except for the`.
  **L57 CN**: 注释解释附近代码的逻辑、约束或设计意图：`w: computes a vector type identical to what 'v' computes except for the`。
- **L58 EN**: Comment explains nearby logic, constraints, or intent: `element type which is twice as wide as the element type of 'v'`.
  **L58 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element type which is twice as wide as the element type of 'v'`。
- **L59 EN**: Comment explains nearby logic, constraints, or intent: `q: computes a vector type identical to what 'v' computes except for the`.
  **L59 CN**: 注释解释附近代码的逻辑、约束或设计意图：`q: computes a vector type identical to what 'v' computes except for the`。
- **L60 EN**: Comment explains nearby logic, constraints, or intent: `element type which is four times as wide as the element type of 'v'`.
  **L60 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element type which is four times as wide as the element type of 'v'`。
- **L61 EN**: Comment explains nearby logic, constraints, or intent: `o: computes a vector type identical to what 'v' computes except for the`.
  **L61 CN**: 注释解释附近代码的逻辑、约束或设计意图：`o: computes a vector type identical to what 'v' computes except for the`。
- **L62 EN**: Comment explains nearby logic, constraints, or intent: `element type which is eight times as wide as the element type of 'v'`.
  **L62 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element type which is eight times as wide as the element type of 'v'`。
- **L63 EN**: Comment explains nearby logic, constraints, or intent: `d: computes a vector type identical to what 'v' computes except for the`.
  **L63 CN**: 注释解释附近代码的逻辑、约束或设计意图：`d: computes a vector type identical to what 'v' computes except for the`。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `LMUL which is twice as large as the LMUL of 'v'`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`LMUL which is twice as large as the LMUL of 'v'`。
- **L65 EN**: Comment explains nearby logic, constraints, or intent: `m: computes a vector type identical to what 'v' computes except for the`.
  **L65 CN**: 注释解释附近代码的逻辑、约束或设计意图：`m: computes a vector type identical to what 'v' computes except for the`。
- **L66 EN**: Comment explains nearby logic, constraints, or intent: `element type which is bool`.
  **L66 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element type which is bool`。
- **L67 EN**: Comment explains nearby logic, constraints, or intent: `0: void type, ignores "t"`.
  **L67 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: void type, ignores "t"`。
- **L68 EN**: Comment explains nearby logic, constraints, or intent: `z: size_t, ignores "t"`.
  **L68 CN**: 注释解释附近代码的逻辑、约束或设计意图：`z: size_t, ignores "t"`。
- **L69 EN**: Comment explains nearby logic, constraints, or intent: `t: ptrdiff_t, ignores "t"`.
  **L69 CN**: 注释解释附近代码的逻辑、约束或设计意图：`t: ptrdiff_t, ignores "t"`。
- **L70 EN**: Comment explains nearby logic, constraints, or intent: `u: unsigned long, ignores "t"`.
  **L70 CN**: 注释解释附近代码的逻辑、约束或设计意图：`u: unsigned long, ignores "t"`。
- **L71 EN**: Comment explains nearby logic, constraints, or intent: `l: long, ignores "t"`.
  **L71 CN**: 注释解释附近代码的逻辑、约束或设计意图：`l: long, ignores "t"`。
- **L72 EN**: Comment explains nearby logic, constraints, or intent: `f: float32, ignores "t"`.
  **L72 CN**: 注释解释附近代码的逻辑、约束或设计意图：`f: float32, ignores "t"`。

### Lines 73-96

````tablegen
//
// So for instance if t is "i", i.e. int, then "e" will yield int again. "v"
// will yield an RVV vector type (assume LMUL=1), so __rvv_int32m1_t.
// Accordingly "w" would yield __rvv_int64m2_t.
//
// A type transformer can be prefixed by other non-primitive type transformers.
//
//   P: constructs a pointer to the current type
//   C: adds const to the type
//   K: requires the integer type to be a constant expression
//   U: given an integer type or vector type, computes its unsigned variant
//   I: given a vector type, compute the vector type with integer type
//      elements of the same width
//   F: given a vector type, compute the vector type with floating-point type
//      elements of the same width
//   Y: given a vector type, compute the vector type with bfloat16 type elements
//      of the same width
//   S: given a vector type, computes its equivalent one for LMUL=1. This is a
//      no-op if the vector was already LMUL=1
//   (Log2EEW:Value): Log2EEW value could be 3/4/5/6 (8/16/32/64), given a
//      vector type (SEW and LMUL) and EEW (8/16/32/64), computes its
//      equivalent integer vector type with EEW and corresponding ELMUL (elmul =
//      (eew/sew) * lmul). For example, vector type is __rvv_float16m4
//      (SEW=16, LMUL=4) and Log2EEW is 3 (EEW=8), and then equivalent vector
````
- **L73 EN**: Separator comment used for visual grouping.
  **L73 CN**: 用于视觉分组的分隔注释。
- **L74 EN**: Comment explains nearby logic, constraints, or intent: `So for instance if t is "i", i.e. int, then "e" will yield int again. "v"`.
  **L74 CN**: 注释解释附近代码的逻辑、约束或设计意图：`So for instance if t is "i", i.e. int, then "e" will yield int again. "v"`。
- **L75 EN**: Comment explains nearby logic, constraints, or intent: `will yield an RVV vector type (assume LMUL 1), so __rvv_int32m1_t.`.
  **L75 CN**: 注释解释附近代码的逻辑、约束或设计意图：`will yield an RVV vector type (assume LMUL 1), so __rvv_int32m1_t.`。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `Accordingly "w" would yield __rvv_int64m2_t.`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Accordingly "w" would yield __rvv_int64m2_t.`。
- **L77 EN**: Separator comment used for visual grouping.
  **L77 CN**: 用于视觉分组的分隔注释。
- **L78 EN**: Comment explains nearby logic, constraints, or intent: `A type transformer can be prefixed by other non-primitive type transformers.`.
  **L78 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A type transformer can be prefixed by other non-primitive type transformers.`。
- **L79 EN**: Separator comment used for visual grouping.
  **L79 CN**: 用于视觉分组的分隔注释。
- **L80 EN**: Comment explains nearby logic, constraints, or intent: `P: constructs a pointer to the current type`.
  **L80 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P: constructs a pointer to the current type`。
- **L81 EN**: Comment explains nearby logic, constraints, or intent: `C: adds const to the type`.
  **L81 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C: adds const to the type`。
- **L82 EN**: Comment explains nearby logic, constraints, or intent: `K: requires the integer type to be a constant expression`.
  **L82 CN**: 注释解释附近代码的逻辑、约束或设计意图：`K: requires the integer type to be a constant expression`。
- **L83 EN**: Comment explains nearby logic, constraints, or intent: `U: given an integer type or vector type, computes its unsigned variant`.
  **L83 CN**: 注释解释附近代码的逻辑、约束或设计意图：`U: given an integer type or vector type, computes its unsigned variant`。
- **L84 EN**: Comment explains nearby logic, constraints, or intent: `I: given a vector type, compute the vector type with integer type`.
  **L84 CN**: 注释解释附近代码的逻辑、约束或设计意图：`I: given a vector type, compute the vector type with integer type`。
- **L85 EN**: Comment explains nearby logic, constraints, or intent: `elements of the same width`.
  **L85 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements of the same width`。
- **L86 EN**: Comment explains nearby logic, constraints, or intent: `F: given a vector type, compute the vector type with floating-point type`.
  **L86 CN**: 注释解释附近代码的逻辑、约束或设计意图：`F: given a vector type, compute the vector type with floating-point type`。
- **L87 EN**: Comment explains nearby logic, constraints, or intent: `elements of the same width`.
  **L87 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements of the same width`。
- **L88 EN**: Comment explains nearby logic, constraints, or intent: `Y: given a vector type, compute the vector type with bfloat16 type elements`.
  **L88 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Y: given a vector type, compute the vector type with bfloat16 type elements`。
- **L89 EN**: Comment explains nearby logic, constraints, or intent: `of the same width`.
  **L89 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the same width`。
- **L90 EN**: Comment explains nearby logic, constraints, or intent: `S: given a vector type, computes its equivalent one for LMUL 1. This is a`.
  **L90 CN**: 注释解释附近代码的逻辑、约束或设计意图：`S: given a vector type, computes its equivalent one for LMUL 1. This is a`。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `no-op if the vector was already LMUL 1`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`no-op if the vector was already LMUL 1`。
- **L92 EN**: Comment explains nearby logic, constraints, or intent: `(Log2EEW:Value): Log2EEW value could be 3/4/5/6 (8/16/32/64), given a`.
  **L92 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(Log2EEW:Value): Log2EEW value could be 3/4/5/6 (8/16/32/64), given a`。
- **L93 EN**: Comment explains nearby logic, constraints, or intent: `vector type (SEW and LMUL) and EEW (8/16/32/64), computes its`.
  **L93 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector type (SEW and LMUL) and EEW (8/16/32/64), computes its`。
- **L94 EN**: Comment explains nearby logic, constraints, or intent: `equivalent integer vector type with EEW and corresponding ELMUL (elmul`.
  **L94 CN**: 注释解释附近代码的逻辑、约束或设计意图：`equivalent integer vector type with EEW and corresponding ELMUL (elmul`。
- **L95 EN**: Comment explains nearby logic, constraints, or intent: `(eew/sew) * lmul). For example, vector type is __rvv_float16m4`.
  **L95 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(eew/sew) * lmul). For example, vector type is __rvv_float16m4`。
- **L96 EN**: Comment explains nearby logic, constraints, or intent: `(SEW 16, LMUL 4) and Log2EEW is 3 (EEW 8), and then equivalent vector`.
  **L96 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(SEW 16, LMUL 4) and Log2EEW is 3 (EEW 8), and then equivalent vector`。

### Lines 97-120

````tablegen
//      type is __rvv_uint8m2_t (elmul=(8/16)*4 = 2). Ignore to define a new
//      builtins if its equivalent type has illegal lmul.
//   (FixedSEW:Value): Given a vector type (SEW and LMUL), and computes another
//      vector type which only changed SEW as given value. Ignore to define a new
//      builtin if its equivalent type has illegal lmul or the SEW does not changed.
//   (SFixedLog2LMUL:Value): Smaller Fixed Log2LMUL. Given a vector type (SEW
//      and LMUL), and computes another vector type which only changed LMUL as
//      given value. The new LMUL should be smaller than the old one. Ignore to
//      define a new builtin if its equivalent type has illegal lmul.
//   (SEFixedLog2LMUL:Value): Smaller or Equal Fixed Log2LMUL. Given a vector
//      type (SEW and LMUL), and computes another vector type which only
//      changed LMUL as given value. The new LMUL should be smaller than or
//      equal to the old one. Ignore to define a new builtin if its equivalent
//      type has illegal lmul.
//   (LFixedLog2LMUL:Value): Larger Fixed Log2LMUL. Given a vector type (SEW
//      and LMUL), and computes another vector type which only changed LMUL as
//      given value. The new LMUL should be larger than the old one. Ignore to
//      define a new builtin if its equivalent type has illegal lmul.
//
// Following with the example above, if t is "i", then "Ue" will yield unsigned
// int and "Fv" will yield __rvv_float32m1_t (again assuming LMUL=1), Fw would
// yield __rvv_float64m2_t, etc.
//
// Each builtin is then defined by applying each type in TypeRange against the
````
- **L97 EN**: Comment explains nearby logic, constraints, or intent: `type is __rvv_uint8m2_t (elmul (8/16)*4 2). Ignore to define a new`.
  **L97 CN**: 注释解释附近代码的逻辑、约束或设计意图：`type is __rvv_uint8m2_t (elmul (8/16)*4 2). Ignore to define a new`。
- **L98 EN**: Comment explains nearby logic, constraints, or intent: `builtins if its equivalent type has illegal lmul.`.
  **L98 CN**: 注释解释附近代码的逻辑、约束或设计意图：`builtins if its equivalent type has illegal lmul.`。
- **L99 EN**: Comment explains nearby logic, constraints, or intent: `(FixedSEW:Value): Given a vector type (SEW and LMUL), and computes another`.
  **L99 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(FixedSEW:Value): Given a vector type (SEW and LMUL), and computes another`。
- **L100 EN**: Comment explains nearby logic, constraints, or intent: `vector type which only changed SEW as given value. Ignore to define a new`.
  **L100 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector type which only changed SEW as given value. Ignore to define a new`。
- **L101 EN**: Comment explains nearby logic, constraints, or intent: `builtin if its equivalent type has illegal lmul or the SEW does not changed.`.
  **L101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`builtin if its equivalent type has illegal lmul or the SEW does not changed.`。
- **L102 EN**: Comment explains nearby logic, constraints, or intent: `(SFixedLog2LMUL:Value): Smaller Fixed Log2LMUL. Given a vector type (SEW`.
  **L102 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(SFixedLog2LMUL:Value): Smaller Fixed Log2LMUL. Given a vector type (SEW`。
- **L103 EN**: Comment explains nearby logic, constraints, or intent: `and LMUL), and computes another vector type which only changed LMUL as`.
  **L103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and LMUL), and computes another vector type which only changed LMUL as`。
- **L104 EN**: Comment explains nearby logic, constraints, or intent: `given value. The new LMUL should be smaller than the old one. Ignore to`.
  **L104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given value. The new LMUL should be smaller than the old one. Ignore to`。
- **L105 EN**: Comment explains nearby logic, constraints, or intent: `define a new builtin if its equivalent type has illegal lmul.`.
  **L105 CN**: 注释解释附近代码的逻辑、约束或设计意图：`define a new builtin if its equivalent type has illegal lmul.`。
- **L106 EN**: Comment explains nearby logic, constraints, or intent: `(SEFixedLog2LMUL:Value): Smaller or Equal Fixed Log2LMUL. Given a vector`.
  **L106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(SEFixedLog2LMUL:Value): Smaller or Equal Fixed Log2LMUL. Given a vector`。
- **L107 EN**: Comment explains nearby logic, constraints, or intent: `type (SEW and LMUL), and computes another vector type which only`.
  **L107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`type (SEW and LMUL), and computes another vector type which only`。
- **L108 EN**: Comment explains nearby logic, constraints, or intent: `changed LMUL as given value. The new LMUL should be smaller than or`.
  **L108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`changed LMUL as given value. The new LMUL should be smaller than or`。
- **L109 EN**: Comment explains nearby logic, constraints, or intent: `equal to the old one. Ignore to define a new builtin if its equivalent`.
  **L109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`equal to the old one. Ignore to define a new builtin if its equivalent`。
- **L110 EN**: Comment explains nearby logic, constraints, or intent: `type has illegal lmul.`.
  **L110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`type has illegal lmul.`。
- **L111 EN**: Comment explains nearby logic, constraints, or intent: `(LFixedLog2LMUL:Value): Larger Fixed Log2LMUL. Given a vector type (SEW`.
  **L111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(LFixedLog2LMUL:Value): Larger Fixed Log2LMUL. Given a vector type (SEW`。
- **L112 EN**: Comment explains nearby logic, constraints, or intent: `and LMUL), and computes another vector type which only changed LMUL as`.
  **L112 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and LMUL), and computes another vector type which only changed LMUL as`。
- **L113 EN**: Comment explains nearby logic, constraints, or intent: `given value. The new LMUL should be larger than the old one. Ignore to`.
  **L113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given value. The new LMUL should be larger than the old one. Ignore to`。
- **L114 EN**: Comment explains nearby logic, constraints, or intent: `define a new builtin if its equivalent type has illegal lmul.`.
  **L114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`define a new builtin if its equivalent type has illegal lmul.`。
- **L115 EN**: Separator comment used for visual grouping.
  **L115 CN**: 用于视觉分组的分隔注释。
- **L116 EN**: Comment explains nearby logic, constraints, or intent: `Following with the example above, if t is "i", then "Ue" will yield unsigned`.
  **L116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Following with the example above, if t is "i", then "Ue" will yield unsigned`。
- **L117 EN**: Comment explains nearby logic, constraints, or intent: `int and "Fv" will yield __rvv_float32m1_t (again assuming LMUL 1), Fw would`.
  **L117 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int and "Fv" will yield __rvv_float32m1_t (again assuming LMUL 1), Fw would`。
- **L118 EN**: Comment explains nearby logic, constraints, or intent: `yield __rvv_float64m2_t, etc.`.
  **L118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`yield __rvv_float64m2_t, etc.`。
- **L119 EN**: Separator comment used for visual grouping.
  **L119 CN**: 用于视觉分组的分隔注释。
- **L120 EN**: Comment explains nearby logic, constraints, or intent: `Each builtin is then defined by applying each type in TypeRange against the`.
  **L120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each builtin is then defined by applying each type in TypeRange against the`。

### Lines 121-144

````tablegen
// sequence of type transformers described in Suffix and Prototype.
//
// The name of the builtin is defined by the Name attribute (which defaults to
// the name of the class) appended (separated with an underscore) the Suffix
// attribute. For instance with Name="foo", Suffix = "v" and TypeRange = "il",
// the builtin generated will be __builtin_rvv_foo_i32m1 and
// __builtin_rvv_foo_i64m1 (under LMUL=1). If Suffix contains more than one
// type transformer (say "vv") each of the types is separated with an
// underscore as in "__builtin_rvv_foo_i32m1_i32m1".
//
// The C/C++ prototype of the builtin is defined by the Prototype attribute.
// Prototype is a non-empty sequence of type transformers, the first of which
// is the return type of the builtin and the rest are the parameters of the
// builtin, in order. For instance if Prototype is "wvv" and TypeRange is "si"
// a first builtin will have type
// __rvv_int32m2_t (__rvv_int16m1_t, __rvv_int16m1_t) and the second builtin
// will have type __rvv_int64m2_t (__rvv_int32m1_t, __rvv_int32m1_t) (again
// under LMUL=1).
//
// There are a number of attributes that are used to constraint the number and
// shape of the builtins generated. Refer to the comments below for them.

class PolicyScheme<int val>{
  int Value = val;
````
- **L121 EN**: Comment explains nearby logic, constraints, or intent: `sequence of type transformers described in Suffix and Prototype.`.
  **L121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sequence of type transformers described in Suffix and Prototype.`。
- **L122 EN**: Separator comment used for visual grouping.
  **L122 CN**: 用于视觉分组的分隔注释。
- **L123 EN**: Comment explains nearby logic, constraints, or intent: `The name of the builtin is defined by the Name attribute (which defaults to`.
  **L123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The name of the builtin is defined by the Name attribute (which defaults to`。
- **L124 EN**: Comment explains nearby logic, constraints, or intent: `the name of the class) appended (separated with an underscore) the Suffix`.
  **L124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the name of the class) appended (separated with an underscore) the Suffix`。
- **L125 EN**: Comment explains nearby logic, constraints, or intent: `attribute. For instance with Name "foo", Suffix "v" and TypeRange "il",`.
  **L125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`attribute. For instance with Name "foo", Suffix "v" and TypeRange "il",`。
- **L126 EN**: Comment explains nearby logic, constraints, or intent: `the builtin generated will be __builtin_rvv_foo_i32m1 and`.
  **L126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the builtin generated will be __builtin_rvv_foo_i32m1 and`。
- **L127 EN**: Comment explains nearby logic, constraints, or intent: `__builtin_rvv_foo_i64m1 (under LMUL 1). If Suffix contains more than one`.
  **L127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__builtin_rvv_foo_i64m1 (under LMUL 1). If Suffix contains more than one`。
- **L128 EN**: Comment explains nearby logic, constraints, or intent: `type transformer (say "vv") each of the types is separated with an`.
  **L128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`type transformer (say "vv") each of the types is separated with an`。
- **L129 EN**: Comment explains nearby logic, constraints, or intent: `underscore as in "__builtin_rvv_foo_i32m1_i32m1".`.
  **L129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`underscore as in "__builtin_rvv_foo_i32m1_i32m1".`。
- **L130 EN**: Separator comment used for visual grouping.
  **L130 CN**: 用于视觉分组的分隔注释。
- **L131 EN**: Comment explains nearby logic, constraints, or intent: `The C/C++ prototype of the builtin is defined by the Prototype attribute.`.
  **L131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The C/C++ prototype of the builtin is defined by the Prototype attribute.`。
- **L132 EN**: Comment explains nearby logic, constraints, or intent: `Prototype is a non-empty sequence of type transformers, the first of which`.
  **L132 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prototype is a non-empty sequence of type transformers, the first of which`。
- **L133 EN**: Comment explains nearby logic, constraints, or intent: `is the return type of the builtin and the rest are the parameters of the`.
  **L133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is the return type of the builtin and the rest are the parameters of the`。
- **L134 EN**: Comment explains nearby logic, constraints, or intent: `builtin, in order. For instance if Prototype is "wvv" and TypeRange is "si"`.
  **L134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`builtin, in order. For instance if Prototype is "wvv" and TypeRange is "si"`。
- **L135 EN**: Comment explains nearby logic, constraints, or intent: `a first builtin will have type`.
  **L135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a first builtin will have type`。
- **L136 EN**: Comment explains nearby logic, constraints, or intent: `__rvv_int32m2_t (__rvv_int16m1_t, __rvv_int16m1_t) and the second builtin`.
  **L136 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__rvv_int32m2_t (__rvv_int16m1_t, __rvv_int16m1_t) and the second builtin`。
- **L137 EN**: Comment explains nearby logic, constraints, or intent: `will have type __rvv_int64m2_t (__rvv_int32m1_t, __rvv_int32m1_t) (again`.
  **L137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`will have type __rvv_int64m2_t (__rvv_int32m1_t, __rvv_int32m1_t) (again`。
- **L138 EN**: Comment explains nearby logic, constraints, or intent: `under LMUL 1).`.
  **L138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`under LMUL 1).`。
- **L139 EN**: Separator comment used for visual grouping.
  **L139 CN**: 用于视觉分组的分隔注释。
- **L140 EN**: Comment explains nearby logic, constraints, or intent: `There are a number of attributes that are used to constraint the number and`.
  **L140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`There are a number of attributes that are used to constraint the number and`。
- **L141 EN**: Comment explains nearby logic, constraints, or intent: `shape of the builtins generated. Refer to the comments below for them.`.
  **L141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`shape of the builtins generated. Refer to the comments below for them.`。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L143 EN**: Declares TableGen class record `PolicyScheme`.
  **L143 CN**: 声明 TableGen class 记录 `PolicyScheme`。
- **L144 EN**: Initializes variable `Value` from the expression on the right-hand side.
  **L144 CN**: 使用右侧表达式初始化变量 `Value`。

### Lines 145-168

````tablegen
}
def NonePolicy : PolicyScheme<0>;
def HasPassthruOperand : PolicyScheme<1>;
def HasPolicyOperand : PolicyScheme<2>;

class RVVBuiltin<string suffix, string prototype, string type_range,
                 string overloaded_suffix = ""> {
  // Base name that will be prepended in __builtin_rvv_ and appended the
  // computed Suffix.
  string Name = NAME;

  // If not empty, each instantiated builtin will have this appended after an
  // underscore (_). It is instantiated like Prototype.
  string Suffix = suffix;

  // If empty, default OverloadedName is sub string of `Name` which end of first
  // '_'. For example, the default overloaded name  is `vadd` for Name `vadd_vv`.
  // It's used for describe some special naming cases.
  string OverloadedName = "";

  // If not empty, each OverloadedName will have this appended after an
  // underscore (_). It is instantiated like Prototype.
  string OverloadedSuffix = overloaded_suffix;

````
- **L145 EN**: Closes the current lexical scope or compound statement.
  **L145 CN**: 结束当前词法作用域或复合语句块。
- **L146 EN**: Declares TableGen def record `NonePolicy`.
  **L146 CN**: 声明 TableGen def 记录 `NonePolicy`。
- **L147 EN**: Declares TableGen def record `HasPassthruOperand`.
  **L147 CN**: 声明 TableGen def 记录 `HasPassthruOperand`。
- **L148 EN**: Declares TableGen def record `HasPolicyOperand`.
  **L148 CN**: 声明 TableGen def 记录 `HasPolicyOperand`。
- **L149 EN**: Blank line separating nearby declarations or logic blocks.
  **L149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L150 EN**: Declares TableGen class record `RVVBuiltin`.
  **L150 CN**: 声明 TableGen class 记录 `RVVBuiltin`。
- **L151 EN**: Continues the surrounding expression or declaration: `string overloaded_suffix = ""> {`.
  **L151 CN**: 继续构造周围的表达式或声明：`string overloaded_suffix = ""> {`。
- **L152 EN**: Comment explains nearby logic, constraints, or intent: `Base name that will be prepended in __builtin_rvv_ and appended the`.
  **L152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Base name that will be prepended in __builtin_rvv_ and appended the`。
- **L153 EN**: Comment explains nearby logic, constraints, or intent: `computed Suffix.`.
  **L153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`computed Suffix.`。
- **L154 EN**: Initializes variable `Name` from the expression on the right-hand side.
  **L154 CN**: 使用右侧表达式初始化变量 `Name`。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L156 EN**: Comment explains nearby logic, constraints, or intent: `If not empty, each instantiated builtin will have this appended after an`.
  **L156 CN**: 注释解释附近代码的逻辑、约束或设计意图：`If not empty, each instantiated builtin will have this appended after an`。
- **L157 EN**: Comment explains nearby logic, constraints, or intent: `underscore (_). It is instantiated like Prototype.`.
  **L157 CN**: 注释解释附近代码的逻辑、约束或设计意图：`underscore (_). It is instantiated like Prototype.`。
- **L158 EN**: Initializes variable `Suffix` from the expression on the right-hand side.
  **L158 CN**: 使用右侧表达式初始化变量 `Suffix`。
- **L159 EN**: Blank line separating nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L160 EN**: Comment explains nearby logic, constraints, or intent: `If empty, default OverloadedName is sub string of `Name` which end of first`.
  **L160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`If empty, default OverloadedName is sub string of `Name` which end of first`。
- **L161 EN**: Comment explains nearby logic, constraints, or intent: `'_'. For example, the default overloaded name is `vadd` for Name `vadd_vv`.`.
  **L161 CN**: 注释解释附近代码的逻辑、约束或设计意图：`'_'. For example, the default overloaded name is `vadd` for Name `vadd_vv`.`。
- **L162 EN**: Comment explains nearby logic, constraints, or intent: `It's used for describe some special naming cases.`.
  **L162 CN**: 注释解释附近代码的逻辑、约束或设计意图：`It's used for describe some special naming cases.`。
- **L163 EN**: Initializes variable `OverloadedName` from the expression on the right-hand side.
  **L163 CN**: 使用右侧表达式初始化变量 `OverloadedName`。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Comment explains nearby logic, constraints, or intent: `If not empty, each OverloadedName will have this appended after an`.
  **L165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`If not empty, each OverloadedName will have this appended after an`。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `underscore (_). It is instantiated like Prototype.`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`underscore (_). It is instantiated like Prototype.`。
- **L167 EN**: Initializes variable `OverloadedSuffix` from the expression on the right-hand side.
  **L167 CN**: 使用右侧表达式初始化变量 `OverloadedSuffix`。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 169-192

````tablegen
  // The different variants of the builtin, parameterised with a type.
  string TypeRange = type_range;

  // We use each type described in TypeRange and LMUL with prototype to
  // instantiate a specific element of the set of builtins being defined.
  // Prototype attribute defines the C/C++ prototype of the builtin. It is a
  // non-empty sequence of type transformers, the first of which is the return
  // type of the builtin and the rest are the parameters of the builtin, in
  // order. For instance if Prototype is "wvv", TypeRange is "si" and LMUL=1, a
  // first builtin will have type
  // __rvv_int32m2_t (__rvv_int16m1_t, __rvv_int16m1_t), and the second builtin
  // will have type __rvv_int64m2_t (__rvv_int32m1_t, __rvv_int32m1_t).
  string Prototype = prototype;

  // This builtin has a masked form.
  bit HasMasked = true;

  // This is used by intrinsics that need vtype.altfmt
  bit AltFmt = false;

  // If HasMasked, this flag states that this builtin has a maskedoff operand. It
  // is always the first operand in builtin and IR intrinsic.
  bit HasMaskedOffOperand = true;

````
- **L169 EN**: Comment explains nearby logic, constraints, or intent: `The different variants of the builtin, parameterised with a type.`.
  **L169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The different variants of the builtin, parameterised with a type.`。
- **L170 EN**: Initializes variable `TypeRange` from the expression on the right-hand side.
  **L170 CN**: 使用右侧表达式初始化变量 `TypeRange`。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L172 EN**: Comment explains nearby logic, constraints, or intent: `We use each type described in TypeRange and LMUL with prototype to`.
  **L172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`We use each type described in TypeRange and LMUL with prototype to`。
- **L173 EN**: Comment explains nearby logic, constraints, or intent: `instantiate a specific element of the set of builtins being defined.`.
  **L173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instantiate a specific element of the set of builtins being defined.`。
- **L174 EN**: Comment explains nearby logic, constraints, or intent: `Prototype attribute defines the C/C++ prototype of the builtin. It is a`.
  **L174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prototype attribute defines the C/C++ prototype of the builtin. It is a`。
- **L175 EN**: Comment explains nearby logic, constraints, or intent: `non-empty sequence of type transformers, the first of which is the return`.
  **L175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`non-empty sequence of type transformers, the first of which is the return`。
- **L176 EN**: Comment explains nearby logic, constraints, or intent: `type of the builtin and the rest are the parameters of the builtin, in`.
  **L176 CN**: 注释解释附近代码的逻辑、约束或设计意图：`type of the builtin and the rest are the parameters of the builtin, in`。
- **L177 EN**: Comment explains nearby logic, constraints, or intent: `order. For instance if Prototype is "wvv", TypeRange is "si" and LMUL 1, a`.
  **L177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`order. For instance if Prototype is "wvv", TypeRange is "si" and LMUL 1, a`。
- **L178 EN**: Comment explains nearby logic, constraints, or intent: `first builtin will have type`.
  **L178 CN**: 注释解释附近代码的逻辑、约束或设计意图：`first builtin will have type`。
- **L179 EN**: Comment explains nearby logic, constraints, or intent: `__rvv_int32m2_t (__rvv_int16m1_t, __rvv_int16m1_t), and the second builtin`.
  **L179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__rvv_int32m2_t (__rvv_int16m1_t, __rvv_int16m1_t), and the second builtin`。
- **L180 EN**: Comment explains nearby logic, constraints, or intent: `will have type __rvv_int64m2_t (__rvv_int32m1_t, __rvv_int32m1_t).`.
  **L180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`will have type __rvv_int64m2_t (__rvv_int32m1_t, __rvv_int32m1_t).`。
- **L181 EN**: Initializes variable `Prototype` from the expression on the right-hand side.
  **L181 CN**: 使用右侧表达式初始化变量 `Prototype`。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L183 EN**: Comment explains nearby logic, constraints, or intent: `This builtin has a masked form.`.
  **L183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This builtin has a masked form.`。
- **L184 EN**: Initializes variable `HasMasked` from the expression on the right-hand side.
  **L184 CN**: 使用右侧表达式初始化变量 `HasMasked`。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L186 EN**: Comment explains nearby logic, constraints, or intent: `This is used by intrinsics that need vtype.altfmt`.
  **L186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This is used by intrinsics that need vtype.altfmt`。
- **L187 EN**: Initializes variable `AltFmt` from the expression on the right-hand side.
  **L187 CN**: 使用右侧表达式初始化变量 `AltFmt`。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L189 EN**: Comment explains nearby logic, constraints, or intent: `If HasMasked, this flag states that this builtin has a maskedoff operand. It`.
  **L189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`If HasMasked, this flag states that this builtin has a maskedoff operand. It`。
- **L190 EN**: Comment explains nearby logic, constraints, or intent: `is always the first operand in builtin and IR intrinsic.`.
  **L190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is always the first operand in builtin and IR intrinsic.`。
- **L191 EN**: Initializes variable `HasMaskedOffOperand` from the expression on the right-hand side.
  **L191 CN**: 使用右侧表达式初始化变量 `HasMaskedOffOperand`。
- **L192 EN**: Blank line separating nearby declarations or logic blocks.
  **L192 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 193-216

````tablegen
  // This builtin has a granted vector length parameter.
  bit HasVL = true;

  // The policy scheme for masked intrinsic IR.
  // It could be NonePolicy or HasPolicyOperand.
  // HasPolicyOperand: Has a policy operand. 0 is tail and mask undisturbed, 1 is
  // tail agnostic, 2 is mask undisturbed, and 3 is tail and mask agnostic. The
  // policy operand is located at the last position.
  PolicyScheme MaskedPolicyScheme = HasPolicyOperand;

  // The policy scheme for unmasked intrinsic IR.
  // It could be NonePolicy, HasPassthruOperand or HasPolicyOperand.
  // HasPassthruOperand: Has a passthru operand to decide tail policy. If it is
  // poison, tail policy is tail agnostic, otherwise policy is tail undisturbed.
  // HasPolicyOperand: Has a policy operand. 1 is tail agnostic and 0 is tail
  // undisturbed.
  PolicyScheme UnMaskedPolicyScheme = NonePolicy;

  // This builtin support tail agnostic and undisturbed policy.
  bit HasTailPolicy = true;
  // This builtin support mask agnostic and undisturbed policy.
  bit HasMaskPolicy = true;

  // This builtin prototype with TA or TAMA policy could not support overloading
````
- **L193 EN**: Comment explains nearby logic, constraints, or intent: `This builtin has a granted vector length parameter.`.
  **L193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This builtin has a granted vector length parameter.`。
- **L194 EN**: Initializes variable `HasVL` from the expression on the right-hand side.
  **L194 CN**: 使用右侧表达式初始化变量 `HasVL`。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L196 EN**: Comment explains nearby logic, constraints, or intent: `The policy scheme for masked intrinsic IR.`.
  **L196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The policy scheme for masked intrinsic IR.`。
- **L197 EN**: Comment explains nearby logic, constraints, or intent: `It could be NonePolicy or HasPolicyOperand.`.
  **L197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`It could be NonePolicy or HasPolicyOperand.`。
- **L198 EN**: Comment explains nearby logic, constraints, or intent: `HasPolicyOperand: Has a policy operand. 0 is tail and mask undisturbed, 1 is`.
  **L198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HasPolicyOperand: Has a policy operand. 0 is tail and mask undisturbed, 1 is`。
- **L199 EN**: Comment explains nearby logic, constraints, or intent: `tail agnostic, 2 is mask undisturbed, and 3 is tail and mask agnostic. The`.
  **L199 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tail agnostic, 2 is mask undisturbed, and 3 is tail and mask agnostic. The`。
- **L200 EN**: Comment explains nearby logic, constraints, or intent: `policy operand is located at the last position.`.
  **L200 CN**: 注释解释附近代码的逻辑、约束或设计意图：`policy operand is located at the last position.`。
- **L201 EN**: Initializes variable `MaskedPolicyScheme` from the expression on the right-hand side.
  **L201 CN**: 使用右侧表达式初始化变量 `MaskedPolicyScheme`。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L203 EN**: Comment explains nearby logic, constraints, or intent: `The policy scheme for unmasked intrinsic IR.`.
  **L203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The policy scheme for unmasked intrinsic IR.`。
- **L204 EN**: Comment explains nearby logic, constraints, or intent: `It could be NonePolicy, HasPassthruOperand or HasPolicyOperand.`.
  **L204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`It could be NonePolicy, HasPassthruOperand or HasPolicyOperand.`。
- **L205 EN**: Comment explains nearby logic, constraints, or intent: `HasPassthruOperand: Has a passthru operand to decide tail policy. If it is`.
  **L205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HasPassthruOperand: Has a passthru operand to decide tail policy. If it is`。
- **L206 EN**: Comment explains nearby logic, constraints, or intent: `poison, tail policy is tail agnostic, otherwise policy is tail undisturbed.`.
  **L206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`poison, tail policy is tail agnostic, otherwise policy is tail undisturbed.`。
- **L207 EN**: Comment explains nearby logic, constraints, or intent: `HasPolicyOperand: Has a policy operand. 1 is tail agnostic and 0 is tail`.
  **L207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HasPolicyOperand: Has a policy operand. 1 is tail agnostic and 0 is tail`。
- **L208 EN**: Comment explains nearby logic, constraints, or intent: `undisturbed.`.
  **L208 CN**: 注释解释附近代码的逻辑、约束或设计意图：`undisturbed.`。
- **L209 EN**: Initializes variable `UnMaskedPolicyScheme` from the expression on the right-hand side.
  **L209 CN**: 使用右侧表达式初始化变量 `UnMaskedPolicyScheme`。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L211 EN**: Comment explains nearby logic, constraints, or intent: `This builtin support tail agnostic and undisturbed policy.`.
  **L211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This builtin support tail agnostic and undisturbed policy.`。
- **L212 EN**: Initializes variable `HasTailPolicy` from the expression on the right-hand side.
  **L212 CN**: 使用右侧表达式初始化变量 `HasTailPolicy`。
- **L213 EN**: Comment explains nearby logic, constraints, or intent: `This builtin support mask agnostic and undisturbed policy.`.
  **L213 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This builtin support mask agnostic and undisturbed policy.`。
- **L214 EN**: Initializes variable `HasMaskPolicy` from the expression on the right-hand side.
  **L214 CN**: 使用右侧表达式初始化变量 `HasMaskPolicy`。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L216 EN**: Comment explains nearby logic, constraints, or intent: `This builtin prototype with TA or TAMA policy could not support overloading`.
  **L216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This builtin prototype with TA or TAMA policy could not support overloading`。

### Lines 217-240

````tablegen
  // API. Other policy intrinsic functions would support overloading API with
  // suffix `_tu`, `tumu`, `tuma`, `tamu` and `tama`.
  bit SupportOverloading = true;

  // This builtin is valid for the given Log2LMULs.
  list<int> Log2LMUL = [0, 1, 2, 3, -1, -2, -3];

  // Manual code in clang codegen riscv_vector_builtin_cg.inc
  code ManualCodegen = [{}];

  // When emit the automatic clang codegen, it describes what types we have to use
  // to obtain the specific LLVM intrinsic. -1 means the return type, otherwise,
  // k >= 0 meaning the k-th operand (counting from zero) of the codegen'd
  // parameter of the unmasked version. k can't be the mask operand's position.
  list<int> IntrinsicTypes = [];

  // If these names are not empty, this is the ID of the LLVM intrinsic
  // we want to lower to.
  string IRName = NAME;

  // If HasMasked, this is the ID of the LLVM intrinsic we want to lower to.
  string MaskedIRName = NAME #"_mask";

  // Use clang_builtin_alias to save the number of builtins.
````
- **L217 EN**: Comment explains nearby logic, constraints, or intent: `API. Other policy intrinsic functions would support overloading API with`.
  **L217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`API. Other policy intrinsic functions would support overloading API with`。
- **L218 EN**: Comment explains nearby logic, constraints, or intent: `suffix `_tu`, `tumu`, `tuma`, `tamu` and `tama`.`.
  **L218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`suffix `_tu`, `tumu`, `tuma`, `tamu` and `tama`.`。
- **L219 EN**: Initializes variable `SupportOverloading` from the expression on the right-hand side.
  **L219 CN**: 使用右侧表达式初始化变量 `SupportOverloading`。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L221 EN**: Comment explains nearby logic, constraints, or intent: `This builtin is valid for the given Log2LMULs.`.
  **L221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This builtin is valid for the given Log2LMULs.`。
- **L222 EN**: Initializes variable `Log2LMUL` from the expression on the right-hand side.
  **L222 CN**: 使用右侧表达式初始化变量 `Log2LMUL`。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L224 EN**: Comment explains nearby logic, constraints, or intent: `Manual code in clang codegen riscv_vector_builtin_cg.inc`.
  **L224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Manual code in clang codegen riscv_vector_builtin_cg.inc`。
- **L225 EN**: Initializes variable `ManualCodegen` from the expression on the right-hand side.
  **L225 CN**: 使用右侧表达式初始化变量 `ManualCodegen`。
- **L226 EN**: Blank line separating nearby declarations or logic blocks.
  **L226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L227 EN**: Comment explains nearby logic, constraints, or intent: `When emit the automatic clang codegen, it describes what types we have to use`.
  **L227 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When emit the automatic clang codegen, it describes what types we have to use`。
- **L228 EN**: Comment explains nearby logic, constraints, or intent: `to obtain the specific LLVM intrinsic. -1 means the return type, otherwise,`.
  **L228 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to obtain the specific LLVM intrinsic. -1 means the return type, otherwise,`。
- **L229 EN**: Comment explains nearby logic, constraints, or intent: `k > 0 meaning the k-th operand (counting from zero) of the codegen'd`.
  **L229 CN**: 注释解释附近代码的逻辑、约束或设计意图：`k > 0 meaning the k-th operand (counting from zero) of the codegen'd`。
- **L230 EN**: Comment explains nearby logic, constraints, or intent: `parameter of the unmasked version. k can't be the mask operand's position.`.
  **L230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter of the unmasked version. k can't be the mask operand's position.`。
- **L231 EN**: Initializes variable `IntrinsicTypes` from the expression on the right-hand side.
  **L231 CN**: 使用右侧表达式初始化变量 `IntrinsicTypes`。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L233 EN**: Comment explains nearby logic, constraints, or intent: `If these names are not empty, this is the ID of the LLVM intrinsic`.
  **L233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`If these names are not empty, this is the ID of the LLVM intrinsic`。
- **L234 EN**: Comment explains nearby logic, constraints, or intent: `we want to lower to.`.
  **L234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`we want to lower to.`。
- **L235 EN**: Initializes variable `IRName` from the expression on the right-hand side.
  **L235 CN**: 使用右侧表达式初始化变量 `IRName`。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L237 EN**: Comment explains nearby logic, constraints, or intent: `If HasMasked, this is the ID of the LLVM intrinsic we want to lower to.`.
  **L237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`If HasMasked, this is the ID of the LLVM intrinsic we want to lower to.`。
- **L238 EN**: Initializes variable `MaskedIRName` from the expression on the right-hand side.
  **L238 CN**: 使用右侧表达式初始化变量 `MaskedIRName`。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L240 EN**: Comment explains nearby logic, constraints, or intent: `Use clang_builtin_alias to save the number of builtins.`.
  **L240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use clang_builtin_alias to save the number of builtins.`。

### Lines 241-264

````tablegen
  bit HasBuiltinAlias = true;

  // Features required to enable for this builtin.
  list<string> RequiredFeatures = [];

  // Number of fields for Load/Store Segment instructions.
  int NF = 1;

  // Set to true if the builtin is associated with tuple types.
  bit IsTuple = false;

  // Set to true if the builtin has a parameter that models floating-point
  // rounding mode control
  bit HasFRMRoundModeOp = false;

  // TWiden for XSfmm.
  int TWiden = 0;
}

// This is the code emitted in the header.
class RVVHeader {
  code HeaderCode;
}

````
- **L241 EN**: Initializes variable `HasBuiltinAlias` from the expression on the right-hand side.
  **L241 CN**: 使用右侧表达式初始化变量 `HasBuiltinAlias`。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L243 EN**: Comment explains nearby logic, constraints, or intent: `Features required to enable for this builtin.`.
  **L243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Features required to enable for this builtin.`。
- **L244 EN**: Initializes variable `RequiredFeatures` from the expression on the right-hand side.
  **L244 CN**: 使用右侧表达式初始化变量 `RequiredFeatures`。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L246 EN**: Comment explains nearby logic, constraints, or intent: `Number of fields for Load/Store Segment instructions.`.
  **L246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Number of fields for Load/Store Segment instructions.`。
- **L247 EN**: Initializes variable `NF` from the expression on the right-hand side.
  **L247 CN**: 使用右侧表达式初始化变量 `NF`。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L249 EN**: Comment explains nearby logic, constraints, or intent: `Set to true if the builtin is associated with tuple types.`.
  **L249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set to true if the builtin is associated with tuple types.`。
- **L250 EN**: Initializes variable `IsTuple` from the expression on the right-hand side.
  **L250 CN**: 使用右侧表达式初始化变量 `IsTuple`。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L252 EN**: Comment explains nearby logic, constraints, or intent: `Set to true if the builtin has a parameter that models floating-point`.
  **L252 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set to true if the builtin has a parameter that models floating-point`。
- **L253 EN**: Comment explains nearby logic, constraints, or intent: `rounding mode control`.
  **L253 CN**: 注释解释附近代码的逻辑、约束或设计意图：`rounding mode control`。
- **L254 EN**: Initializes variable `HasFRMRoundModeOp` from the expression on the right-hand side.
  **L254 CN**: 使用右侧表达式初始化变量 `HasFRMRoundModeOp`。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L256 EN**: Comment explains nearby logic, constraints, or intent: `TWiden for XSfmm.`.
  **L256 CN**: 注释解释附近代码的逻辑、约束或设计意图：`TWiden for XSfmm.`。
- **L257 EN**: Initializes variable `TWiden` from the expression on the right-hand side.
  **L257 CN**: 使用右侧表达式初始化变量 `TWiden`。
- **L258 EN**: Closes the current lexical scope or compound statement.
  **L258 CN**: 结束当前词法作用域或复合语句块。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L260 EN**: Comment explains nearby logic, constraints, or intent: `This is the code emitted in the header.`.
  **L260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This is the code emitted in the header.`。
- **L261 EN**: Declares TableGen class record `RVVHeader`.
  **L261 CN**: 声明 TableGen class 记录 `RVVHeader`。
- **L262 EN**: Adds a standalone statement or declaration: `code HeaderCode;`.
  **L262 CN**: 添加一条独立语句或声明：`code HeaderCode;`。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 265-288

````tablegen
//===----------------------------------------------------------------------===//
// Basic classes with automatic codegen.
//===----------------------------------------------------------------------===//

class RVVOutBuiltin<string suffix, string prototype, string type_range>
    : RVVBuiltin<suffix, prototype, type_range> {
  let IntrinsicTypes = [-1];
}

class RVVOp0Builtin<string suffix, string prototype, string type_range>
    : RVVBuiltin<suffix, prototype, type_range> {
  let IntrinsicTypes = [0];
}

class RVVOutOp0Builtin<string suffix, string prototype, string type_range>
    : RVVBuiltin<suffix, prototype, type_range> {
  let IntrinsicTypes = [-1, 0];
}

class RVVOutOp1Builtin<string suffix, string prototype, string type_range>
    : RVVBuiltin<suffix, prototype, type_range> {
  let IntrinsicTypes = [-1, 1];
}

````
- **L265 EN**: Banner comment marking a file or section boundary.
  **L265 CN**: 横幅注释，用于标记文件或章节边界。
- **L266 EN**: Comment explains nearby logic, constraints, or intent: `Basic classes with automatic codegen.`.
  **L266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Basic classes with automatic codegen.`。
- **L267 EN**: Banner comment marking a file or section boundary.
  **L267 CN**: 横幅注释，用于标记文件或章节边界。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L269 EN**: Declares TableGen class record `RVVOutBuiltin`.
  **L269 CN**: 声明 TableGen class 记录 `RVVOutBuiltin`。
- **L270 EN**: Continues the surrounding expression or declaration: `: RVVBuiltin<suffix, prototype, type_range> {`.
  **L270 CN**: 继续构造周围的表达式或声明：`: RVVBuiltin<suffix, prototype, type_range> {`。
- **L271 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IntrinsicTypes = [-1];`.
  **L271 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IntrinsicTypes = [-1];`。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Blank line separating nearby declarations or logic blocks.
  **L273 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L274 EN**: Declares TableGen class record `RVVOp0Builtin`.
  **L274 CN**: 声明 TableGen class 记录 `RVVOp0Builtin`。
- **L275 EN**: Continues the surrounding expression or declaration: `: RVVBuiltin<suffix, prototype, type_range> {`.
  **L275 CN**: 继续构造周围的表达式或声明：`: RVVBuiltin<suffix, prototype, type_range> {`。
- **L276 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IntrinsicTypes = [0];`.
  **L276 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IntrinsicTypes = [0];`。
- **L277 EN**: Closes the current lexical scope or compound statement.
  **L277 CN**: 结束当前词法作用域或复合语句块。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L279 EN**: Declares TableGen class record `RVVOutOp0Builtin`.
  **L279 CN**: 声明 TableGen class 记录 `RVVOutOp0Builtin`。
- **L280 EN**: Continues the surrounding expression or declaration: `: RVVBuiltin<suffix, prototype, type_range> {`.
  **L280 CN**: 继续构造周围的表达式或声明：`: RVVBuiltin<suffix, prototype, type_range> {`。
- **L281 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IntrinsicTypes = [-1, 0];`.
  **L281 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IntrinsicTypes = [-1, 0];`。
- **L282 EN**: Closes the current lexical scope or compound statement.
  **L282 CN**: 结束当前词法作用域或复合语句块。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L284 EN**: Declares TableGen class record `RVVOutOp1Builtin`.
  **L284 CN**: 声明 TableGen class 记录 `RVVOutOp1Builtin`。
- **L285 EN**: Continues the surrounding expression or declaration: `: RVVBuiltin<suffix, prototype, type_range> {`.
  **L285 CN**: 继续构造周围的表达式或声明：`: RVVBuiltin<suffix, prototype, type_range> {`。
- **L286 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IntrinsicTypes = [-1, 1];`.
  **L286 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IntrinsicTypes = [-1, 1];`。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 289-312

````tablegen
class RVVOutOp0Op1Builtin<string suffix, string prototype, string type_range>
    : RVVBuiltin<suffix, prototype, type_range> {
  let IntrinsicTypes = [-1, 0, 1];
}

multiclass RVVBuiltinSet<string intrinsic_name, string type_range,
                         list<list<string>> suffixes_prototypes,
                         list<int> intrinsic_types> {
  let IRName = intrinsic_name, MaskedIRName = intrinsic_name # "_mask",
      IntrinsicTypes = intrinsic_types in {
    foreach s_p = suffixes_prototypes in {
      let Name = NAME # "_" # s_p[0] in {
        defvar suffix = s_p[1];
        defvar prototype = s_p[2];
        def : RVVBuiltin<suffix, prototype, type_range>;
      }
    }
  }
}

// IntrinsicTypes is output, op0, op1 [-1, 0, 1]
multiclass RVVOutOp0Op1BuiltinSet<string intrinsic_name, string type_range,
                                  list<list<string>> suffixes_prototypes>
    : RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes,
````
- **L289 EN**: Declares TableGen class record `RVVOutOp0Op1Builtin`.
  **L289 CN**: 声明 TableGen class 记录 `RVVOutOp0Op1Builtin`。
- **L290 EN**: Continues the surrounding expression or declaration: `: RVVBuiltin<suffix, prototype, type_range> {`.
  **L290 CN**: 继续构造周围的表达式或声明：`: RVVBuiltin<suffix, prototype, type_range> {`。
- **L291 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IntrinsicTypes = [-1, 0, 1];`.
  **L291 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IntrinsicTypes = [-1, 0, 1];`。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Blank line separating nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L294 EN**: Declares TableGen multiclass record `RVVBuiltinSet`.
  **L294 CN**: 声明 TableGen multiclass 记录 `RVVBuiltinSet`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<list<string>> suffixes_prototypes,`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<list<string>> suffixes_prototypes,`。
- **L296 EN**: Continues the surrounding expression or declaration: `list<int> intrinsic_types> {`.
  **L296 CN**: 继续构造周围的表达式或声明：`list<int> intrinsic_types> {`。
- **L297 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IRName = intrinsic_name, MaskedIRName = intrinsic_name # "_mask",`.
  **L297 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IRName = intrinsic_name, MaskedIRName = intrinsic_name # "_mask",`。
- **L298 EN**: Continues the surrounding expression or declaration: `IntrinsicTypes = intrinsic_types in {`.
  **L298 CN**: 继续构造周围的表达式或声明：`IntrinsicTypes = intrinsic_types in {`。
- **L299 EN**: Starts a TableGen iteration used to generate repeated records: `foreach s_p = suffixes_prototypes in {`.
  **L299 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach s_p = suffixes_prototypes in {`。
- **L300 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_" # s_p[0] in {`.
  **L300 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_" # s_p[0] in {`。
- **L301 EN**: Declares TableGen defvar record `suffix = s_p[1];`.
  **L301 CN**: 声明 TableGen defvar 记录 `suffix = s_p[1];`。
- **L302 EN**: Declares TableGen defvar record `prototype = s_p[2];`.
  **L302 CN**: 声明 TableGen defvar 记录 `prototype = s_p[2];`。
- **L303 EN**: Declares TableGen def record `def`.
  **L303 CN**: 声明 TableGen def 记录 `def`。
- **L304 EN**: Closes the current lexical scope or compound statement.
  **L304 CN**: 结束当前词法作用域或复合语句块。
- **L305 EN**: Closes the current lexical scope or compound statement.
  **L305 CN**: 结束当前词法作用域或复合语句块。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Closes the current lexical scope or compound statement.
  **L307 CN**: 结束当前词法作用域或复合语句块。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L309 EN**: Comment explains nearby logic, constraints, or intent: `IntrinsicTypes is output, op0, op1 [-1, 0, 1]`.
  **L309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IntrinsicTypes is output, op0, op1 [-1, 0, 1]`。
- **L310 EN**: Declares TableGen multiclass record `RVVOutOp0Op1BuiltinSet`.
  **L310 CN**: 声明 TableGen multiclass 记录 `RVVOutOp0Op1BuiltinSet`。
- **L311 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes>`.
  **L311 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes>`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes,`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes,`。

### Lines 313-336

````tablegen
                            [-1, 0, 1]>;

multiclass RVVOutBuiltinSet<string intrinsic_name, string type_range,
                            list<list<string>> suffixes_prototypes>
    : RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1]>;

multiclass RVVOp0BuiltinSet<string intrinsic_name, string type_range,
                            list<list<string>> suffixes_prototypes>
    : RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [0]>;

// IntrinsicTypes is output, op1 [-1, 0]
multiclass RVVOutOp0BuiltinSet<string intrinsic_name, string type_range,
                               list<list<string>> suffixes_prototypes>
    : RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 0]>;

// IntrinsicTypes is output, op1 [-1, 1]
multiclass RVVOutOp1BuiltinSet<string intrinsic_name, string type_range,
                               list<list<string>> suffixes_prototypes>
    : RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 1]>;

multiclass RVVOp0Op1BuiltinSet<string intrinsic_name, string type_range,
                               list<list<string>> suffixes_prototypes>
    : RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [0, 1]>;

````
- **L313 EN**: Adds a standalone statement or declaration: `[-1, 0, 1]>;`.
  **L313 CN**: 添加一条独立语句或声明：`[-1, 0, 1]>;`。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Declares TableGen multiclass record `RVVOutBuiltinSet`.
  **L315 CN**: 声明 TableGen multiclass 记录 `RVVOutBuiltinSet`。
- **L316 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes>`.
  **L316 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes>`。
- **L317 EN**: Adds a standalone statement or declaration: `: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1]>;`.
  **L317 CN**: 添加一条独立语句或声明：`: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1]>;`。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L319 EN**: Declares TableGen multiclass record `RVVOp0BuiltinSet`.
  **L319 CN**: 声明 TableGen multiclass 记录 `RVVOp0BuiltinSet`。
- **L320 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes>`.
  **L320 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes>`。
- **L321 EN**: Adds a standalone statement or declaration: `: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [0]>;`.
  **L321 CN**: 添加一条独立语句或声明：`: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [0]>;`。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Comment explains nearby logic, constraints, or intent: `IntrinsicTypes is output, op1 [-1, 0]`.
  **L323 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IntrinsicTypes is output, op1 [-1, 0]`。
- **L324 EN**: Declares TableGen multiclass record `RVVOutOp0BuiltinSet`.
  **L324 CN**: 声明 TableGen multiclass 记录 `RVVOutOp0BuiltinSet`。
- **L325 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes>`.
  **L325 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes>`。
- **L326 EN**: Adds a standalone statement or declaration: `: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 0]>;`.
  **L326 CN**: 添加一条独立语句或声明：`: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 0]>;`。
- **L327 EN**: Blank line separating nearby declarations or logic blocks.
  **L327 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L328 EN**: Comment explains nearby logic, constraints, or intent: `IntrinsicTypes is output, op1 [-1, 1]`.
  **L328 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IntrinsicTypes is output, op1 [-1, 1]`。
- **L329 EN**: Declares TableGen multiclass record `RVVOutOp1BuiltinSet`.
  **L329 CN**: 声明 TableGen multiclass 记录 `RVVOutOp1BuiltinSet`。
- **L330 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes>`.
  **L330 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes>`。
- **L331 EN**: Adds a standalone statement or declaration: `: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 1]>;`.
  **L331 CN**: 添加一条独立语句或声明：`: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 1]>;`。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Declares TableGen multiclass record `RVVOp0Op1BuiltinSet`.
  **L333 CN**: 声明 TableGen multiclass 记录 `RVVOp0Op1BuiltinSet`。
- **L334 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes>`.
  **L334 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes>`。
- **L335 EN**: Adds a standalone statement or declaration: `: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [0, 1]>;`.
  **L335 CN**: 添加一条独立语句或声明：`: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [0, 1]>;`。
- **L336 EN**: Blank line separating nearby declarations or logic blocks.
  **L336 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 337-360

````tablegen
multiclass RVVOutOp1Op2BuiltinSet<string intrinsic_name, string type_range,
                                  list<list<string>> suffixes_prototypes>
    : RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 1, 2]>;

// IntrinsicTypes is output, op2 [-1, 2]
multiclass RVVOutOp2BuiltinSet<string intrinsic_name, string type_range,
                               list<list<string>> suffixes_prototypes>
    : RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 2]>;

multiclass RVVSignedBinBuiltinSet
    : RVVOutOp1BuiltinSet<NAME, "csil",
                          [["vv", "v", "vvv"],
                           ["vx", "v", "vve"]]>;

multiclass RVVSignedBinBuiltinSetRoundingMode
    : RVVOutOp1BuiltinSet<NAME, "csil",
                          [["vv", "v", "vvvu"],
                           ["vx", "v", "vveu"]]>;

multiclass RVVUnsignedBinBuiltinSet
    : RVVOutOp1BuiltinSet<NAME, "csil",
                          [["vv", "Uv", "UvUvUv"],
                           ["vx", "Uv", "UvUvUe"]]>;

````
- **L337 EN**: Declares TableGen multiclass record `RVVOutOp1Op2BuiltinSet`.
  **L337 CN**: 声明 TableGen multiclass 记录 `RVVOutOp1Op2BuiltinSet`。
- **L338 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes>`.
  **L338 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes>`。
- **L339 EN**: Adds a standalone statement or declaration: `: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 1, 2]>;`.
  **L339 CN**: 添加一条独立语句或声明：`: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 1, 2]>;`。
- **L340 EN**: Blank line separating nearby declarations or logic blocks.
  **L340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L341 EN**: Comment explains nearby logic, constraints, or intent: `IntrinsicTypes is output, op2 [-1, 2]`.
  **L341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IntrinsicTypes is output, op2 [-1, 2]`。
- **L342 EN**: Declares TableGen multiclass record `RVVOutOp2BuiltinSet`.
  **L342 CN**: 声明 TableGen multiclass 记录 `RVVOutOp2BuiltinSet`。
- **L343 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes>`.
  **L343 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes>`。
- **L344 EN**: Adds a standalone statement or declaration: `: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 2]>;`.
  **L344 CN**: 添加一条独立语句或声明：`: RVVBuiltinSet<intrinsic_name, type_range, suffixes_prototypes, [-1, 2]>;`。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L346 EN**: Declares TableGen multiclass record `RVVSignedBinBuiltinSet`.
  **L346 CN**: 声明 TableGen multiclass 记录 `RVVSignedBinBuiltinSet`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvv"],`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvv"],`。
- **L349 EN**: Adds a standalone statement or declaration: `["vx", "v", "vve"]]>;`.
  **L349 CN**: 添加一条独立语句或声明：`["vx", "v", "vve"]]>;`。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L351 EN**: Declares TableGen multiclass record `RVVSignedBinBuiltinSetRoundingMode`.
  **L351 CN**: 声明 TableGen multiclass 记录 `RVVSignedBinBuiltinSetRoundingMode`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvu"],`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvu"],`。
- **L354 EN**: Adds a standalone statement or declaration: `["vx", "v", "vveu"]]>;`.
  **L354 CN**: 添加一条独立语句或声明：`["vx", "v", "vveu"]]>;`。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L356 EN**: Declares TableGen multiclass record `RVVUnsignedBinBuiltinSet`.
  **L356 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedBinBuiltinSet`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uv", "UvUvUv"],`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uv", "UvUvUv"],`。
- **L359 EN**: Adds a standalone statement or declaration: `["vx", "Uv", "UvUvUe"]]>;`.
  **L359 CN**: 添加一条独立语句或声明：`["vx", "Uv", "UvUvUe"]]>;`。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 361-384

````tablegen
multiclass RVVUnsignedBinBuiltinSetRoundingMode
    : RVVOutOp1BuiltinSet<NAME, "csil",
                          [["vv", "Uv", "UvUvUvu"],
                           ["vx", "Uv", "UvUvUeu"]]>;

multiclass RVVIntBinBuiltinSet
    : RVVSignedBinBuiltinSet,
      RVVUnsignedBinBuiltinSet;

multiclass RVVInt64BinBuiltinSet
    : RVVOutOp1BuiltinSet<NAME, "l",
                          [["vv", "v", "vvv"],
                           ["vx", "v", "vve"]]>,
      RVVOutOp1BuiltinSet<NAME, "l",
                          [["vv", "Uv", "UvUvUv"],
                           ["vx", "Uv", "UvUvUe"]]>;

multiclass RVVSlideOneBuiltinSet
    : RVVOutOp1BuiltinSet<NAME, "csil",
                          [["vx", "v", "vve"],
                           ["vx", "Uv", "UvUvUe"]]>;

multiclass RVVSignedShiftBuiltinSet
    : RVVOutOp1BuiltinSet<NAME, "csil",
````
- **L361 EN**: Declares TableGen multiclass record `RVVUnsignedBinBuiltinSetRoundingMode`.
  **L361 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedBinBuiltinSetRoundingMode`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uv", "UvUvUvu"],`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uv", "UvUvUvu"],`。
- **L364 EN**: Adds a standalone statement or declaration: `["vx", "Uv", "UvUvUeu"]]>;`.
  **L364 CN**: 添加一条独立语句或声明：`["vx", "Uv", "UvUvUeu"]]>;`。
- **L365 EN**: Blank line separating nearby declarations or logic blocks.
  **L365 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L366 EN**: Declares TableGen multiclass record `RVVIntBinBuiltinSet`.
  **L366 CN**: 声明 TableGen multiclass 记录 `RVVIntBinBuiltinSet`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVSignedBinBuiltinSet,`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVSignedBinBuiltinSet,`。
- **L368 EN**: Adds a standalone statement or declaration: `RVVUnsignedBinBuiltinSet;`.
  **L368 CN**: 添加一条独立语句或声明：`RVVUnsignedBinBuiltinSet;`。
- **L369 EN**: Blank line separating nearby declarations or logic blocks.
  **L369 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L370 EN**: Declares TableGen multiclass record `RVVInt64BinBuiltinSet`.
  **L370 CN**: 声明 TableGen multiclass 记录 `RVVInt64BinBuiltinSet`。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "l",`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "l",`。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvv"],`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvv"],`。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["vx", "v", "vve"]]>,`.
  **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`["vx", "v", "vve"]]>,`。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RVVOutOp1BuiltinSet<NAME, "l",`.
  **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`RVVOutOp1BuiltinSet<NAME, "l",`。
- **L375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uv", "UvUvUv"],`.
  **L375 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uv", "UvUvUv"],`。
- **L376 EN**: Adds a standalone statement or declaration: `["vx", "Uv", "UvUvUe"]]>;`.
  **L376 CN**: 添加一条独立语句或声明：`["vx", "Uv", "UvUvUe"]]>;`。
- **L377 EN**: Blank line separating nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L378 EN**: Declares TableGen multiclass record `RVVSlideOneBuiltinSet`.
  **L378 CN**: 声明 TableGen multiclass 记录 `RVVSlideOneBuiltinSet`。
- **L379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L379 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。
- **L380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vx", "v", "vve"],`.
  **L380 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vx", "v", "vve"],`。
- **L381 EN**: Adds a standalone statement or declaration: `["vx", "Uv", "UvUvUe"]]>;`.
  **L381 CN**: 添加一条独立语句或声明：`["vx", "Uv", "UvUvUe"]]>;`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L383 EN**: Declares TableGen multiclass record `RVVSignedShiftBuiltinSet`.
  **L383 CN**: 声明 TableGen multiclass 记录 `RVVSignedShiftBuiltinSet`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。

### Lines 385-408

````tablegen
                          [["vv", "v", "vvUv"],
                           ["vx", "v", "vvz"]]>;

multiclass RVVSignedShiftBuiltinSetRoundingMode
    : RVVOutOp1BuiltinSet<NAME, "csil",
                          [["vv", "v", "vvUvu"],
                           ["vx", "v", "vvzu"]]>;

multiclass RVVUnsignedShiftBuiltinSet
    : RVVOutOp1BuiltinSet<NAME, "csil",
                          [["vv", "Uv", "UvUvUv"],
                           ["vx", "Uv", "UvUvz"]]>;

multiclass RVVUnsignedShiftBuiltinSetRoundingMode
    : RVVOutOp1BuiltinSet<NAME, "csil",
                          [["vv", "Uv", "UvUvUvu"],
                           ["vx", "Uv", "UvUvzu"]]>;

multiclass RVVShiftBuiltinSet
    : RVVSignedShiftBuiltinSet,
      RVVUnsignedShiftBuiltinSet;

let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
  multiclass RVVSignedNShiftBuiltinSet
````
- **L385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvUv"],`.
  **L385 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvUv"],`。
- **L386 EN**: Adds a standalone statement or declaration: `["vx", "v", "vvz"]]>;`.
  **L386 CN**: 添加一条独立语句或声明：`["vx", "v", "vvz"]]>;`。
- **L387 EN**: Blank line separating nearby declarations or logic blocks.
  **L387 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L388 EN**: Declares TableGen multiclass record `RVVSignedShiftBuiltinSetRoundingMode`.
  **L388 CN**: 声明 TableGen multiclass 记录 `RVVSignedShiftBuiltinSetRoundingMode`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvUvu"],`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvUvu"],`。
- **L391 EN**: Adds a standalone statement or declaration: `["vx", "v", "vvzu"]]>;`.
  **L391 CN**: 添加一条独立语句或声明：`["vx", "v", "vvzu"]]>;`。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L393 EN**: Declares TableGen multiclass record `RVVUnsignedShiftBuiltinSet`.
  **L393 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedShiftBuiltinSet`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uv", "UvUvUv"],`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uv", "UvUvUv"],`。
- **L396 EN**: Adds a standalone statement or declaration: `["vx", "Uv", "UvUvz"]]>;`.
  **L396 CN**: 添加一条独立语句或声明：`["vx", "Uv", "UvUvz"]]>;`。
- **L397 EN**: Blank line separating nearby declarations or logic blocks.
  **L397 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L398 EN**: Declares TableGen multiclass record `RVVUnsignedShiftBuiltinSetRoundingMode`.
  **L398 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedShiftBuiltinSetRoundingMode`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uv", "UvUvUvu"],`.
  **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uv", "UvUvUvu"],`。
- **L401 EN**: Adds a standalone statement or declaration: `["vx", "Uv", "UvUvzu"]]>;`.
  **L401 CN**: 添加一条独立语句或声明：`["vx", "Uv", "UvUvzu"]]>;`。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L403 EN**: Declares TableGen multiclass record `RVVShiftBuiltinSet`.
  **L403 CN**: 声明 TableGen multiclass 记录 `RVVShiftBuiltinSet`。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVSignedShiftBuiltinSet,`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVSignedShiftBuiltinSet,`。
- **L405 EN**: Adds a standalone statement or declaration: `RVVUnsignedShiftBuiltinSet;`.
  **L405 CN**: 添加一条独立语句或声明：`RVVUnsignedShiftBuiltinSet;`。
- **L406 EN**: Blank line separating nearby declarations or logic blocks.
  **L406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L407 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L407 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。
- **L408 EN**: Declares TableGen multiclass record `RVVSignedNShiftBuiltinSet`.
  **L408 CN**: 声明 TableGen multiclass 记录 `RVVSignedNShiftBuiltinSet`。

### Lines 409-432

````tablegen
      : RVVOutOp0Op1BuiltinSet<NAME, "csil",
                                     [["wv", "v", "vwUv"],
                                      ["wx", "v", "vwz"]]>;

  multiclass RVVSignedNShiftBuiltinSetRoundingMode
      : RVVOutOp0Op1BuiltinSet<NAME, "csil",
                                     [["wv", "v", "vwUvu"],
                                      ["wx", "v", "vwzu"]]>;

  multiclass RVVUnsignedNShiftBuiltinSet
      : RVVOutOp0Op1BuiltinSet<NAME, "csil",
                                     [["wv", "Uv", "UvUwUv"],
                                      ["wx", "Uv", "UvUwz"]]>;

  multiclass RVVUnsignedNShiftBuiltinSetRoundingMode
      : RVVOutOp0Op1BuiltinSet<NAME, "csil",
                                     [["wv", "Uv", "UvUwUvu"],
                                      ["wx", "Uv", "UvUwzu"]]>;

}

multiclass RVVCarryinBuiltinSet
    : RVVOutOp1BuiltinSet<NAME, "csil",
                          [["vvm", "v", "vvvm"],
````
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp0Op1BuiltinSet<NAME, "csil",`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp0Op1BuiltinSet<NAME, "csil",`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "v", "vwUv"],`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "v", "vwUv"],`。
- **L411 EN**: Adds a standalone statement or declaration: `["wx", "v", "vwz"]]>;`.
  **L411 CN**: 添加一条独立语句或声明：`["wx", "v", "vwz"]]>;`。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L413 EN**: Declares TableGen multiclass record `RVVSignedNShiftBuiltinSetRoundingMode`.
  **L413 CN**: 声明 TableGen multiclass 记录 `RVVSignedNShiftBuiltinSetRoundingMode`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp0Op1BuiltinSet<NAME, "csil",`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp0Op1BuiltinSet<NAME, "csil",`。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "v", "vwUvu"],`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "v", "vwUvu"],`。
- **L416 EN**: Adds a standalone statement or declaration: `["wx", "v", "vwzu"]]>;`.
  **L416 CN**: 添加一条独立语句或声明：`["wx", "v", "vwzu"]]>;`。
- **L417 EN**: Blank line separating nearby declarations or logic blocks.
  **L417 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L418 EN**: Declares TableGen multiclass record `RVVUnsignedNShiftBuiltinSet`.
  **L418 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedNShiftBuiltinSet`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp0Op1BuiltinSet<NAME, "csil",`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp0Op1BuiltinSet<NAME, "csil",`。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "Uv", "UvUwUv"],`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "Uv", "UvUwUv"],`。
- **L421 EN**: Adds a standalone statement or declaration: `["wx", "Uv", "UvUwz"]]>;`.
  **L421 CN**: 添加一条独立语句或声明：`["wx", "Uv", "UvUwz"]]>;`。
- **L422 EN**: Blank line separating nearby declarations or logic blocks.
  **L422 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L423 EN**: Declares TableGen multiclass record `RVVUnsignedNShiftBuiltinSetRoundingMode`.
  **L423 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedNShiftBuiltinSetRoundingMode`。
- **L424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp0Op1BuiltinSet<NAME, "csil",`.
  **L424 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp0Op1BuiltinSet<NAME, "csil",`。
- **L425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "Uv", "UvUwUvu"],`.
  **L425 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "Uv", "UvUwUvu"],`。
- **L426 EN**: Adds a standalone statement or declaration: `["wx", "Uv", "UvUwzu"]]>;`.
  **L426 CN**: 添加一条独立语句或声明：`["wx", "Uv", "UvUwzu"]]>;`。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L428 EN**: Closes the current lexical scope or compound statement.
  **L428 CN**: 结束当前词法作用域或复合语句块。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L430 EN**: Declares TableGen multiclass record `RVVCarryinBuiltinSet`.
  **L430 CN**: 声明 TableGen multiclass 记录 `RVVCarryinBuiltinSet`。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOutOp1BuiltinSet<NAME, "csil",`.
  **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOutOp1BuiltinSet<NAME, "csil",`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vvm", "v", "vvvm"],`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vvm", "v", "vvvm"],`。

### Lines 433-456

````tablegen
                           ["vxm", "v", "vvem"],
                           ["vvm", "Uv", "UvUvUvm"],
                           ["vxm", "Uv", "UvUvUem"]]>;

multiclass RVVCarryOutInBuiltinSet<string intrinsic_name>
    : RVVOp0Op1BuiltinSet<intrinsic_name, "csil",
                          [["vvm", "vm", "mvvm"],
                           ["vxm", "vm", "mvem"],
                           ["vvm", "Uvm", "mUvUvm"],
                           ["vxm", "Uvm", "mUvUem"]]>;

multiclass RVVSignedMaskOutBuiltinSet
    : RVVOp0Op1BuiltinSet<NAME, "csil",
                          [["vv", "vm", "mvv"],
                           ["vx", "vm", "mve"]]>;

multiclass RVVUnsignedMaskOutBuiltinSet
    : RVVOp0Op1BuiltinSet<NAME, "csil",
                          [["vv", "Uvm", "mUvUv"],
                           ["vx", "Uvm", "mUvUe"]]>;

multiclass RVVIntMaskOutBuiltinSet
    : RVVSignedMaskOutBuiltinSet,
      RVVUnsignedMaskOutBuiltinSet;
````
- **L433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["vxm", "v", "vvem"],`.
  **L433 CN**: 继续一个多行参数列表、初始化器或聚合项：`["vxm", "v", "vvem"],`。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["vvm", "Uv", "UvUvUvm"],`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`["vvm", "Uv", "UvUvUvm"],`。
- **L435 EN**: Adds a standalone statement or declaration: `["vxm", "Uv", "UvUvUem"]]>;`.
  **L435 CN**: 添加一条独立语句或声明：`["vxm", "Uv", "UvUvUem"]]>;`。
- **L436 EN**: Blank line separating nearby declarations or logic blocks.
  **L436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L437 EN**: Declares TableGen multiclass record `RVVCarryOutInBuiltinSet`.
  **L437 CN**: 声明 TableGen multiclass 记录 `RVVCarryOutInBuiltinSet`。
- **L438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOp0Op1BuiltinSet<intrinsic_name, "csil",`.
  **L438 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOp0Op1BuiltinSet<intrinsic_name, "csil",`。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vvm", "vm", "mvvm"],`.
  **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vvm", "vm", "mvvm"],`。
- **L440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["vxm", "vm", "mvem"],`.
  **L440 CN**: 继续一个多行参数列表、初始化器或聚合项：`["vxm", "vm", "mvem"],`。
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["vvm", "Uvm", "mUvUvm"],`.
  **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`["vvm", "Uvm", "mUvUvm"],`。
- **L442 EN**: Adds a standalone statement or declaration: `["vxm", "Uvm", "mUvUem"]]>;`.
  **L442 CN**: 添加一条独立语句或声明：`["vxm", "Uvm", "mUvUem"]]>;`。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L444 EN**: Declares TableGen multiclass record `RVVSignedMaskOutBuiltinSet`.
  **L444 CN**: 声明 TableGen multiclass 记录 `RVVSignedMaskOutBuiltinSet`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOp0Op1BuiltinSet<NAME, "csil",`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOp0Op1BuiltinSet<NAME, "csil",`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vm", "mvv"],`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vm", "mvv"],`。
- **L447 EN**: Adds a standalone statement or declaration: `["vx", "vm", "mve"]]>;`.
  **L447 CN**: 添加一条独立语句或声明：`["vx", "vm", "mve"]]>;`。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L449 EN**: Declares TableGen multiclass record `RVVUnsignedMaskOutBuiltinSet`.
  **L449 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedMaskOutBuiltinSet`。
- **L450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOp0Op1BuiltinSet<NAME, "csil",`.
  **L450 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOp0Op1BuiltinSet<NAME, "csil",`。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uvm", "mUvUv"],`.
  **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uvm", "mUvUv"],`。
- **L452 EN**: Adds a standalone statement or declaration: `["vx", "Uvm", "mUvUe"]]>;`.
  **L452 CN**: 添加一条独立语句或声明：`["vx", "Uvm", "mUvUe"]]>;`。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L454 EN**: Declares TableGen multiclass record `RVVIntMaskOutBuiltinSet`.
  **L454 CN**: 声明 TableGen multiclass 记录 `RVVIntMaskOutBuiltinSet`。
- **L455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVSignedMaskOutBuiltinSet,`.
  **L455 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVSignedMaskOutBuiltinSet,`。
- **L456 EN**: Adds a standalone statement or declaration: `RVVUnsignedMaskOutBuiltinSet;`.
  **L456 CN**: 添加一条独立语句或声明：`RVVUnsignedMaskOutBuiltinSet;`。

### Lines 457-480

````tablegen

class RVVIntExt<string intrinsic_name, string suffix, string prototype,
                string type_range>
    : RVVBuiltin<suffix, prototype, type_range> {
  let IRName = intrinsic_name;
  let MaskedIRName = intrinsic_name # "_mask";
  let OverloadedName = NAME;
  let IntrinsicTypes = [-1, 0];
}

let HasMaskedOffOperand = false in {
  multiclass RVVIntTerBuiltinSet {
    defm "" : RVVOutOp1BuiltinSet<NAME, "csil",
                                  [["vv", "v", "vvvv"],
                                   ["vx", "v", "vvev"],
                                   ["vv", "Uv", "UvUvUvUv"],
                                   ["vx", "Uv", "UvUvUeUv"]]>;
  }
  multiclass RVVFloatingTerBuiltinSet {
    defm "" : RVVOutOp1BuiltinSet<NAME, "fd",
                                  [["vv", "v", "vvvv"],
                                   ["vf", "v", "vvev"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm "" : RVVOutOp1BuiltinSet<NAME, "x",
````
- **L457 EN**: Blank line separating nearby declarations or logic blocks.
  **L457 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L458 EN**: Declares TableGen class record `RVVIntExt`.
  **L458 CN**: 声明 TableGen class 记录 `RVVIntExt`。
- **L459 EN**: Continues the surrounding expression or declaration: `string type_range>`.
  **L459 CN**: 继续构造周围的表达式或声明：`string type_range>`。
- **L460 EN**: Continues the surrounding expression or declaration: `: RVVBuiltin<suffix, prototype, type_range> {`.
  **L460 CN**: 继续构造周围的表达式或声明：`: RVVBuiltin<suffix, prototype, type_range> {`。
- **L461 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IRName = intrinsic_name;`.
  **L461 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IRName = intrinsic_name;`。
- **L462 EN**: Assigns a TableGen property that affects following records or inherited fields: `let MaskedIRName = intrinsic_name # "_mask";`.
  **L462 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let MaskedIRName = intrinsic_name # "_mask";`。
- **L463 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = NAME;`.
  **L463 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = NAME;`。
- **L464 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IntrinsicTypes = [-1, 0];`.
  **L464 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IntrinsicTypes = [-1, 0];`。
- **L465 EN**: Closes the current lexical scope or compound statement.
  **L465 CN**: 结束当前词法作用域或复合语句块。
- **L466 EN**: Blank line separating nearby declarations or logic blocks.
  **L466 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L467 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMaskedOffOperand = false in {`.
  **L467 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMaskedOffOperand = false in {`。
- **L468 EN**: Declares TableGen multiclass record `RVVIntTerBuiltinSet`.
  **L468 CN**: 声明 TableGen multiclass 记录 `RVVIntTerBuiltinSet`。
- **L469 EN**: Declares TableGen defm record `""`.
  **L469 CN**: 声明 TableGen defm 记录 `""`。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvv"],`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvv"],`。
- **L471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["vx", "v", "vvev"],`.
  **L471 CN**: 继续一个多行参数列表、初始化器或聚合项：`["vx", "v", "vvev"],`。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["vv", "Uv", "UvUvUvUv"],`.
  **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`["vv", "Uv", "UvUvUvUv"],`。
- **L473 EN**: Adds a standalone statement or declaration: `["vx", "Uv", "UvUvUeUv"]]>;`.
  **L473 CN**: 添加一条独立语句或声明：`["vx", "Uv", "UvUvUeUv"]]>;`。
- **L474 EN**: Closes the current lexical scope or compound statement.
  **L474 CN**: 结束当前词法作用域或复合语句块。
- **L475 EN**: Declares TableGen multiclass record `RVVFloatingTerBuiltinSet`.
  **L475 CN**: 声明 TableGen multiclass 记录 `RVVFloatingTerBuiltinSet`。
- **L476 EN**: Declares TableGen defm record `""`.
  **L476 CN**: 声明 TableGen defm 记录 `""`。
- **L477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvv"],`.
  **L477 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvv"],`。
- **L478 EN**: Adds a standalone statement or declaration: `["vf", "v", "vvev"]]>;`.
  **L478 CN**: 添加一条独立语句或声明：`["vf", "v", "vvev"]]>;`。
- **L479 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L479 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L480 EN**: Declares TableGen defm record `""`.
  **L480 CN**: 声明 TableGen defm 记录 `""`。

### Lines 481-504

````tablegen
                                    [["vv", "v", "vvvv"],
                                     ["vf", "v", "vvev"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm "" : RVVOutOp1BuiltinSet<NAME, "y",
                                    [["vv", "v", "vvvv"],
                                     ["vf", "v", "vvev"]]>;
  }
  multiclass RVVFloatingTerBuiltinSetRoundingMode {
    defm "" : RVVOutOp1BuiltinSet<NAME, "fd",
                                  [["vv", "v", "vvvvu"],
                                   ["vf", "v", "vvevu"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm "" : RVVOutOp1BuiltinSet<NAME, "x",
                                    [["vv", "v", "vvvvu"],
                                     ["vf", "v", "vvevu"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm "" : RVVOutOp1BuiltinSet<NAME, "y",
                                    [["vv", "v", "vvvvu"],
                                     ["vf", "v", "vvevu"]]>;
  }
}

let HasMaskedOffOperand = false, Log2LMUL = [-2, -1, 0, 1, 2] in {
  multiclass RVVFloatingWidenTerBuiltinSet {
````
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvv"],`.
  **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvv"],`。
- **L482 EN**: Adds a standalone statement or declaration: `["vf", "v", "vvev"]]>;`.
  **L482 CN**: 添加一条独立语句或声明：`["vf", "v", "vvev"]]>;`。
- **L483 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L483 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L484 EN**: Declares TableGen defm record `""`.
  **L484 CN**: 声明 TableGen defm 记录 `""`。
- **L485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvv"],`.
  **L485 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvv"],`。
- **L486 EN**: Adds a standalone statement or declaration: `["vf", "v", "vvev"]]>;`.
  **L486 CN**: 添加一条独立语句或声明：`["vf", "v", "vvev"]]>;`。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Declares TableGen multiclass record `RVVFloatingTerBuiltinSetRoundingMode`.
  **L488 CN**: 声明 TableGen multiclass 记录 `RVVFloatingTerBuiltinSetRoundingMode`。
- **L489 EN**: Declares TableGen defm record `""`.
  **L489 CN**: 声明 TableGen defm 记录 `""`。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvvu"],`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvvu"],`。
- **L491 EN**: Adds a standalone statement or declaration: `["vf", "v", "vvevu"]]>;`.
  **L491 CN**: 添加一条独立语句或声明：`["vf", "v", "vvevu"]]>;`。
- **L492 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L492 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L493 EN**: Declares TableGen defm record `""`.
  **L493 CN**: 声明 TableGen defm 记录 `""`。
- **L494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvvu"],`.
  **L494 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvvu"],`。
- **L495 EN**: Adds a standalone statement or declaration: `["vf", "v", "vvevu"]]>;`.
  **L495 CN**: 添加一条独立语句或声明：`["vf", "v", "vvevu"]]>;`。
- **L496 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L496 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L497 EN**: Declares TableGen defm record `""`.
  **L497 CN**: 声明 TableGen defm 记录 `""`。
- **L498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvvu"],`.
  **L498 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvvu"],`。
- **L499 EN**: Adds a standalone statement or declaration: `["vf", "v", "vvevu"]]>;`.
  **L499 CN**: 添加一条独立语句或声明：`["vf", "v", "vvevu"]]>;`。
- **L500 EN**: Closes the current lexical scope or compound statement.
  **L500 CN**: 结束当前词法作用域或复合语句块。
- **L501 EN**: Closes the current lexical scope or compound statement.
  **L501 CN**: 结束当前词法作用域或复合语句块。
- **L502 EN**: Blank line separating nearby declarations or logic blocks.
  **L502 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L503 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMaskedOffOperand = false, Log2LMUL = [-2, -1, 0, 1, 2] in {`.
  **L503 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMaskedOffOperand = false, Log2LMUL = [-2, -1, 0, 1, 2] in {`。
- **L504 EN**: Declares TableGen multiclass record `RVVFloatingWidenTerBuiltinSet`.
  **L504 CN**: 声明 TableGen multiclass 记录 `RVVFloatingWidenTerBuiltinSet`。

### Lines 505-528

````tablegen
    defm ""  : RVVOutOp1Op2BuiltinSet<NAME, "f",
                                      [["vv", "w", "wwvv"],
                                       ["vf", "w", "wwev"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm ""  : RVVOutOp1Op2BuiltinSet<NAME, "x",
                                        [["vv", "w", "wwvv"],
                                         ["vf", "w", "wwev"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm ""  : RVVOutOp1Op2BuiltinSet<NAME, "y",
                                        [["vv", "vw", "wwvv"],
                                         ["vf", "vw", "wwev"]]>;
  }
  multiclass RVVFloatingWidenTerBuiltinSetRoundingMode {
    defm ""  : RVVOutOp1Op2BuiltinSet<NAME, "f",
                                      [["vv", "w", "wwvvu"],
                                       ["vf", "w", "wwevu"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm ""  : RVVOutOp1Op2BuiltinSet<NAME, "x",
                                        [["vv", "w", "wwvvu"],
                                         ["vf", "w", "wwevu"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm ""  : RVVOutOp1Op2BuiltinSet<NAME, "y",
                                        [["vv", "vw", "wwvvu"],
                                         ["vf", "vw", "wwevu"]]>;
````
- **L505 EN**: Declares TableGen defm record `""`.
  **L505 CN**: 声明 TableGen defm 记录 `""`。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wwvv"],`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wwvv"],`。
- **L507 EN**: Adds a standalone statement or declaration: `["vf", "w", "wwev"]]>;`.
  **L507 CN**: 添加一条独立语句或声明：`["vf", "w", "wwev"]]>;`。
- **L508 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L508 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L509 EN**: Declares TableGen defm record `""`.
  **L509 CN**: 声明 TableGen defm 记录 `""`。
- **L510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wwvv"],`.
  **L510 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wwvv"],`。
- **L511 EN**: Adds a standalone statement or declaration: `["vf", "w", "wwev"]]>;`.
  **L511 CN**: 添加一条独立语句或声明：`["vf", "w", "wwev"]]>;`。
- **L512 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L512 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L513 EN**: Declares TableGen defm record `""`.
  **L513 CN**: 声明 TableGen defm 记录 `""`。
- **L514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vw", "wwvv"],`.
  **L514 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vw", "wwvv"],`。
- **L515 EN**: Adds a standalone statement or declaration: `["vf", "vw", "wwev"]]>;`.
  **L515 CN**: 添加一条独立语句或声明：`["vf", "vw", "wwev"]]>;`。
- **L516 EN**: Closes the current lexical scope or compound statement.
  **L516 CN**: 结束当前词法作用域或复合语句块。
- **L517 EN**: Declares TableGen multiclass record `RVVFloatingWidenTerBuiltinSetRoundingMode`.
  **L517 CN**: 声明 TableGen multiclass 记录 `RVVFloatingWidenTerBuiltinSetRoundingMode`。
- **L518 EN**: Declares TableGen defm record `""`.
  **L518 CN**: 声明 TableGen defm 记录 `""`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wwvvu"],`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wwvvu"],`。
- **L520 EN**: Adds a standalone statement or declaration: `["vf", "w", "wwevu"]]>;`.
  **L520 CN**: 添加一条独立语句或声明：`["vf", "w", "wwevu"]]>;`。
- **L521 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L521 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L522 EN**: Declares TableGen defm record `""`.
  **L522 CN**: 声明 TableGen defm 记录 `""`。
- **L523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wwvvu"],`.
  **L523 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wwvvu"],`。
- **L524 EN**: Adds a standalone statement or declaration: `["vf", "w", "wwevu"]]>;`.
  **L524 CN**: 添加一条独立语句或声明：`["vf", "w", "wwevu"]]>;`。
- **L525 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L525 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L526 EN**: Declares TableGen defm record `""`.
  **L526 CN**: 声明 TableGen defm 记录 `""`。
- **L527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vw", "wwvvu"],`.
  **L527 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vw", "wwvvu"],`。
- **L528 EN**: Adds a standalone statement or declaration: `["vf", "vw", "wwevu"]]>;`.
  **L528 CN**: 添加一条独立语句或声明：`["vf", "vw", "wwevu"]]>;`。

### Lines 529-552

````tablegen
  }
}

multiclass RVVFloatingBinBuiltinSet<bit HasBF = 0> {
  defm "" : RVVOutOp1BuiltinSet<NAME, "fd",
                                [["vv", "v", "vvv"],
                                 ["vf", "v", "vve"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm "" : RVVOutOp1BuiltinSet<NAME, "x",
                                  [["vv", "v", "vvv"],
                                   ["vf", "v", "vve"]]>;
  if HasBF then {
    let RequiredFeatures = ["zvfbfa"] in
      defm "" : RVVOutOp1BuiltinSet<NAME, "y",
                                    [["vv", "v", "vvv"],
                                     ["vf", "v", "vve"]]>;
  }
}

multiclass RVVFloatingBinBuiltinSetRoundingMode<bit HasBF = 0> {
  defm "" : RVVOutOp1BuiltinSet<NAME, "fd",
                                [["vv", "v", "vvvu"],
                                 ["vf", "v", "vveu"]]>;
  let RequiredFeatures = ["zvfh"] in
````
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Closes the current lexical scope or compound statement.
  **L530 CN**: 结束当前词法作用域或复合语句块。
- **L531 EN**: Blank line separating nearby declarations or logic blocks.
  **L531 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L532 EN**: Declares TableGen multiclass record `RVVFloatingBinBuiltinSet`.
  **L532 CN**: 声明 TableGen multiclass 记录 `RVVFloatingBinBuiltinSet`。
- **L533 EN**: Declares TableGen defm record `""`.
  **L533 CN**: 声明 TableGen defm 记录 `""`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvv"],`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvv"],`。
- **L535 EN**: Adds a standalone statement or declaration: `["vf", "v", "vve"]]>;`.
  **L535 CN**: 添加一条独立语句或声明：`["vf", "v", "vve"]]>;`。
- **L536 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L536 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L537 EN**: Declares TableGen defm record `""`.
  **L537 CN**: 声明 TableGen defm 记录 `""`。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvv"],`.
  **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvv"],`。
- **L539 EN**: Adds a standalone statement or declaration: `["vf", "v", "vve"]]>;`.
  **L539 CN**: 添加一条独立语句或声明：`["vf", "v", "vve"]]>;`。
- **L540 EN**: Continues the surrounding expression or declaration: `if HasBF then {`.
  **L540 CN**: 继续构造周围的表达式或声明：`if HasBF then {`。
- **L541 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L541 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L542 EN**: Declares TableGen defm record `""`.
  **L542 CN**: 声明 TableGen defm 记录 `""`。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvv"],`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvv"],`。
- **L544 EN**: Adds a standalone statement or declaration: `["vf", "v", "vve"]]>;`.
  **L544 CN**: 添加一条独立语句或声明：`["vf", "v", "vve"]]>;`。
- **L545 EN**: Closes the current lexical scope or compound statement.
  **L545 CN**: 结束当前词法作用域或复合语句块。
- **L546 EN**: Closes the current lexical scope or compound statement.
  **L546 CN**: 结束当前词法作用域或复合语句块。
- **L547 EN**: Blank line separating nearby declarations or logic blocks.
  **L547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L548 EN**: Declares TableGen multiclass record `RVVFloatingBinBuiltinSetRoundingMode`.
  **L548 CN**: 声明 TableGen multiclass 记录 `RVVFloatingBinBuiltinSetRoundingMode`。
- **L549 EN**: Declares TableGen defm record `""`.
  **L549 CN**: 声明 TableGen defm 记录 `""`。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvu"],`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvu"],`。
- **L551 EN**: Adds a standalone statement or declaration: `["vf", "v", "vveu"]]>;`.
  **L551 CN**: 添加一条独立语句或声明：`["vf", "v", "vveu"]]>;`。
- **L552 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L552 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。

### Lines 553-576

````tablegen
    defm "" : RVVOutOp1BuiltinSet<NAME, "x",
                                  [["vv", "v", "vvvu"],
                                   ["vf", "v", "vveu"]]>;
  if HasBF then {
    let RequiredFeatures = ["zvfbfa"] in
      defm "" : RVVOutOp1BuiltinSet<NAME, "y",
                                    [["vv", "v", "vvvu"],
                                     ["vf", "v", "vveu"]]>;
  }
}

multiclass RVVFloatingBinVFBuiltinSet<bit HasBF = 0> {
  defm "" : RVVOutOp1BuiltinSet<NAME, "fd",
                                [["vf", "v", "vve"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm "" : RVVOutOp1BuiltinSet<NAME, "x",
                                  [["vf", "v", "vve"]]>;
  if HasBF then {
    let RequiredFeatures = ["zvfbfa"] in
      defm "" : RVVOutOp1BuiltinSet<NAME, "y",
                                    [["vf", "v", "vve"]]>;
  }
}

````
- **L553 EN**: Declares TableGen defm record `""`.
  **L553 CN**: 声明 TableGen defm 记录 `""`。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvu"],`.
  **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvu"],`。
- **L555 EN**: Adds a standalone statement or declaration: `["vf", "v", "vveu"]]>;`.
  **L555 CN**: 添加一条独立语句或声明：`["vf", "v", "vveu"]]>;`。
- **L556 EN**: Continues the surrounding expression or declaration: `if HasBF then {`.
  **L556 CN**: 继续构造周围的表达式或声明：`if HasBF then {`。
- **L557 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L557 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L558 EN**: Declares TableGen defm record `""`.
  **L558 CN**: 声明 TableGen defm 记录 `""`。
- **L559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvvu"],`.
  **L559 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvvu"],`。
- **L560 EN**: Adds a standalone statement or declaration: `["vf", "v", "vveu"]]>;`.
  **L560 CN**: 添加一条独立语句或声明：`["vf", "v", "vveu"]]>;`。
- **L561 EN**: Closes the current lexical scope or compound statement.
  **L561 CN**: 结束当前词法作用域或复合语句块。
- **L562 EN**: Closes the current lexical scope or compound statement.
  **L562 CN**: 结束当前词法作用域或复合语句块。
- **L563 EN**: Blank line separating nearby declarations or logic blocks.
  **L563 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L564 EN**: Declares TableGen multiclass record `RVVFloatingBinVFBuiltinSet`.
  **L564 CN**: 声明 TableGen multiclass 记录 `RVVFloatingBinVFBuiltinSet`。
- **L565 EN**: Declares TableGen defm record `""`.
  **L565 CN**: 声明 TableGen defm 记录 `""`。
- **L566 EN**: Adds a standalone statement or declaration: `[["vf", "v", "vve"]]>;`.
  **L566 CN**: 添加一条独立语句或声明：`[["vf", "v", "vve"]]>;`。
- **L567 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L567 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L568 EN**: Declares TableGen defm record `""`.
  **L568 CN**: 声明 TableGen defm 记录 `""`。
- **L569 EN**: Adds a standalone statement or declaration: `[["vf", "v", "vve"]]>;`.
  **L569 CN**: 添加一条独立语句或声明：`[["vf", "v", "vve"]]>;`。
- **L570 EN**: Continues the surrounding expression or declaration: `if HasBF then {`.
  **L570 CN**: 继续构造周围的表达式或声明：`if HasBF then {`。
- **L571 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L571 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L572 EN**: Declares TableGen defm record `""`.
  **L572 CN**: 声明 TableGen defm 记录 `""`。
- **L573 EN**: Adds a standalone statement or declaration: `[["vf", "v", "vve"]]>;`.
  **L573 CN**: 添加一条独立语句或声明：`[["vf", "v", "vve"]]>;`。
- **L574 EN**: Closes the current lexical scope or compound statement.
  **L574 CN**: 结束当前词法作用域或复合语句块。
- **L575 EN**: Closes the current lexical scope or compound statement.
  **L575 CN**: 结束当前词法作用域或复合语句块。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 577-600

````tablegen
multiclass RVVFloatingBinVFBuiltinSetRoundingMode<bit HasBF = 0> {
  defm "" : RVVOutOp1BuiltinSet<NAME, "fd",
                                [["vf", "v", "vveu"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm "" : RVVOutOp1BuiltinSet<NAME, "x",
                                  [["vf", "v", "vveu"]]>;
  if HasBF then {
    let RequiredFeatures = ["zvfbfa"] in
      defm "" : RVVOutOp1BuiltinSet<NAME, "y",
                                    [["vf", "v", "vveu"]]>;
  }
}

multiclass RVVFloatingMaskOutBuiltinSet {
  defm "" : RVVOp0Op1BuiltinSet<NAME, "fd",
                                [["vv", "vm", "mvv"],
                                 ["vf", "vm", "mve"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm "" : RVVOp0Op1BuiltinSet<NAME, "x",
                                  [["vv", "vm", "mvv"],
                                   ["vf", "vm", "mve"]]>;
  let RequiredFeatures = ["zvfbfa"] in
    defm "" : RVVOp0Op1BuiltinSet<NAME, "y",
                                  [["vv", "vm", "mvv"],
````
- **L577 EN**: Declares TableGen multiclass record `RVVFloatingBinVFBuiltinSetRoundingMode`.
  **L577 CN**: 声明 TableGen multiclass 记录 `RVVFloatingBinVFBuiltinSetRoundingMode`。
- **L578 EN**: Declares TableGen defm record `""`.
  **L578 CN**: 声明 TableGen defm 记录 `""`。
- **L579 EN**: Adds a standalone statement or declaration: `[["vf", "v", "vveu"]]>;`.
  **L579 CN**: 添加一条独立语句或声明：`[["vf", "v", "vveu"]]>;`。
- **L580 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L580 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L581 EN**: Declares TableGen defm record `""`.
  **L581 CN**: 声明 TableGen defm 记录 `""`。
- **L582 EN**: Adds a standalone statement or declaration: `[["vf", "v", "vveu"]]>;`.
  **L582 CN**: 添加一条独立语句或声明：`[["vf", "v", "vveu"]]>;`。
- **L583 EN**: Continues the surrounding expression or declaration: `if HasBF then {`.
  **L583 CN**: 继续构造周围的表达式或声明：`if HasBF then {`。
- **L584 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L584 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L585 EN**: Declares TableGen defm record `""`.
  **L585 CN**: 声明 TableGen defm 记录 `""`。
- **L586 EN**: Adds a standalone statement or declaration: `[["vf", "v", "vveu"]]>;`.
  **L586 CN**: 添加一条独立语句或声明：`[["vf", "v", "vveu"]]>;`。
- **L587 EN**: Closes the current lexical scope or compound statement.
  **L587 CN**: 结束当前词法作用域或复合语句块。
- **L588 EN**: Closes the current lexical scope or compound statement.
  **L588 CN**: 结束当前词法作用域或复合语句块。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L590 EN**: Declares TableGen multiclass record `RVVFloatingMaskOutBuiltinSet`.
  **L590 CN**: 声明 TableGen multiclass 记录 `RVVFloatingMaskOutBuiltinSet`。
- **L591 EN**: Declares TableGen defm record `""`.
  **L591 CN**: 声明 TableGen defm 记录 `""`。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vm", "mvv"],`.
  **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vm", "mvv"],`。
- **L593 EN**: Adds a standalone statement or declaration: `["vf", "vm", "mve"]]>;`.
  **L593 CN**: 添加一条独立语句或声明：`["vf", "vm", "mve"]]>;`。
- **L594 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L594 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L595 EN**: Declares TableGen defm record `""`.
  **L595 CN**: 声明 TableGen defm 记录 `""`。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vm", "mvv"],`.
  **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vm", "mvv"],`。
- **L597 EN**: Adds a standalone statement or declaration: `["vf", "vm", "mve"]]>;`.
  **L597 CN**: 添加一条独立语句或声明：`["vf", "vm", "mve"]]>;`。
- **L598 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L598 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L599 EN**: Declares TableGen defm record `""`.
  **L599 CN**: 声明 TableGen defm 记录 `""`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vm", "mvv"],`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vm", "mvv"],`。

### Lines 601-624

````tablegen
                                   ["vf", "vm", "mve"]]>;
}

multiclass RVVFloatingMaskOutVFBuiltinSet
    : RVVOp0Op1BuiltinSet<NAME, "fd",
                          [["vf", "vm", "mve"]]>;

multiclass RVVConvBuiltinSet<string intrinsic_name, string type_range,
                         list<list<string>> suffixes_prototypes> {
let Name = intrinsic_name,
    IRName = intrinsic_name,
    MaskedIRName = intrinsic_name # "_mask",
    IntrinsicTypes = [-1, 0] in {
  foreach s_p = suffixes_prototypes in {
      defvar suffix = s_p[0];
      defvar prototype = s_p[1];
      def : RVVBuiltin<suffix, prototype, type_range>;
    }
  }
}


class RVVMaskBinBuiltin : RVVOutBuiltin<"m", "mmm", "c"> {
  let Name = NAME # "_mm";
````
- **L601 EN**: Adds a standalone statement or declaration: `["vf", "vm", "mve"]]>;`.
  **L601 CN**: 添加一条独立语句或声明：`["vf", "vm", "mve"]]>;`。
- **L602 EN**: Closes the current lexical scope or compound statement.
  **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L604 EN**: Declares TableGen multiclass record `RVVFloatingMaskOutVFBuiltinSet`.
  **L604 CN**: 声明 TableGen multiclass 记录 `RVVFloatingMaskOutVFBuiltinSet`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVOp0Op1BuiltinSet<NAME, "fd",`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVOp0Op1BuiltinSet<NAME, "fd",`。
- **L606 EN**: Adds a standalone statement or declaration: `[["vf", "vm", "mve"]]>;`.
  **L606 CN**: 添加一条独立语句或声明：`[["vf", "vm", "mve"]]>;`。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L608 EN**: Declares TableGen multiclass record `RVVConvBuiltinSet`.
  **L608 CN**: 声明 TableGen multiclass 记录 `RVVConvBuiltinSet`。
- **L609 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes> {`.
  **L609 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes> {`。
- **L610 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = intrinsic_name,`.
  **L610 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = intrinsic_name,`。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = intrinsic_name,`.
  **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = intrinsic_name,`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = intrinsic_name # "_mask",`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = intrinsic_name # "_mask",`。
- **L613 EN**: Continues the surrounding expression or declaration: `IntrinsicTypes = [-1, 0] in {`.
  **L613 CN**: 继续构造周围的表达式或声明：`IntrinsicTypes = [-1, 0] in {`。
- **L614 EN**: Starts a TableGen iteration used to generate repeated records: `foreach s_p = suffixes_prototypes in {`.
  **L614 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach s_p = suffixes_prototypes in {`。
- **L615 EN**: Declares TableGen defvar record `suffix = s_p[0];`.
  **L615 CN**: 声明 TableGen defvar 记录 `suffix = s_p[0];`。
- **L616 EN**: Declares TableGen defvar record `prototype = s_p[1];`.
  **L616 CN**: 声明 TableGen defvar 记录 `prototype = s_p[1];`。
- **L617 EN**: Declares TableGen def record `def`.
  **L617 CN**: 声明 TableGen def 记录 `def`。
- **L618 EN**: Closes the current lexical scope or compound statement.
  **L618 CN**: 结束当前词法作用域或复合语句块。
- **L619 EN**: Closes the current lexical scope or compound statement.
  **L619 CN**: 结束当前词法作用域或复合语句块。
- **L620 EN**: Closes the current lexical scope or compound statement.
  **L620 CN**: 结束当前词法作用域或复合语句块。
- **L621 EN**: Blank line separating nearby declarations or logic blocks.
  **L621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L623 EN**: Declares TableGen class record `RVVMaskBinBuiltin`.
  **L623 CN**: 声明 TableGen class 记录 `RVVMaskBinBuiltin`。
- **L624 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_mm";`.
  **L624 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_mm";`。

### Lines 625-648

````tablegen
  let HasMasked = false;
}

class RVVMaskUnaryBuiltin : RVVOutBuiltin<"m", "mm", "c"> {
  let Name = NAME # "_m";
}

class RVVMaskNullaryBuiltin : RVVOutBuiltin<"m", "m", "c"> {
  let Name = NAME # "_m";
  let HasMasked = false;
  let SupportOverloading = false;
}

class RVVMaskOp0Builtin<string prototype> : RVVOp0Builtin<"m", prototype, "c"> {
  let Name = NAME # "_m";
  let HasMaskedOffOperand = false;
}

let UnMaskedPolicyScheme = HasPolicyOperand,
    HasMaskedOffOperand = false in {
  multiclass RVVSlideUpBuiltinSet {
    defm "" : RVVOutBuiltinSet<NAME, "csilxfdy",
                               [["vx","v", "vvvz"]]>;
    defm "" : RVVOutBuiltinSet<NAME, "csil",
````
- **L625 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false;`.
  **L625 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false;`。
- **L626 EN**: Closes the current lexical scope or compound statement.
  **L626 CN**: 结束当前词法作用域或复合语句块。
- **L627 EN**: Blank line separating nearby declarations or logic blocks.
  **L627 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L628 EN**: Declares TableGen class record `RVVMaskUnaryBuiltin`.
  **L628 CN**: 声明 TableGen class 记录 `RVVMaskUnaryBuiltin`。
- **L629 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_m";`.
  **L629 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_m";`。
- **L630 EN**: Closes the current lexical scope or compound statement.
  **L630 CN**: 结束当前词法作用域或复合语句块。
- **L631 EN**: Blank line separating nearby declarations or logic blocks.
  **L631 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L632 EN**: Declares TableGen class record `RVVMaskNullaryBuiltin`.
  **L632 CN**: 声明 TableGen class 记录 `RVVMaskNullaryBuiltin`。
- **L633 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_m";`.
  **L633 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_m";`。
- **L634 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false;`.
  **L634 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false;`。
- **L635 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SupportOverloading = false;`.
  **L635 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SupportOverloading = false;`。
- **L636 EN**: Closes the current lexical scope or compound statement.
  **L636 CN**: 结束当前词法作用域或复合语句块。
- **L637 EN**: Blank line separating nearby declarations or logic blocks.
  **L637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L638 EN**: Declares TableGen class record `RVVMaskOp0Builtin`.
  **L638 CN**: 声明 TableGen class 记录 `RVVMaskOp0Builtin`。
- **L639 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_m";`.
  **L639 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_m";`。
- **L640 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMaskedOffOperand = false;`.
  **L640 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMaskedOffOperand = false;`。
- **L641 EN**: Closes the current lexical scope or compound statement.
  **L641 CN**: 结束当前词法作用域或复合语句块。
- **L642 EN**: Blank line separating nearby declarations or logic blocks.
  **L642 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L643 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPolicyOperand,`.
  **L643 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPolicyOperand,`。
- **L644 EN**: Continues the surrounding expression or declaration: `HasMaskedOffOperand = false in {`.
  **L644 CN**: 继续构造周围的表达式或声明：`HasMaskedOffOperand = false in {`。
- **L645 EN**: Declares TableGen multiclass record `RVVSlideUpBuiltinSet`.
  **L645 CN**: 声明 TableGen multiclass 记录 `RVVSlideUpBuiltinSet`。
- **L646 EN**: Declares TableGen defm record `""`.
  **L646 CN**: 声明 TableGen defm 记录 `""`。
- **L647 EN**: Adds a standalone statement or declaration: `[["vx","v", "vvvz"]]>;`.
  **L647 CN**: 添加一条独立语句或声明：`[["vx","v", "vvvz"]]>;`。
- **L648 EN**: Declares TableGen defm record `""`.
  **L648 CN**: 声明 TableGen defm 记录 `""`。

### Lines 649-672

````tablegen
                               [["vx","Uv", "UvUvUvz"]]>;
  }
}

let UnMaskedPolicyScheme = HasPassthruOperand,
    ManualCodegen = [{
      if (IsMasked) {
        std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end() - 1);
        if ((PolicyAttrs & RVV_VTA) && (PolicyAttrs & RVV_VMA))
          Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
      } else {
        if (PolicyAttrs & RVV_VTA)
          Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
      }

      Ops.push_back(ConstantInt::get(Ops.back()->getType(), PolicyAttrs));
      IntrinsicTypes = {ResultType, Ops.back()->getType()};
    }] in {
  multiclass RVVSlideDownBuiltinSet {
    defm "" : RVVOutBuiltinSet<NAME, "csilxfdy",
                               [["vx","v", "vvz"]]>;
    defm "" : RVVOutBuiltinSet<NAME, "csil",
                               [["vx","Uv", "UvUvz"]]>;
  }
````
- **L649 EN**: Adds a standalone statement or declaration: `[["vx","Uv", "UvUvUvz"]]>;`.
  **L649 CN**: 添加一条独立语句或声明：`[["vx","Uv", "UvUvUvz"]]>;`。
- **L650 EN**: Closes the current lexical scope or compound statement.
  **L650 CN**: 结束当前词法作用域或复合语句块。
- **L651 EN**: Closes the current lexical scope or compound statement.
  **L651 CN**: 结束当前词法作用域或复合语句块。
- **L652 EN**: Blank line separating nearby declarations or logic blocks.
  **L652 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L653 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L653 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L654 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L654 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L655 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L655 CN**: 开始 `if` 控制流语句并计算其条件。
- **L656 EN**: Executes a call or declaration centered on `std::rotate`.
  **L656 CN**: 执行以 `std::rotate` 为核心的调用或声明。
- **L657 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L657 CN**: 开始 `if` 控制流语句并计算其条件。
- **L658 EN**: Executes a call or declaration centered on `Ops.insert`.
  **L658 CN**: 执行以 `Ops.insert` 为核心的调用或声明。
- **L659 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L659 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L660 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L660 CN**: 开始 `if` 控制流语句并计算其条件。
- **L661 EN**: Executes a call or declaration centered on `Ops.insert`.
  **L661 CN**: 执行以 `Ops.insert` 为核心的调用或声明。
- **L662 EN**: Closes the current lexical scope or compound statement.
  **L662 CN**: 结束当前词法作用域或复合语句块。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L664 EN**: Executes a call or declaration centered on `Ops.push_back`.
  **L664 CN**: 执行以 `Ops.push_back` 为核心的调用或声明。
- **L665 EN**: Executes a call or declaration centered on `Ops.back`.
  **L665 CN**: 执行以 `Ops.back` 为核心的调用或声明。
- **L666 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L666 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L667 EN**: Declares TableGen multiclass record `RVVSlideDownBuiltinSet`.
  **L667 CN**: 声明 TableGen multiclass 记录 `RVVSlideDownBuiltinSet`。
- **L668 EN**: Declares TableGen defm record `""`.
  **L668 CN**: 声明 TableGen defm 记录 `""`。
- **L669 EN**: Adds a standalone statement or declaration: `[["vx","v", "vvz"]]>;`.
  **L669 CN**: 添加一条独立语句或声明：`[["vx","v", "vvz"]]>;`。
- **L670 EN**: Declares TableGen defm record `""`.
  **L670 CN**: 声明 TableGen defm 记录 `""`。
- **L671 EN**: Adds a standalone statement or declaration: `[["vx","Uv", "UvUvz"]]>;`.
  **L671 CN**: 添加一条独立语句或声明：`[["vx","Uv", "UvUvz"]]>;`。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-696

````tablegen
}

class RVVConvBuiltin<string suffix, string prototype, string type_range,
                     string overloaded_name>
    : RVVBuiltin<suffix, prototype, type_range> {
  let IntrinsicTypes = [-1, 0];
  let OverloadedName = overloaded_name;
}

class RVVConvToSignedBuiltin<string overloaded_name>
    : RVVConvBuiltin<"Iv", "Ivv", "xfd", overloaded_name>;

class RVVConvToUnsignedBuiltin<string overloaded_name>
    : RVVConvBuiltin<"Uv", "Uvv", "xfd", overloaded_name>;

class RVVConvToWidenSignedBuiltin<string overloaded_name>
    : RVVConvBuiltin<"Iw", "Iwv", "xf", overloaded_name>;

class RVVConvToWidenUnsignedBuiltin<string overloaded_name>
    : RVVConvBuiltin<"Uw", "Uwv", "xf", overloaded_name>;

class RVVConvToNarrowingSignedBuiltin<string overloaded_name>
    : RVVConvBuiltin<"Iv", "IvFw", "si", overloaded_name>;

````
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L675 EN**: Declares TableGen class record `RVVConvBuiltin`.
  **L675 CN**: 声明 TableGen class 记录 `RVVConvBuiltin`。
- **L676 EN**: Continues the surrounding expression or declaration: `string overloaded_name>`.
  **L676 CN**: 继续构造周围的表达式或声明：`string overloaded_name>`。
- **L677 EN**: Continues the surrounding expression or declaration: `: RVVBuiltin<suffix, prototype, type_range> {`.
  **L677 CN**: 继续构造周围的表达式或声明：`: RVVBuiltin<suffix, prototype, type_range> {`。
- **L678 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IntrinsicTypes = [-1, 0];`.
  **L678 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IntrinsicTypes = [-1, 0];`。
- **L679 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = overloaded_name;`.
  **L679 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = overloaded_name;`。
- **L680 EN**: Closes the current lexical scope or compound statement.
  **L680 CN**: 结束当前词法作用域或复合语句块。
- **L681 EN**: Blank line separating nearby declarations or logic blocks.
  **L681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L682 EN**: Declares TableGen class record `RVVConvToSignedBuiltin`.
  **L682 CN**: 声明 TableGen class 记录 `RVVConvToSignedBuiltin`。
- **L683 EN**: Adds a standalone statement or declaration: `: RVVConvBuiltin<"Iv", "Ivv", "xfd", overloaded_name>;`.
  **L683 CN**: 添加一条独立语句或声明：`: RVVConvBuiltin<"Iv", "Ivv", "xfd", overloaded_name>;`。
- **L684 EN**: Blank line separating nearby declarations or logic blocks.
  **L684 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L685 EN**: Declares TableGen class record `RVVConvToUnsignedBuiltin`.
  **L685 CN**: 声明 TableGen class 记录 `RVVConvToUnsignedBuiltin`。
- **L686 EN**: Adds a standalone statement or declaration: `: RVVConvBuiltin<"Uv", "Uvv", "xfd", overloaded_name>;`.
  **L686 CN**: 添加一条独立语句或声明：`: RVVConvBuiltin<"Uv", "Uvv", "xfd", overloaded_name>;`。
- **L687 EN**: Blank line separating nearby declarations or logic blocks.
  **L687 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L688 EN**: Declares TableGen class record `RVVConvToWidenSignedBuiltin`.
  **L688 CN**: 声明 TableGen class 记录 `RVVConvToWidenSignedBuiltin`。
- **L689 EN**: Adds a standalone statement or declaration: `: RVVConvBuiltin<"Iw", "Iwv", "xf", overloaded_name>;`.
  **L689 CN**: 添加一条独立语句或声明：`: RVVConvBuiltin<"Iw", "Iwv", "xf", overloaded_name>;`。
- **L690 EN**: Blank line separating nearby declarations or logic blocks.
  **L690 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L691 EN**: Declares TableGen class record `RVVConvToWidenUnsignedBuiltin`.
  **L691 CN**: 声明 TableGen class 记录 `RVVConvToWidenUnsignedBuiltin`。
- **L692 EN**: Adds a standalone statement or declaration: `: RVVConvBuiltin<"Uw", "Uwv", "xf", overloaded_name>;`.
  **L692 CN**: 添加一条独立语句或声明：`: RVVConvBuiltin<"Uw", "Uwv", "xf", overloaded_name>;`。
- **L693 EN**: Blank line separating nearby declarations or logic blocks.
  **L693 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L694 EN**: Declares TableGen class record `RVVConvToNarrowingSignedBuiltin`.
  **L694 CN**: 声明 TableGen class 记录 `RVVConvToNarrowingSignedBuiltin`。
- **L695 EN**: Adds a standalone statement or declaration: `: RVVConvBuiltin<"Iv", "IvFw", "si", overloaded_name>;`.
  **L695 CN**: 添加一条独立语句或声明：`: RVVConvBuiltin<"Iv", "IvFw", "si", overloaded_name>;`。
- **L696 EN**: Blank line separating nearby declarations or logic blocks.
  **L696 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 697-720

````tablegen
class RVVConvToNarrowingUnsignedBuiltin<string overloaded_name>
    : RVVConvBuiltin<"Uv", "UvFw", "si", overloaded_name>;

let HasMaskedOffOperand = true in {
  multiclass RVVSignedReductionBuiltin {
    defm "" : RVVOutOp0BuiltinSet<NAME, "csil",
                                  [["vs", "vSv", "SvvSv"]]>;
  }
  multiclass RVVUnsignedReductionBuiltin {
    defm "" : RVVOutOp0BuiltinSet<NAME, "csil",
                                  [["vs", "UvUSv", "USvUvUSv"]]>;
  }
  multiclass RVVFloatingReductionBuiltin {
    defm "" : RVVOutOp0BuiltinSet<NAME, "fd",
                                  [["vs", "vSv", "SvvSv"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm "" : RVVOutOp0BuiltinSet<NAME, "x",
                                    [["vs", "vSv", "SvvSv"]]>;
  }
  multiclass RVVFloatingReductionBuiltinRoundingMode {
    defm "" : RVVOutOp0BuiltinSet<NAME, "fd",
                                  [["vs", "vSv", "SvvSvu"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm "" : RVVOutOp0BuiltinSet<NAME, "x",
````
- **L697 EN**: Declares TableGen class record `RVVConvToNarrowingUnsignedBuiltin`.
  **L697 CN**: 声明 TableGen class 记录 `RVVConvToNarrowingUnsignedBuiltin`。
- **L698 EN**: Adds a standalone statement or declaration: `: RVVConvBuiltin<"Uv", "UvFw", "si", overloaded_name>;`.
  **L698 CN**: 添加一条独立语句或声明：`: RVVConvBuiltin<"Uv", "UvFw", "si", overloaded_name>;`。
- **L699 EN**: Blank line separating nearby declarations or logic blocks.
  **L699 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L700 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMaskedOffOperand = true in {`.
  **L700 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMaskedOffOperand = true in {`。
- **L701 EN**: Declares TableGen multiclass record `RVVSignedReductionBuiltin`.
  **L701 CN**: 声明 TableGen multiclass 记录 `RVVSignedReductionBuiltin`。
- **L702 EN**: Declares TableGen defm record `""`.
  **L702 CN**: 声明 TableGen defm 记录 `""`。
- **L703 EN**: Adds a standalone statement or declaration: `[["vs", "vSv", "SvvSv"]]>;`.
  **L703 CN**: 添加一条独立语句或声明：`[["vs", "vSv", "SvvSv"]]>;`。
- **L704 EN**: Closes the current lexical scope or compound statement.
  **L704 CN**: 结束当前词法作用域或复合语句块。
- **L705 EN**: Declares TableGen multiclass record `RVVUnsignedReductionBuiltin`.
  **L705 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedReductionBuiltin`。
- **L706 EN**: Declares TableGen defm record `""`.
  **L706 CN**: 声明 TableGen defm 记录 `""`。
- **L707 EN**: Adds a standalone statement or declaration: `[["vs", "UvUSv", "USvUvUSv"]]>;`.
  **L707 CN**: 添加一条独立语句或声明：`[["vs", "UvUSv", "USvUvUSv"]]>;`。
- **L708 EN**: Closes the current lexical scope or compound statement.
  **L708 CN**: 结束当前词法作用域或复合语句块。
- **L709 EN**: Declares TableGen multiclass record `RVVFloatingReductionBuiltin`.
  **L709 CN**: 声明 TableGen multiclass 记录 `RVVFloatingReductionBuiltin`。
- **L710 EN**: Declares TableGen defm record `""`.
  **L710 CN**: 声明 TableGen defm 记录 `""`。
- **L711 EN**: Adds a standalone statement or declaration: `[["vs", "vSv", "SvvSv"]]>;`.
  **L711 CN**: 添加一条独立语句或声明：`[["vs", "vSv", "SvvSv"]]>;`。
- **L712 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L712 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L713 EN**: Declares TableGen defm record `""`.
  **L713 CN**: 声明 TableGen defm 记录 `""`。
- **L714 EN**: Adds a standalone statement or declaration: `[["vs", "vSv", "SvvSv"]]>;`.
  **L714 CN**: 添加一条独立语句或声明：`[["vs", "vSv", "SvvSv"]]>;`。
- **L715 EN**: Closes the current lexical scope or compound statement.
  **L715 CN**: 结束当前词法作用域或复合语句块。
- **L716 EN**: Declares TableGen multiclass record `RVVFloatingReductionBuiltinRoundingMode`.
  **L716 CN**: 声明 TableGen multiclass 记录 `RVVFloatingReductionBuiltinRoundingMode`。
- **L717 EN**: Declares TableGen defm record `""`.
  **L717 CN**: 声明 TableGen defm 记录 `""`。
- **L718 EN**: Adds a standalone statement or declaration: `[["vs", "vSv", "SvvSvu"]]>;`.
  **L718 CN**: 添加一条独立语句或声明：`[["vs", "vSv", "SvvSvu"]]>;`。
- **L719 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L719 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L720 EN**: Declares TableGen defm record `""`.
  **L720 CN**: 声明 TableGen defm 记录 `""`。

### Lines 721-744

````tablegen
                                    [["vs", "vSv", "SvvSvu"]]>;
  }
  multiclass RVVFloatingWidenReductionBuiltin {
    defm "" : RVVOutOp0BuiltinSet<NAME, "f",
                                  [["vs", "vSw", "SwvSw"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm "" : RVVOutOp0BuiltinSet<NAME, "x",
                                    [["vs", "vSw", "SwvSw"]]>;
  }
  multiclass RVVFloatingWidenReductionBuiltinRoundingMode {
    defm "" : RVVOutOp0BuiltinSet<NAME, "f",
                                  [["vs", "vSw", "SwvSwu"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm "" : RVVOutOp0BuiltinSet<NAME, "x",
                                    [["vs", "vSw", "SwvSwu"]]>;
  }
}

multiclass RVVIntReductionBuiltinSet
    : RVVSignedReductionBuiltin,
      RVVUnsignedReductionBuiltin;

// For widen operation which has different mangling name.
multiclass RVVWidenBuiltinSet<string intrinsic_name, string type_range,
````
- **L721 EN**: Adds a standalone statement or declaration: `[["vs", "vSv", "SvvSvu"]]>;`.
  **L721 CN**: 添加一条独立语句或声明：`[["vs", "vSv", "SvvSvu"]]>;`。
- **L722 EN**: Closes the current lexical scope or compound statement.
  **L722 CN**: 结束当前词法作用域或复合语句块。
- **L723 EN**: Declares TableGen multiclass record `RVVFloatingWidenReductionBuiltin`.
  **L723 CN**: 声明 TableGen multiclass 记录 `RVVFloatingWidenReductionBuiltin`。
- **L724 EN**: Declares TableGen defm record `""`.
  **L724 CN**: 声明 TableGen defm 记录 `""`。
- **L725 EN**: Adds a standalone statement or declaration: `[["vs", "vSw", "SwvSw"]]>;`.
  **L725 CN**: 添加一条独立语句或声明：`[["vs", "vSw", "SwvSw"]]>;`。
- **L726 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L726 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L727 EN**: Declares TableGen defm record `""`.
  **L727 CN**: 声明 TableGen defm 记录 `""`。
- **L728 EN**: Adds a standalone statement or declaration: `[["vs", "vSw", "SwvSw"]]>;`.
  **L728 CN**: 添加一条独立语句或声明：`[["vs", "vSw", "SwvSw"]]>;`。
- **L729 EN**: Closes the current lexical scope or compound statement.
  **L729 CN**: 结束当前词法作用域或复合语句块。
- **L730 EN**: Declares TableGen multiclass record `RVVFloatingWidenReductionBuiltinRoundingMode`.
  **L730 CN**: 声明 TableGen multiclass 记录 `RVVFloatingWidenReductionBuiltinRoundingMode`。
- **L731 EN**: Declares TableGen defm record `""`.
  **L731 CN**: 声明 TableGen defm 记录 `""`。
- **L732 EN**: Adds a standalone statement or declaration: `[["vs", "vSw", "SwvSwu"]]>;`.
  **L732 CN**: 添加一条独立语句或声明：`[["vs", "vSw", "SwvSwu"]]>;`。
- **L733 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L733 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L734 EN**: Declares TableGen defm record `""`.
  **L734 CN**: 声明 TableGen defm 记录 `""`。
- **L735 EN**: Adds a standalone statement or declaration: `[["vs", "vSw", "SwvSwu"]]>;`.
  **L735 CN**: 添加一条独立语句或声明：`[["vs", "vSw", "SwvSwu"]]>;`。
- **L736 EN**: Closes the current lexical scope or compound statement.
  **L736 CN**: 结束当前词法作用域或复合语句块。
- **L737 EN**: Closes the current lexical scope or compound statement.
  **L737 CN**: 结束当前词法作用域或复合语句块。
- **L738 EN**: Blank line separating nearby declarations or logic blocks.
  **L738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L739 EN**: Declares TableGen multiclass record `RVVIntReductionBuiltinSet`.
  **L739 CN**: 声明 TableGen multiclass 记录 `RVVIntReductionBuiltinSet`。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVSignedReductionBuiltin,`.
  **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVSignedReductionBuiltin,`。
- **L741 EN**: Adds a standalone statement or declaration: `RVVUnsignedReductionBuiltin;`.
  **L741 CN**: 添加一条独立语句或声明：`RVVUnsignedReductionBuiltin;`。
- **L742 EN**: Blank line separating nearby declarations or logic blocks.
  **L742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L743 EN**: Comment explains nearby logic, constraints, or intent: `For widen operation which has different mangling name.`.
  **L743 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For widen operation which has different mangling name.`。
- **L744 EN**: Declares TableGen multiclass record `RVVWidenBuiltinSet`.
  **L744 CN**: 声明 TableGen multiclass 记录 `RVVWidenBuiltinSet`。

### Lines 745-768

````tablegen
                              list<list<string>> suffixes_prototypes> {
  let Log2LMUL = [-3, -2, -1, 0, 1, 2],
      IRName = intrinsic_name, MaskedIRName = intrinsic_name # "_mask" in {
    foreach s_p = suffixes_prototypes in {
      let Name = NAME # "_" # s_p[0],
          OverloadedName = NAME # "_" # s_p[0] in {
        defvar suffix = s_p[1];
        defvar prototype = s_p[2];
        def : RVVOutOp0Op1Builtin<suffix, prototype, type_range>;
      }
    }
  }
}

// For widen operation with widen operand which has different mangling name.
multiclass RVVWidenWOp0BuiltinSet<string intrinsic_name, string type_range,
                                  list<list<string>> suffixes_prototypes> {
  let Log2LMUL = [-3, -2, -1, 0, 1, 2],
      IRName = intrinsic_name, MaskedIRName = intrinsic_name # "_mask" in {
    foreach s_p = suffixes_prototypes in {
      let Name = NAME # "_" # s_p[0],
          OverloadedName = NAME # "_" # s_p[0] in {
        defvar suffix = s_p[1];
        defvar prototype = s_p[2];
````
- **L745 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes> {`.
  **L745 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes> {`。
- **L746 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2],`.
  **L746 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2],`。
- **L747 EN**: Continues the surrounding expression or declaration: `IRName = intrinsic_name, MaskedIRName = intrinsic_name # "_mask" in {`.
  **L747 CN**: 继续构造周围的表达式或声明：`IRName = intrinsic_name, MaskedIRName = intrinsic_name # "_mask" in {`。
- **L748 EN**: Starts a TableGen iteration used to generate repeated records: `foreach s_p = suffixes_prototypes in {`.
  **L748 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach s_p = suffixes_prototypes in {`。
- **L749 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_" # s_p[0],`.
  **L749 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_" # s_p[0],`。
- **L750 EN**: Continues the surrounding expression or declaration: `OverloadedName = NAME # "_" # s_p[0] in {`.
  **L750 CN**: 继续构造周围的表达式或声明：`OverloadedName = NAME # "_" # s_p[0] in {`。
- **L751 EN**: Declares TableGen defvar record `suffix = s_p[1];`.
  **L751 CN**: 声明 TableGen defvar 记录 `suffix = s_p[1];`。
- **L752 EN**: Declares TableGen defvar record `prototype = s_p[2];`.
  **L752 CN**: 声明 TableGen defvar 记录 `prototype = s_p[2];`。
- **L753 EN**: Declares TableGen def record `def`.
  **L753 CN**: 声明 TableGen def 记录 `def`。
- **L754 EN**: Closes the current lexical scope or compound statement.
  **L754 CN**: 结束当前词法作用域或复合语句块。
- **L755 EN**: Closes the current lexical scope or compound statement.
  **L755 CN**: 结束当前词法作用域或复合语句块。
- **L756 EN**: Closes the current lexical scope or compound statement.
  **L756 CN**: 结束当前词法作用域或复合语句块。
- **L757 EN**: Closes the current lexical scope or compound statement.
  **L757 CN**: 结束当前词法作用域或复合语句块。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L759 EN**: Comment explains nearby logic, constraints, or intent: `For widen operation with widen operand which has different mangling name.`.
  **L759 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For widen operation with widen operand which has different mangling name.`。
- **L760 EN**: Declares TableGen multiclass record `RVVWidenWOp0BuiltinSet`.
  **L760 CN**: 声明 TableGen multiclass 记录 `RVVWidenWOp0BuiltinSet`。
- **L761 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes> {`.
  **L761 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes> {`。
- **L762 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2],`.
  **L762 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2],`。
- **L763 EN**: Continues the surrounding expression or declaration: `IRName = intrinsic_name, MaskedIRName = intrinsic_name # "_mask" in {`.
  **L763 CN**: 继续构造周围的表达式或声明：`IRName = intrinsic_name, MaskedIRName = intrinsic_name # "_mask" in {`。
- **L764 EN**: Starts a TableGen iteration used to generate repeated records: `foreach s_p = suffixes_prototypes in {`.
  **L764 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach s_p = suffixes_prototypes in {`。
- **L765 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_" # s_p[0],`.
  **L765 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_" # s_p[0],`。
- **L766 EN**: Continues the surrounding expression or declaration: `OverloadedName = NAME # "_" # s_p[0] in {`.
  **L766 CN**: 继续构造周围的表达式或声明：`OverloadedName = NAME # "_" # s_p[0] in {`。
- **L767 EN**: Declares TableGen defvar record `suffix = s_p[1];`.
  **L767 CN**: 声明 TableGen defvar 记录 `suffix = s_p[1];`。
- **L768 EN**: Declares TableGen defvar record `prototype = s_p[2];`.
  **L768 CN**: 声明 TableGen defvar 记录 `prototype = s_p[2];`。

### Lines 769-792

````tablegen
        def : RVVOutOp1Builtin<suffix, prototype, type_range>;
      }
    }
  }
}

multiclass RVVSignedWidenBinBuiltinSet
    : RVVWidenBuiltinSet<NAME, "csi",
                         [["vv", "w", "wvv"],
                          ["vx", "w", "wve"]]>;

multiclass RVVSignedWidenOp0BinBuiltinSet
    : RVVWidenWOp0BuiltinSet<NAME # "_w", "csi",
                             [["wv", "w", "wwv"],
                              ["wx", "w", "wwe"]]>;

multiclass RVVUnsignedWidenBinBuiltinSet
    : RVVWidenBuiltinSet<NAME, "csi",
                         [["vv", "Uw", "UwUvUv"],
                          ["vx", "Uw", "UwUvUe"]]>;

multiclass RVVUnsignedWidenOp0BinBuiltinSet
    : RVVWidenWOp0BuiltinSet<NAME # "_w", "csi",
                             [["wv", "Uw", "UwUwUv"],
````
- **L769 EN**: Declares TableGen def record `def`.
  **L769 CN**: 声明 TableGen def 记录 `def`。
- **L770 EN**: Closes the current lexical scope or compound statement.
  **L770 CN**: 结束当前词法作用域或复合语句块。
- **L771 EN**: Closes the current lexical scope or compound statement.
  **L771 CN**: 结束当前词法作用域或复合语句块。
- **L772 EN**: Closes the current lexical scope or compound statement.
  **L772 CN**: 结束当前词法作用域或复合语句块。
- **L773 EN**: Closes the current lexical scope or compound statement.
  **L773 CN**: 结束当前词法作用域或复合语句块。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L775 EN**: Declares TableGen multiclass record `RVVSignedWidenBinBuiltinSet`.
  **L775 CN**: 声明 TableGen multiclass 记录 `RVVSignedWidenBinBuiltinSet`。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVWidenBuiltinSet<NAME, "csi",`.
  **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVWidenBuiltinSet<NAME, "csi",`。
- **L777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvv"],`.
  **L777 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvv"],`。
- **L778 EN**: Adds a standalone statement or declaration: `["vx", "w", "wve"]]>;`.
  **L778 CN**: 添加一条独立语句或声明：`["vx", "w", "wve"]]>;`。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L780 EN**: Declares TableGen multiclass record `RVVSignedWidenOp0BinBuiltinSet`.
  **L780 CN**: 声明 TableGen multiclass 记录 `RVVSignedWidenOp0BinBuiltinSet`。
- **L781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVWidenWOp0BuiltinSet<NAME # "_w", "csi",`.
  **L781 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVWidenWOp0BuiltinSet<NAME # "_w", "csi",`。
- **L782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "w", "wwv"],`.
  **L782 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "w", "wwv"],`。
- **L783 EN**: Adds a standalone statement or declaration: `["wx", "w", "wwe"]]>;`.
  **L783 CN**: 添加一条独立语句或声明：`["wx", "w", "wwe"]]>;`。
- **L784 EN**: Blank line separating nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L785 EN**: Declares TableGen multiclass record `RVVUnsignedWidenBinBuiltinSet`.
  **L785 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedWidenBinBuiltinSet`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVWidenBuiltinSet<NAME, "csi",`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVWidenBuiltinSet<NAME, "csi",`。
- **L787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uw", "UwUvUv"],`.
  **L787 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uw", "UwUvUv"],`。
- **L788 EN**: Adds a standalone statement or declaration: `["vx", "Uw", "UwUvUe"]]>;`.
  **L788 CN**: 添加一条独立语句或声明：`["vx", "Uw", "UwUvUe"]]>;`。
- **L789 EN**: Blank line separating nearby declarations or logic blocks.
  **L789 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L790 EN**: Declares TableGen multiclass record `RVVUnsignedWidenOp0BinBuiltinSet`.
  **L790 CN**: 声明 TableGen multiclass 记录 `RVVUnsignedWidenOp0BinBuiltinSet`。
- **L791 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVWidenWOp0BuiltinSet<NAME # "_w", "csi",`.
  **L791 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVWidenWOp0BuiltinSet<NAME # "_w", "csi",`。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "Uw", "UwUwUv"],`.
  **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "Uw", "UwUwUv"],`。

### Lines 793-816

````tablegen
                              ["wx", "Uw", "UwUwUe"]]>;

multiclass RVVFloatingWidenBinBuiltinSet {
  defm "" : RVVWidenBuiltinSet<NAME, "f",
                               [["vv", "w", "wvv"],
                                ["vf", "w", "wve"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm "" : RVVWidenBuiltinSet<NAME, "x",
                                 [["vv", "w", "wvv"],
                                  ["vf", "w", "wve"]]>;
  let RequiredFeatures = ["zvfbfa"] in
    defm "" : RVVWidenBuiltinSet<NAME, "y",
                                 [["vv", "vw", "wvv"],
                                  ["vf", "vw", "wve"]]>;
}

multiclass RVVFloatingWidenBinBuiltinSetRoundingMode {
  defm "" : RVVWidenBuiltinSet<NAME, "f",
                               [["vv", "w", "wvvu"],
                                ["vf", "w", "wveu"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm "" : RVVWidenBuiltinSet<NAME, "x",
                                 [["vv", "w", "wvvu"],
                                  ["vf", "w", "wveu"]]>;
````
- **L793 EN**: Adds a standalone statement or declaration: `["wx", "Uw", "UwUwUe"]]>;`.
  **L793 CN**: 添加一条独立语句或声明：`["wx", "Uw", "UwUwUe"]]>;`。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L795 EN**: Declares TableGen multiclass record `RVVFloatingWidenBinBuiltinSet`.
  **L795 CN**: 声明 TableGen multiclass 记录 `RVVFloatingWidenBinBuiltinSet`。
- **L796 EN**: Declares TableGen defm record `""`.
  **L796 CN**: 声明 TableGen defm 记录 `""`。
- **L797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvv"],`.
  **L797 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvv"],`。
- **L798 EN**: Adds a standalone statement or declaration: `["vf", "w", "wve"]]>;`.
  **L798 CN**: 添加一条独立语句或声明：`["vf", "w", "wve"]]>;`。
- **L799 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L799 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L800 EN**: Declares TableGen defm record `""`.
  **L800 CN**: 声明 TableGen defm 记录 `""`。
- **L801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvv"],`.
  **L801 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvv"],`。
- **L802 EN**: Adds a standalone statement or declaration: `["vf", "w", "wve"]]>;`.
  **L802 CN**: 添加一条独立语句或声明：`["vf", "w", "wve"]]>;`。
- **L803 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L803 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L804 EN**: Declares TableGen defm record `""`.
  **L804 CN**: 声明 TableGen defm 记录 `""`。
- **L805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vw", "wvv"],`.
  **L805 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vw", "wvv"],`。
- **L806 EN**: Adds a standalone statement or declaration: `["vf", "vw", "wve"]]>;`.
  **L806 CN**: 添加一条独立语句或声明：`["vf", "vw", "wve"]]>;`。
- **L807 EN**: Closes the current lexical scope or compound statement.
  **L807 CN**: 结束当前词法作用域或复合语句块。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L809 EN**: Declares TableGen multiclass record `RVVFloatingWidenBinBuiltinSetRoundingMode`.
  **L809 CN**: 声明 TableGen multiclass 记录 `RVVFloatingWidenBinBuiltinSetRoundingMode`。
- **L810 EN**: Declares TableGen defm record `""`.
  **L810 CN**: 声明 TableGen defm 记录 `""`。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvvu"],`.
  **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvvu"],`。
- **L812 EN**: Adds a standalone statement or declaration: `["vf", "w", "wveu"]]>;`.
  **L812 CN**: 添加一条独立语句或声明：`["vf", "w", "wveu"]]>;`。
- **L813 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L813 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L814 EN**: Declares TableGen defm record `""`.
  **L814 CN**: 声明 TableGen defm 记录 `""`。
- **L815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvvu"],`.
  **L815 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvvu"],`。
- **L816 EN**: Adds a standalone statement or declaration: `["vf", "w", "wveu"]]>;`.
  **L816 CN**: 添加一条独立语句或声明：`["vf", "w", "wveu"]]>;`。

### Lines 817-840

````tablegen
  let RequiredFeatures = ["zvfbfa"] in
    defm "" : RVVWidenBuiltinSet<NAME, "y",
                                 [["vv", "vw", "wvvu"],
                                  ["vf", "vw", "wveu"]]>;
}

multiclass RVVFloatingWidenOp0BinBuiltinSet {
  defm "" : RVVWidenWOp0BuiltinSet<NAME # "_w", "f",
                                   [["wv", "w", "wwv"],
                                    ["wf", "w", "wwe"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm "" : RVVWidenWOp0BuiltinSet<NAME # "_w", "x",
                                     [["wv", "w", "wwv"],
                                      ["wf", "w", "wwe"]]>;
  let RequiredFeatures = ["zvfbfa"] in
    defm "" : RVVWidenWOp0BuiltinSet<NAME # "_w", "y",
                                     [["wv", "vw", "wwv"],
                                      ["wf", "ew", "wwe"]]>;
}

multiclass RVVFloatingWidenOp0BinBuiltinSetRoundingMode {
  defm "" : RVVWidenWOp0BuiltinSet<NAME # "_w", "f",
                                   [["wv", "w", "wwvu"],
                                    ["wf", "w", "wweu"]]>;
````
- **L817 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L817 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L818 EN**: Declares TableGen defm record `""`.
  **L818 CN**: 声明 TableGen defm 记录 `""`。
- **L819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vw", "wvvu"],`.
  **L819 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vw", "wvvu"],`。
- **L820 EN**: Adds a standalone statement or declaration: `["vf", "vw", "wveu"]]>;`.
  **L820 CN**: 添加一条独立语句或声明：`["vf", "vw", "wveu"]]>;`。
- **L821 EN**: Closes the current lexical scope or compound statement.
  **L821 CN**: 结束当前词法作用域或复合语句块。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Declares TableGen multiclass record `RVVFloatingWidenOp0BinBuiltinSet`.
  **L823 CN**: 声明 TableGen multiclass 记录 `RVVFloatingWidenOp0BinBuiltinSet`。
- **L824 EN**: Declares TableGen defm record `""`.
  **L824 CN**: 声明 TableGen defm 记录 `""`。
- **L825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "w", "wwv"],`.
  **L825 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "w", "wwv"],`。
- **L826 EN**: Adds a standalone statement or declaration: `["wf", "w", "wwe"]]>;`.
  **L826 CN**: 添加一条独立语句或声明：`["wf", "w", "wwe"]]>;`。
- **L827 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L827 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L828 EN**: Declares TableGen defm record `""`.
  **L828 CN**: 声明 TableGen defm 记录 `""`。
- **L829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "w", "wwv"],`.
  **L829 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "w", "wwv"],`。
- **L830 EN**: Adds a standalone statement or declaration: `["wf", "w", "wwe"]]>;`.
  **L830 CN**: 添加一条独立语句或声明：`["wf", "w", "wwe"]]>;`。
- **L831 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L831 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L832 EN**: Declares TableGen defm record `""`.
  **L832 CN**: 声明 TableGen defm 记录 `""`。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "vw", "wwv"],`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "vw", "wwv"],`。
- **L834 EN**: Adds a standalone statement or declaration: `["wf", "ew", "wwe"]]>;`.
  **L834 CN**: 添加一条独立语句或声明：`["wf", "ew", "wwe"]]>;`。
- **L835 EN**: Closes the current lexical scope or compound statement.
  **L835 CN**: 结束当前词法作用域或复合语句块。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L837 EN**: Declares TableGen multiclass record `RVVFloatingWidenOp0BinBuiltinSetRoundingMode`.
  **L837 CN**: 声明 TableGen multiclass 记录 `RVVFloatingWidenOp0BinBuiltinSetRoundingMode`。
- **L838 EN**: Declares TableGen defm record `""`.
  **L838 CN**: 声明 TableGen defm 记录 `""`。
- **L839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "w", "wwvu"],`.
  **L839 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "w", "wwvu"],`。
- **L840 EN**: Adds a standalone statement or declaration: `["wf", "w", "wweu"]]>;`.
  **L840 CN**: 添加一条独立语句或声明：`["wf", "w", "wweu"]]>;`。

### Lines 841-849

````tablegen
  let RequiredFeatures = ["zvfh"] in
    defm "" : RVVWidenWOp0BuiltinSet<NAME # "_w", "x",
                                     [["wv", "w", "wwvu"],
                                      ["wf", "w", "wweu"]]>;
  let RequiredFeatures = ["zvfbfa"] in
    defm "" : RVVWidenWOp0BuiltinSet<NAME # "_w", "y",
                                     [["wv", "vw", "wwvu"],
                                      ["wf", "ew", "wweu"]]>;
}
````
- **L841 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L841 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L842 EN**: Declares TableGen defm record `""`.
  **L842 CN**: 声明 TableGen defm 记录 `""`。
- **L843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "w", "wwvu"],`.
  **L843 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "w", "wwvu"],`。
- **L844 EN**: Adds a standalone statement or declaration: `["wf", "w", "wweu"]]>;`.
  **L844 CN**: 添加一条独立语句或声明：`["wf", "w", "wweu"]]>;`。
- **L845 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L845 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L846 EN**: Declares TableGen defm record `""`.
  **L846 CN**: 声明 TableGen defm 记录 `""`。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["wv", "vw", "wwvu"],`.
  **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["wv", "vw", "wwvu"],`。
- **L848 EN**: Adds a standalone statement or declaration: `["wf", "ew", "wweu"]]>;`.
  **L848 CN**: 添加一条独立语句或声明：`["wf", "ew", "wweu"]]>;`。
- **L849 EN**: Closes the current lexical scope or compound statement.
  **L849 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Compiler configuration / 编译器配置**
  - **EN**: Holds low-level enums, constants, feature switches, and descriptive metadata used across Clang.
  - **CN**: 保存 Clang 各处复用的底层枚举、常量、特性开关与描述性元数据。
- **Target and language metadata / 目标与语言元数据**
  - **EN**: Models ABI, address-space, builtin, and target-feature information shared by many subsystems.
  - **CN**: 建模多个子系统共享的 ABI、地址空间、builtin 与目标特性信息。
- **Generated definition tables / 生成式定义表**
  - **EN**: Many Basic headers use `.td` or `.def` files to describe families of diagnostics, attributes, or builtins declaratively.
  - **CN**: 许多 Basic 头文件使用 `.td` 或 `.def` 文件以声明式方式描述诊断、属性或 builtin 家族。
- **TableGen declarative modeling / TableGen 声明式建模**
  - **EN**: Uses TableGen records, inheritance, and generators to describe families of compiler entities compactly.
  - **CN**: 使用 TableGen 记录、继承与生成器紧凑地描述一组编译器实体。
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。
- **Attribute metadata / 属性元数据**
  - **EN**: Captures attribute spellings, subjects, semantic flags, and generated helper data.
  - **CN**: 刻画属性的拼写、适用对象、语义标志与生成的辅助数据。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **RISC-V target support / RISC-V 目标支持**
  - **EN**: Captures RISC-V-specific builtin, vector, or target-feature information.
  - **CN**: 刻画 RISC-V 专用 builtin、向量或目标特性信息。
- **Record families / 记录族**
  - **EN**: Groups related TableGen records so backends can derive structured generated output.
  - **CN**: 将相关 TableGen 记录组织成族，以便后端导出结构化的生成结果。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `for`, `RVVBuiltin`, `PolicyScheme`, `RVVHeader`, `RVVOutBuiltin`, `RVVOp0Builtin`, `RVVOutOp0Builtin`, `RVVOutOp1Builtin`, `RVVOutOp0Op1Builtin`, `RVVIntExt`, `RVVMaskBinBuiltin`, `RVVMaskUnaryBuiltin`
- **Functions or callables / 函数或可调用对象**: `int8_t`, `int16_t`, `int32_t`, `int64_t`, `float16_t`, `float32_t`, `float64_t`, `bfloat16_t`, `builtin`, `is`, `type`, `__rvv_uint8m2_t`
- **TableGen records / TableGen 记录**: `PolicyScheme`, `NonePolicy`, `HasPassthruOperand`, `HasPolicyOperand`, `RVVBuiltin`, `RVVHeader`, `RVVOutBuiltin`, `RVVOp0Builtin`, `RVVOutOp0Builtin`, `RVVOutOp1Builtin`, `RVVOutOp0Op1Builtin`, `RVVBuiltinSet`, `suffix`, `prototype`, `RVVOutOp0Op1BuiltinSet`, `RVVOutBuiltinSet`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
