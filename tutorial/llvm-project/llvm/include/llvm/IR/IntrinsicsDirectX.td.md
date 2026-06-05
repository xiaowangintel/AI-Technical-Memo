# IntrinsicsDirectX.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsDirectX.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the DirectX-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsDirectX` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````tablegen
//===- IntrinsicsDirectX.td - Defines DirectX intrinsics ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the DirectX-specific intrinsics.
//
//===----------------------------------------------------------------------===//

let TargetPrefix = "dx" in {

def int_dx_thread_id : Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem, IntrWillReturn]>;
def int_dx_group_id : Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem, IntrWillReturn]>;
def int_dx_thread_id_in_group : Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem, IntrWillReturn]>;
def int_dx_flattened_thread_id_in_group : Intrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrWillReturn]>;

// Create resource handle given binding information. Returns a `target("dx.")`
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the DirectX-specific intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the DirectX-specific intrinsics.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L13 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Declares TableGen def `int_dx_thread_id`.
  **L15 CN**: 声明 TableGen def `int_dx_thread_id`。
- **L16 EN**: Declares TableGen def `int_dx_group_id`.
  **L16 CN**: 声明 TableGen def `int_dx_group_id`。
- **L17 EN**: Declares TableGen def `int_dx_thread_id_in_group`.
  **L17 CN**: 声明 TableGen def `int_dx_thread_id_in_group`。
- **L18 EN**: Declares TableGen def `int_dx_flattened_thread_id_in_group`.
  **L18 CN**: 声明 TableGen def `int_dx_flattened_thread_id_in_group`。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Comment explains nearby logic, invariants, or intent: `Create resource handle given binding information. Returns a `target("dx.")``.
  **L20 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create resource handle given binding information. Returns a `target("dx.")``。

### Lines 21-40

````tablegen
// type appropriate for the kind of resource given a register space ID, lower
// bound and range size of the binding, as well as an index and an indicator
// whether that index may be non-uniform.
def int_dx_resource_handlefrombinding
    : DefaultAttrsIntrinsic<
          [llvm_any_ty],
          [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty],
          [IntrNoMem]>;

// Create resource handle with implicit binding in given register space.
// Returns a `target("dx.")` type appropriate for the kind of resource and
// the range size and index of the binding.
def int_dx_resource_handlefromimplicitbinding
    : DefaultAttrsIntrinsic<
          [llvm_any_ty],
          [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty],
          [IntrNoMem]>;

def int_dx_resource_getpointer
    : DefaultAttrsIntrinsic<[llvm_anyptr_ty], [llvm_any_ty, llvm_any_ty],
````
- **L21 EN**: Comment explains nearby logic, invariants, or intent: `type appropriate for the kind of resource given a register space ID, lower`.
  **L21 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`type appropriate for the kind of resource given a register space ID, lower`。
- **L22 EN**: Comment explains nearby logic, invariants, or intent: `bound and range size of the binding, as well as an index and an indicator`.
  **L22 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bound and range size of the binding, as well as an index and an indicator`。
- **L23 EN**: Comment explains nearby logic, invariants, or intent: `whether that index may be non-uniform.`.
  **L23 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`whether that index may be non-uniform.`。
- **L24 EN**: Declares TableGen def `int_dx_resource_handlefrombinding`.
  **L24 CN**: 声明 TableGen def `int_dx_resource_handlefrombinding`。
- **L25 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L25 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L26 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty],`.
  **L26 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty],`。
- **L27 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty],`.
  **L27 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty],`。
- **L28 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L28 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Comment explains nearby logic, invariants, or intent: `Create resource handle with implicit binding in given register space.`.
  **L30 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create resource handle with implicit binding in given register space.`。
- **L31 EN**: Comment explains nearby logic, invariants, or intent: `Returns a `target("dx.")` type appropriate for the kind of resource and`.
  **L31 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns a `target("dx.")` type appropriate for the kind of resource and`。
- **L32 EN**: Comment explains nearby logic, invariants, or intent: `the range size and index of the binding.`.
  **L32 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the range size and index of the binding.`。
- **L33 EN**: Declares TableGen def `int_dx_resource_handlefromimplicitbinding`.
  **L33 CN**: 声明 TableGen def `int_dx_resource_handlefromimplicitbinding`。
- **L34 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L34 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty],`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty],`。
- **L36 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty],`.
  **L36 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty],`。
- **L37 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L37 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L39 EN**: Declares TableGen def `int_dx_resource_getpointer`.
  **L39 CN**: 声明 TableGen def `int_dx_resource_getpointer`。
- **L40 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyptr_ty], [llvm_any_ty, llvm_any_ty],`.
  **L40 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyptr_ty], [llvm_any_ty, llvm_any_ty],`。

### Lines 41-60

````tablegen
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

def int_dx_resource_getbasepointer
    : DefaultAttrsIntrinsic<[llvm_anyptr_ty], [llvm_any_ty],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

def int_dx_resource_nonuniformindex
    : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;

def int_dx_resource_load_typedbuffer
    : DefaultAttrsIntrinsic<[llvm_any_ty, llvm_i1_ty],
                            [llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;
def int_dx_resource_store_typedbuffer
    : DefaultAttrsIntrinsic<[], [llvm_any_ty, llvm_i32_ty, llvm_any_ty],
                            [IntrWriteMem]>;
def int_dx_resource_load_rawbuffer
    : DefaultAttrsIntrinsic<[llvm_any_ty, llvm_i1_ty],
                            [llvm_any_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrReadMem]>;
def int_dx_resource_store_rawbuffer
````
- **L41 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L41 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Declares TableGen def `int_dx_resource_getbasepointer`.
  **L43 CN**: 声明 TableGen def `int_dx_resource_getbasepointer`。
- **L44 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyptr_ty], [llvm_any_ty],`.
  **L44 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyptr_ty], [llvm_any_ty],`。
- **L45 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L45 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Declares TableGen def `int_dx_resource_nonuniformindex`.
  **L47 CN**: 声明 TableGen def `int_dx_resource_nonuniformindex`。
- **L48 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L48 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Declares TableGen def `int_dx_resource_load_typedbuffer`.
  **L50 CN**: 声明 TableGen def `int_dx_resource_load_typedbuffer`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_i1_ty],`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_i1_ty],`。
- **L52 EN**: Executes a standalone statement or declaration: `[llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`.
  **L52 CN**: 执行一条独立语句或声明：`[llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`。
- **L53 EN**: Declares TableGen def `int_dx_resource_store_typedbuffer`.
  **L53 CN**: 声明 TableGen def `int_dx_resource_store_typedbuffer`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_any_ty, llvm_i32_ty, llvm_any_ty],`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_any_ty, llvm_i32_ty, llvm_any_ty],`。
- **L55 EN**: Executes a standalone statement or declaration: `[IntrWriteMem]>;`.
  **L55 CN**: 执行一条独立语句或声明：`[IntrWriteMem]>;`。
- **L56 EN**: Declares TableGen def `int_dx_resource_load_rawbuffer`.
  **L56 CN**: 声明 TableGen def `int_dx_resource_load_rawbuffer`。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_i1_ty],`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_i1_ty],`。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L59 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L59 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L60 EN**: Declares TableGen def `int_dx_resource_store_rawbuffer`.
  **L60 CN**: 声明 TableGen def `int_dx_resource_store_rawbuffer`。

### Lines 61-80

````tablegen
    : DefaultAttrsIntrinsic<
          [], [llvm_any_ty, llvm_i32_ty, llvm_i32_ty, llvm_any_ty],
          [IntrWriteMem]>;

// dx.resource.load.cbufferrow encodes the number of elements returned in the
// function name. The total size of the return should always be 128 bits.
def int_dx_resource_load_cbufferrow_8
    : DefaultAttrsIntrinsic<
          [llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,
           llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty],
          [llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;
def int_dx_resource_load_cbufferrow_4
    : DefaultAttrsIntrinsic<
          [llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty],
          [llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;
def int_dx_resource_load_cbufferrow_2
    : DefaultAttrsIntrinsic<[llvm_any_ty, llvm_any_ty],
                            [llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;

def int_dx_resource_updatecounter
````
- **L61 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L61 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_any_ty, llvm_i32_ty, llvm_i32_ty, llvm_any_ty],`.
  **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_any_ty, llvm_i32_ty, llvm_i32_ty, llvm_any_ty],`。
- **L63 EN**: Executes a standalone statement or declaration: `[IntrWriteMem]>;`.
  **L63 CN**: 执行一条独立语句或声明：`[IntrWriteMem]>;`。
- **L64 EN**: Blank line separating nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L65 EN**: Comment explains nearby logic, invariants, or intent: `dx.resource.load.cbufferrow encodes the number of elements returned in the`.
  **L65 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`dx.resource.load.cbufferrow encodes the number of elements returned in the`。
- **L66 EN**: Comment explains nearby logic, invariants, or intent: `function name. The total size of the return should always be 128 bits.`.
  **L66 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`function name. The total size of the return should always be 128 bits.`。
- **L67 EN**: Declares TableGen def `int_dx_resource_load_cbufferrow_8`.
  **L67 CN**: 声明 TableGen def `int_dx_resource_load_cbufferrow_8`。
- **L68 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L68 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L69 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L69 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L70 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty],`.
  **L70 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty],`。
- **L71 EN**: Executes a standalone statement or declaration: `[llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`.
  **L71 CN**: 执行一条独立语句或声明：`[llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`。
- **L72 EN**: Declares TableGen def `int_dx_resource_load_cbufferrow_4`.
  **L72 CN**: 声明 TableGen def `int_dx_resource_load_cbufferrow_4`。
- **L73 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L73 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty],`.
  **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty],`。
- **L75 EN**: Executes a standalone statement or declaration: `[llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`.
  **L75 CN**: 执行一条独立语句或声明：`[llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`。
- **L76 EN**: Declares TableGen def `int_dx_resource_load_cbufferrow_2`.
  **L76 CN**: 声明 TableGen def `int_dx_resource_load_cbufferrow_2`。
- **L77 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_any_ty],`.
  **L77 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty, llvm_any_ty],`。
- **L78 EN**: Executes a standalone statement or declaration: `[llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`.
  **L78 CN**: 执行一条独立语句或声明：`[llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`。
- **L79 EN**: Blank line separating nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L80 EN**: Declares TableGen def `int_dx_resource_updatecounter`.
  **L80 CN**: 声明 TableGen def `int_dx_resource_updatecounter`。

### Lines 81-100

````tablegen
    : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_any_ty, llvm_i8_ty],
                            [IntrInaccessibleMemOrArgMemOnly]>;

def int_dx_resource_getdimensions_x
    : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_any_ty], [IntrReadMem]>;
def int_dx_resource_getdimensions_xy
    : DefaultAttrsIntrinsic<[llvm_v2i32_ty], [llvm_any_ty], [IntrReadMem]>;
def int_dx_resource_getdimensions_levels_xy
    : DefaultAttrsIntrinsic<[llvm_v3i32_ty], [llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;

def int_dx_resource_sample
    : DefaultAttrsIntrinsic<
          [llvm_any_ty], [llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty],
          [IntrReadMem]>;

def int_dx_resource_sample_clamp
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_float_ty],
                            [IntrReadMem]>;
````
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_any_ty, llvm_i8_ty],`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_any_ty, llvm_i8_ty],`。
- **L82 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly]>;`.
  **L82 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly]>;`。
- **L83 EN**: Blank line separating nearby declarations or logic blocks.
  **L83 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L84 EN**: Declares TableGen def `int_dx_resource_getdimensions_x`.
  **L84 CN**: 声明 TableGen def `int_dx_resource_getdimensions_x`。
- **L85 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_any_ty], [IntrReadMem]>;`.
  **L85 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_any_ty], [IntrReadMem]>;`。
- **L86 EN**: Declares TableGen def `int_dx_resource_getdimensions_xy`.
  **L86 CN**: 声明 TableGen def `int_dx_resource_getdimensions_xy`。
- **L87 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v2i32_ty], [llvm_any_ty], [IntrReadMem]>;`.
  **L87 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v2i32_ty], [llvm_any_ty], [IntrReadMem]>;`。
- **L88 EN**: Declares TableGen def `int_dx_resource_getdimensions_levels_xy`.
  **L88 CN**: 声明 TableGen def `int_dx_resource_getdimensions_levels_xy`。
- **L89 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v3i32_ty], [llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`.
  **L89 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v3i32_ty], [llvm_any_ty, llvm_i32_ty], [IntrReadMem]>;`。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Declares TableGen def `int_dx_resource_sample`.
  **L91 CN**: 声明 TableGen def `int_dx_resource_sample`。
- **L92 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L92 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty], [llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty],`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty], [llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty],`。
- **L94 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L94 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L96 EN**: Declares TableGen def `int_dx_resource_sample_clamp`.
  **L96 CN**: 声明 TableGen def `int_dx_resource_sample_clamp`。
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L98 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L98 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L99 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_float_ty],`.
  **L99 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_float_ty],`。
- **L100 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L100 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。

### Lines 101-120

````tablegen
def int_dx_resource_samplebias
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_float_ty, llvm_any_ty],
                            [IntrReadMem]>;

def int_dx_resource_samplebias_clamp
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_float_ty, llvm_any_ty, llvm_float_ty],
                            [IntrReadMem]>;

def int_dx_resource_samplegrad
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_any_ty, llvm_any_ty],
                            [IntrReadMem]>;

def int_dx_resource_samplegrad_clamp
    : DefaultAttrsIntrinsic<[llvm_any_ty],
````
- **L101 EN**: Declares TableGen def `int_dx_resource_samplebias`.
  **L101 CN**: 声明 TableGen def `int_dx_resource_samplebias`。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_float_ty, llvm_any_ty],`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_float_ty, llvm_any_ty],`。
- **L105 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L105 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L107 EN**: Declares TableGen def `int_dx_resource_samplebias_clamp`.
  **L107 CN**: 声明 TableGen def `int_dx_resource_samplebias_clamp`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_float_ty, llvm_any_ty, llvm_float_ty],`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_float_ty, llvm_any_ty, llvm_float_ty],`。
- **L111 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L111 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L112 EN**: Blank line separating nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L113 EN**: Declares TableGen def `int_dx_resource_samplegrad`.
  **L113 CN**: 声明 TableGen def `int_dx_resource_samplegrad`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_any_ty, llvm_any_ty],`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_any_ty, llvm_any_ty],`。
- **L117 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L117 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L119 EN**: Declares TableGen def `int_dx_resource_samplegrad_clamp`.
  **L119 CN**: 声明 TableGen def `int_dx_resource_samplegrad_clamp`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。

### Lines 121-140

````tablegen
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_any_ty, llvm_any_ty, llvm_float_ty],
                            [IntrReadMem]>;

def int_dx_resource_samplelevel
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_float_ty, llvm_any_ty],
                            [IntrReadMem]>;

def int_dx_resource_load_level
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_any_ty],
                            [IntrReadMem]>;

def int_dx_resource_calculate_lod
    : DefaultAttrsIntrinsic<[llvm_float_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty],
                            [IntrReadMem]>;
````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_any_ty, llvm_any_ty, llvm_float_ty],`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_any_ty, llvm_any_ty, llvm_float_ty],`。
- **L123 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L123 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L125 EN**: Declares TableGen def `int_dx_resource_samplelevel`.
  **L125 CN**: 声明 TableGen def `int_dx_resource_samplelevel`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L127 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_float_ty, llvm_any_ty],`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_float_ty, llvm_any_ty],`。
- **L129 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L129 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L131 EN**: Declares TableGen def `int_dx_resource_load_level`.
  **L131 CN**: 声明 TableGen def `int_dx_resource_load_level`。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_any_ty],`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_any_ty],`。
- **L135 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L135 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L137 EN**: Declares TableGen def `int_dx_resource_calculate_lod`.
  **L137 CN**: 声明 TableGen def `int_dx_resource_calculate_lod`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty],`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty],`。
- **L140 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L140 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。

### Lines 141-160

````tablegen

def int_dx_resource_calculate_lod_unclamped
    : DefaultAttrsIntrinsic<[llvm_float_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty],
                            [IntrReadMem]>;

def int_dx_resource_samplecmp
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_float_ty, llvm_any_ty],
                            [IntrReadMem]>;

def int_dx_resource_samplecmp_clamp
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_float_ty, llvm_any_ty, llvm_float_ty],
                            [IntrReadMem]>;

def int_dx_resource_samplecmplevelzero
    : DefaultAttrsIntrinsic<[llvm_any_ty],
````
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L142 EN**: Declares TableGen def `int_dx_resource_calculate_lod_unclamped`.
  **L142 CN**: 声明 TableGen def `int_dx_resource_calculate_lod_unclamped`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty],`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty],`。
- **L145 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L145 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L147 EN**: Declares TableGen def `int_dx_resource_samplecmp`.
  **L147 CN**: 声明 TableGen def `int_dx_resource_samplecmp`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L149 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_float_ty, llvm_any_ty],`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_float_ty, llvm_any_ty],`。
- **L151 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L151 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L153 EN**: Declares TableGen def `int_dx_resource_samplecmp_clamp`.
  **L153 CN**: 声明 TableGen def `int_dx_resource_samplecmp_clamp`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_float_ty, llvm_any_ty, llvm_float_ty],`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_float_ty, llvm_any_ty, llvm_float_ty],`。
- **L157 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L157 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L159 EN**: Declares TableGen def `int_dx_resource_samplecmplevelzero`.
  **L159 CN**: 声明 TableGen def `int_dx_resource_samplecmplevelzero`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。

### Lines 161-180

````tablegen
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_float_ty, llvm_any_ty],
                            [IntrReadMem]>;

def int_dx_resource_gather
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_i32_ty, llvm_any_ty],
                            [IntrReadMem]>;

def int_dx_resource_gather_cmp
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_any_ty, llvm_any_ty, llvm_any_ty,
                             llvm_float_ty, llvm_i32_ty, llvm_any_ty],
                            [IntrReadMem]>;

// Cast between target extension handle types and dxil-style opaque handles
def int_dx_resource_casthandle : Intrinsic<[llvm_any_ty], [llvm_any_ty]>;

def int_dx_all : DefaultAttrsIntrinsic<[llvm_i1_ty], [llvm_any_ty], [IntrNoMem]>;
````
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_float_ty, llvm_any_ty],`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_float_ty, llvm_any_ty],`。
- **L163 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L163 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L165 EN**: Declares TableGen def `int_dx_resource_gather`.
  **L165 CN**: 声明 TableGen def `int_dx_resource_gather`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_any_ty],`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_any_ty],`。
- **L169 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L169 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L171 EN**: Declares TableGen def `int_dx_resource_gather_cmp`.
  **L171 CN**: 声明 TableGen def `int_dx_resource_gather_cmp`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty, llvm_any_ty, llvm_any_ty,`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty, llvm_any_ty, llvm_any_ty,`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_float_ty, llvm_i32_ty, llvm_any_ty],`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_float_ty, llvm_i32_ty, llvm_any_ty],`。
- **L175 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L175 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L177 EN**: Comment explains nearby logic, invariants, or intent: `Cast between target extension handle types and dxil-style opaque handles`.
  **L177 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Cast between target extension handle types and dxil-style opaque handles`。
- **L178 EN**: Declares TableGen def `int_dx_resource_casthandle`.
  **L178 CN**: 声明 TableGen def `int_dx_resource_casthandle`。
- **L179 EN**: Blank line separating nearby declarations or logic blocks.
  **L179 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L180 EN**: Declares TableGen def `int_dx_all`.
  **L180 CN**: 声明 TableGen def `int_dx_all`。

### Lines 181-200

````tablegen
def int_dx_any : DefaultAttrsIntrinsic<[llvm_i1_ty], [llvm_any_ty], [IntrNoMem]>;
def int_dx_asdouble : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_double_ty>], [llvm_anyint_ty, LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_uclamp : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;
def int_dx_sclamp : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;
def int_dx_nclamp : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;
def int_dx_cross : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;
def int_dx_saturate : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;

def int_dx_dot2 : DefaultAttrsIntrinsic<[LLVMMatchType<0>],
                                        [
                                          llvm_anyfloat_ty, LLVMMatchType<0>,
                                          LLVMMatchType<0>, LLVMMatchType<0>
                                        ],
                                        [IntrNoMem, Commutative]>;
def int_dx_dot3 : DefaultAttrsIntrinsic<[LLVMMatchType<0>],
                                        [
                                          llvm_anyfloat_ty, LLVMMatchType<0>,
                                          LLVMMatchType<0>, LLVMMatchType<0>,
                                          LLVMMatchType<0>, LLVMMatchType<0>
                                        ],
````
- **L181 EN**: Declares TableGen def `int_dx_any`.
  **L181 CN**: 声明 TableGen def `int_dx_any`。
- **L182 EN**: Declares TableGen def `int_dx_asdouble`.
  **L182 CN**: 声明 TableGen def `int_dx_asdouble`。
- **L183 EN**: Declares TableGen def `int_dx_uclamp`.
  **L183 CN**: 声明 TableGen def `int_dx_uclamp`。
- **L184 EN**: Declares TableGen def `int_dx_sclamp`.
  **L184 CN**: 声明 TableGen def `int_dx_sclamp`。
- **L185 EN**: Declares TableGen def `int_dx_nclamp`.
  **L185 CN**: 声明 TableGen def `int_dx_nclamp`。
- **L186 EN**: Declares TableGen def `int_dx_cross`.
  **L186 CN**: 声明 TableGen def `int_dx_cross`。
- **L187 EN**: Declares TableGen def `int_dx_saturate`.
  **L187 CN**: 声明 TableGen def `int_dx_saturate`。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L189 EN**: Declares TableGen def `int_dx_dot2`.
  **L189 CN**: 声明 TableGen def `int_dx_dot2`。
- **L190 EN**: Continues the surrounding expression or declaration: `[`.
  **L190 CN**: 继续构造周围的表达式或声明：`[`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyfloat_ty, LLVMMatchType<0>,`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyfloat_ty, LLVMMatchType<0>,`。
- **L192 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, LLVMMatchType<0>`.
  **L192 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, LLVMMatchType<0>`。
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L194 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L194 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L195 EN**: Declares TableGen def `int_dx_dot3`.
  **L195 CN**: 声明 TableGen def `int_dx_dot3`。
- **L196 EN**: Continues the surrounding expression or declaration: `[`.
  **L196 CN**: 继续构造周围的表达式或声明：`[`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyfloat_ty, LLVMMatchType<0>,`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyfloat_ty, LLVMMatchType<0>,`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L199 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, LLVMMatchType<0>`.
  **L199 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, LLVMMatchType<0>`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。

### Lines 201-220

````tablegen
                                        [IntrNoMem, Commutative]>;
def int_dx_dot4 : DefaultAttrsIntrinsic<[LLVMMatchType<0>],
                                        [
                                          llvm_anyfloat_ty, LLVMMatchType<0>,
                                          LLVMMatchType<0>, LLVMMatchType<0>,
                                          LLVMMatchType<0>, LLVMMatchType<0>,
                                          LLVMMatchType<0>, LLVMMatchType<0>
                                        ],
                                        [IntrNoMem, Commutative]>;
def int_dx_fdot :
    DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],
    [llvm_anyfloat_ty, LLVMScalarOrSameVectorWidth<0, LLVMVectorElementType<0>>],
    [IntrNoMem, Commutative] >;
def int_dx_sdot :
    DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],
    [llvm_anyint_ty, LLVMScalarOrSameVectorWidth<0, LLVMVectorElementType<0>>],
    [IntrNoMem, Commutative] >;
def int_dx_udot :
    DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],
    [llvm_anyint_ty, LLVMScalarOrSameVectorWidth<0, LLVMVectorElementType<0>>],
````
- **L201 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L201 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L202 EN**: Declares TableGen def `int_dx_dot4`.
  **L202 CN**: 声明 TableGen def `int_dx_dot4`。
- **L203 EN**: Continues the surrounding expression or declaration: `[`.
  **L203 CN**: 继续构造周围的表达式或声明：`[`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyfloat_ty, LLVMMatchType<0>,`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyfloat_ty, LLVMMatchType<0>,`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L207 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, LLVMMatchType<0>`.
  **L207 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, LLVMMatchType<0>`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L209 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L209 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L210 EN**: Declares TableGen def `int_dx_fdot`.
  **L210 CN**: 声明 TableGen def `int_dx_fdot`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyfloat_ty, LLVMScalarOrSameVectorWidth<0, LLVMVectorElementType<0>>],`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyfloat_ty, LLVMScalarOrSameVectorWidth<0, LLVMVectorElementType<0>>],`。
- **L213 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative] >;`.
  **L213 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative] >;`。
- **L214 EN**: Declares TableGen def `int_dx_sdot`.
  **L214 CN**: 声明 TableGen def `int_dx_sdot`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyint_ty, LLVMScalarOrSameVectorWidth<0, LLVMVectorElementType<0>>],`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyint_ty, LLVMScalarOrSameVectorWidth<0, LLVMVectorElementType<0>>],`。
- **L217 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative] >;`.
  **L217 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative] >;`。
- **L218 EN**: Declares TableGen def `int_dx_udot`.
  **L218 CN**: 声明 TableGen def `int_dx_udot`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyint_ty, LLVMScalarOrSameVectorWidth<0, LLVMVectorElementType<0>>],`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyint_ty, LLVMScalarOrSameVectorWidth<0, LLVMVectorElementType<0>>],`。

### Lines 221-240

````tablegen
    [IntrNoMem, Commutative] >;
def int_dx_dot2add :
    DefaultAttrsIntrinsic<[llvm_float_ty],
    [llvm_float_ty, llvm_half_ty, llvm_half_ty, llvm_half_ty, llvm_half_ty],
    [IntrNoMem, Commutative]>;
def int_dx_dot4add_i8packed : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_dx_dot4add_u8packed : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;

def int_dx_frac  : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_degrees : DefaultAttrsIntrinsic<[LLVMMatchType<0>], [llvm_anyfloat_ty], [IntrNoMem]>;

def int_dx_isinf : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
    [llvm_anyfloat_ty], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_isnan : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
    [llvm_anyfloat_ty], [IntrNoMem, IntrTriviallyScalarizable]>;

def int_dx_legacyf16tof32 : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_float_ty>],
    [llvm_anyint_ty], [IntrNoMem, IntrTriviallyScalarizable]>;

def int_dx_legacyf32tof16 : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>],
````
- **L221 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative] >;`.
  **L221 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative] >;`。
- **L222 EN**: Declares TableGen def `int_dx_dot2add`.
  **L222 CN**: 声明 TableGen def `int_dx_dot2add`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_half_ty, llvm_half_ty, llvm_half_ty, llvm_half_ty],`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_half_ty, llvm_half_ty, llvm_half_ty, llvm_half_ty],`。
- **L225 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L225 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L226 EN**: Declares TableGen def `int_dx_dot4add_i8packed`.
  **L226 CN**: 声明 TableGen def `int_dx_dot4add_i8packed`。
- **L227 EN**: Declares TableGen def `int_dx_dot4add_u8packed`.
  **L227 CN**: 声明 TableGen def `int_dx_dot4add_u8packed`。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L229 EN**: Declares TableGen def `int_dx_frac`.
  **L229 CN**: 声明 TableGen def `int_dx_frac`。
- **L230 EN**: Declares TableGen def `int_dx_degrees`.
  **L230 CN**: 声明 TableGen def `int_dx_degrees`。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L232 EN**: Declares TableGen def `int_dx_isinf`.
  **L232 CN**: 声明 TableGen def `int_dx_isinf`。
- **L233 EN**: Executes a standalone statement or declaration: `[llvm_anyfloat_ty], [IntrNoMem, IntrTriviallyScalarizable]>;`.
  **L233 CN**: 执行一条独立语句或声明：`[llvm_anyfloat_ty], [IntrNoMem, IntrTriviallyScalarizable]>;`。
- **L234 EN**: Declares TableGen def `int_dx_isnan`.
  **L234 CN**: 声明 TableGen def `int_dx_isnan`。
- **L235 EN**: Executes a standalone statement or declaration: `[llvm_anyfloat_ty], [IntrNoMem, IntrTriviallyScalarizable]>;`.
  **L235 CN**: 执行一条独立语句或声明：`[llvm_anyfloat_ty], [IntrNoMem, IntrTriviallyScalarizable]>;`。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L237 EN**: Declares TableGen def `int_dx_legacyf16tof32`.
  **L237 CN**: 声明 TableGen def `int_dx_legacyf16tof32`。
- **L238 EN**: Executes a standalone statement or declaration: `[llvm_anyint_ty], [IntrNoMem, IntrTriviallyScalarizable]>;`.
  **L238 CN**: 执行一条独立语句或声明：`[llvm_anyint_ty], [IntrNoMem, IntrTriviallyScalarizable]>;`。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L240 EN**: Declares TableGen def `int_dx_legacyf32tof16`.
  **L240 CN**: 声明 TableGen def `int_dx_legacyf32tof16`。

### Lines 241-260

````tablegen
    [llvm_anyfloat_ty], [IntrNoMem, IntrTriviallyScalarizable]>;

def int_dx_lerp : DefaultAttrsIntrinsic<[LLVMMatchType<0>], [llvm_anyfloat_ty, LLVMMatchType<0>,LLVMMatchType<0>],
    [IntrNoMem]>;

def int_dx_imad : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_umad : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_normalize : DefaultAttrsIntrinsic<[LLVMMatchType<0>], [llvm_anyfloat_ty], [IntrNoMem]>;
def int_dx_wave_prefix_bit_count : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i1_ty], [IntrConvergent, IntrNoMem]>;
def int_dx_rsqrt  : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_active_countbits : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i1_ty], [IntrConvergent, IntrNoMem]>;
def int_dx_wave_all_equal : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>], [llvm_any_ty], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_all : DefaultAttrsIntrinsic<[llvm_i1_ty], [llvm_i1_ty], [IntrConvergent, IntrNoMem]>;
def int_dx_wave_any : DefaultAttrsIntrinsic<[llvm_i1_ty], [llvm_i1_ty], [IntrConvergent, IntrNoMem]>;
def int_dx_wave_ballot : DefaultAttrsIntrinsic<[llvm_anyint_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>], [llvm_i1_ty], [IntrConvergent, IntrNoMem]>;
def int_dx_wave_getlaneindex : DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrConvergent, IntrNoMem]>;
def int_dx_wave_reduce_or : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_reduce_xor : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_reduce_and : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_reduce_max : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
````
- **L241 EN**: Executes a standalone statement or declaration: `[llvm_anyfloat_ty], [IntrNoMem, IntrTriviallyScalarizable]>;`.
  **L241 CN**: 执行一条独立语句或声明：`[llvm_anyfloat_ty], [IntrNoMem, IntrTriviallyScalarizable]>;`。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L243 EN**: Declares TableGen def `int_dx_lerp`.
  **L243 CN**: 声明 TableGen def `int_dx_lerp`。
- **L244 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L244 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L246 EN**: Declares TableGen def `int_dx_imad`.
  **L246 CN**: 声明 TableGen def `int_dx_imad`。
- **L247 EN**: Declares TableGen def `int_dx_umad`.
  **L247 CN**: 声明 TableGen def `int_dx_umad`。
- **L248 EN**: Declares TableGen def `int_dx_normalize`.
  **L248 CN**: 声明 TableGen def `int_dx_normalize`。
- **L249 EN**: Declares TableGen def `int_dx_wave_prefix_bit_count`.
  **L249 CN**: 声明 TableGen def `int_dx_wave_prefix_bit_count`。
- **L250 EN**: Declares TableGen def `int_dx_rsqrt`.
  **L250 CN**: 声明 TableGen def `int_dx_rsqrt`。
- **L251 EN**: Declares TableGen def `int_dx_wave_active_countbits`.
  **L251 CN**: 声明 TableGen def `int_dx_wave_active_countbits`。
- **L252 EN**: Declares TableGen def `int_dx_wave_all_equal`.
  **L252 CN**: 声明 TableGen def `int_dx_wave_all_equal`。
- **L253 EN**: Declares TableGen def `int_dx_wave_all`.
  **L253 CN**: 声明 TableGen def `int_dx_wave_all`。
- **L254 EN**: Declares TableGen def `int_dx_wave_any`.
  **L254 CN**: 声明 TableGen def `int_dx_wave_any`。
- **L255 EN**: Declares TableGen def `int_dx_wave_ballot`.
  **L255 CN**: 声明 TableGen def `int_dx_wave_ballot`。
- **L256 EN**: Declares TableGen def `int_dx_wave_getlaneindex`.
  **L256 CN**: 声明 TableGen def `int_dx_wave_getlaneindex`。
- **L257 EN**: Declares TableGen def `int_dx_wave_reduce_or`.
  **L257 CN**: 声明 TableGen def `int_dx_wave_reduce_or`。
- **L258 EN**: Declares TableGen def `int_dx_wave_reduce_xor`.
  **L258 CN**: 声明 TableGen def `int_dx_wave_reduce_xor`。
- **L259 EN**: Declares TableGen def `int_dx_wave_reduce_and`.
  **L259 CN**: 声明 TableGen def `int_dx_wave_reduce_and`。
- **L260 EN**: Declares TableGen def `int_dx_wave_reduce_max`.
  **L260 CN**: 声明 TableGen def `int_dx_wave_reduce_max`。

### Lines 261-280

````tablegen
def int_dx_wave_reduce_umax : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_reduce_min : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_reduce_umin : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_reduce_sum : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_reduce_usum : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_product : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_uproduct : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_is_first_lane : DefaultAttrsIntrinsic<[llvm_i1_ty], [], [IntrConvergent]>;
def int_dx_wave_readlane : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>, llvm_i32_ty], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_get_lane_count
    : DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrConvergent]>;
def int_dx_wave_prefix_sum : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_prefix_usum : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_prefix_product : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_wave_prefix_uproduct : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_quad_read_across_x : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_quad_read_across_y : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrConvergent, IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_sign : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>], [llvm_any_ty], [IntrNoMem]>;
def int_dx_step : DefaultAttrsIntrinsic<[LLVMMatchType<0>], [llvm_anyfloat_ty, LLVMMatchType<0>], [IntrNoMem]>;
def int_dx_splitdouble : DefaultAttrsIntrinsic<[llvm_anyint_ty, LLVMMatchType<0>],
````
- **L261 EN**: Declares TableGen def `int_dx_wave_reduce_umax`.
  **L261 CN**: 声明 TableGen def `int_dx_wave_reduce_umax`。
- **L262 EN**: Declares TableGen def `int_dx_wave_reduce_min`.
  **L262 CN**: 声明 TableGen def `int_dx_wave_reduce_min`。
- **L263 EN**: Declares TableGen def `int_dx_wave_reduce_umin`.
  **L263 CN**: 声明 TableGen def `int_dx_wave_reduce_umin`。
- **L264 EN**: Declares TableGen def `int_dx_wave_reduce_sum`.
  **L264 CN**: 声明 TableGen def `int_dx_wave_reduce_sum`。
- **L265 EN**: Declares TableGen def `int_dx_wave_reduce_usum`.
  **L265 CN**: 声明 TableGen def `int_dx_wave_reduce_usum`。
- **L266 EN**: Declares TableGen def `int_dx_wave_product`.
  **L266 CN**: 声明 TableGen def `int_dx_wave_product`。
- **L267 EN**: Declares TableGen def `int_dx_wave_uproduct`.
  **L267 CN**: 声明 TableGen def `int_dx_wave_uproduct`。
- **L268 EN**: Declares TableGen def `int_dx_wave_is_first_lane`.
  **L268 CN**: 声明 TableGen def `int_dx_wave_is_first_lane`。
- **L269 EN**: Declares TableGen def `int_dx_wave_readlane`.
  **L269 CN**: 声明 TableGen def `int_dx_wave_readlane`。
- **L270 EN**: Declares TableGen def `int_dx_wave_get_lane_count`.
  **L270 CN**: 声明 TableGen def `int_dx_wave_get_lane_count`。
- **L271 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrConvergent]>;`.
  **L271 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrConvergent]>;`。
- **L272 EN**: Declares TableGen def `int_dx_wave_prefix_sum`.
  **L272 CN**: 声明 TableGen def `int_dx_wave_prefix_sum`。
- **L273 EN**: Declares TableGen def `int_dx_wave_prefix_usum`.
  **L273 CN**: 声明 TableGen def `int_dx_wave_prefix_usum`。
- **L274 EN**: Declares TableGen def `int_dx_wave_prefix_product`.
  **L274 CN**: 声明 TableGen def `int_dx_wave_prefix_product`。
- **L275 EN**: Declares TableGen def `int_dx_wave_prefix_uproduct`.
  **L275 CN**: 声明 TableGen def `int_dx_wave_prefix_uproduct`。
- **L276 EN**: Declares TableGen def `int_dx_quad_read_across_x`.
  **L276 CN**: 声明 TableGen def `int_dx_quad_read_across_x`。
- **L277 EN**: Declares TableGen def `int_dx_quad_read_across_y`.
  **L277 CN**: 声明 TableGen def `int_dx_quad_read_across_y`。
- **L278 EN**: Declares TableGen def `int_dx_sign`.
  **L278 CN**: 声明 TableGen def `int_dx_sign`。
- **L279 EN**: Declares TableGen def `int_dx_step`.
  **L279 CN**: 声明 TableGen def `int_dx_step`。
- **L280 EN**: Declares TableGen def `int_dx_splitdouble`.
  **L280 CN**: 声明 TableGen def `int_dx_splitdouble`。

### Lines 281-300

````tablegen
    [LLVMScalarOrSameVectorWidth<0, llvm_double_ty>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_radians : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem]>;
def int_dx_discard : DefaultAttrsIntrinsic<[], [llvm_i1_ty], []>;
def int_dx_ddx_coarse : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_ddy_coarse : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_ddx_fine : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_ddy_fine : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_firstbituhigh : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>], [llvm_anyint_ty], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_firstbitshigh : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>], [llvm_anyint_ty], [IntrNoMem, IntrTriviallyScalarizable]>;
def int_dx_firstbitlow : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>], [llvm_anyint_ty], [IntrNoMem, IntrTriviallyScalarizable]>;

def int_dx_all_memory_barrier
    : DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;

def int_dx_all_memory_barrier_with_group_sync
    : DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;

def int_dx_device_memory_barrier
    : DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;

````
- **L281 EN**: Executes a standalone statement or declaration: `[LLVMScalarOrSameVectorWidth<0, llvm_double_ty>], [IntrNoMem, IntrTriviallyScalarizable]>;`.
  **L281 CN**: 执行一条独立语句或声明：`[LLVMScalarOrSameVectorWidth<0, llvm_double_ty>], [IntrNoMem, IntrTriviallyScalarizable]>;`。
- **L282 EN**: Declares TableGen def `int_dx_radians`.
  **L282 CN**: 声明 TableGen def `int_dx_radians`。
- **L283 EN**: Declares TableGen def `int_dx_discard`.
  **L283 CN**: 声明 TableGen def `int_dx_discard`。
- **L284 EN**: Declares TableGen def `int_dx_ddx_coarse`.
  **L284 CN**: 声明 TableGen def `int_dx_ddx_coarse`。
- **L285 EN**: Declares TableGen def `int_dx_ddy_coarse`.
  **L285 CN**: 声明 TableGen def `int_dx_ddy_coarse`。
- **L286 EN**: Declares TableGen def `int_dx_ddx_fine`.
  **L286 CN**: 声明 TableGen def `int_dx_ddx_fine`。
- **L287 EN**: Declares TableGen def `int_dx_ddy_fine`.
  **L287 CN**: 声明 TableGen def `int_dx_ddy_fine`。
- **L288 EN**: Declares TableGen def `int_dx_firstbituhigh`.
  **L288 CN**: 声明 TableGen def `int_dx_firstbituhigh`。
- **L289 EN**: Declares TableGen def `int_dx_firstbitshigh`.
  **L289 CN**: 声明 TableGen def `int_dx_firstbitshigh`。
- **L290 EN**: Declares TableGen def `int_dx_firstbitlow`.
  **L290 CN**: 声明 TableGen def `int_dx_firstbitlow`。
- **L291 EN**: Blank line separating nearby declarations or logic blocks.
  **L291 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L292 EN**: Declares TableGen def `int_dx_all_memory_barrier`.
  **L292 CN**: 声明 TableGen def `int_dx_all_memory_barrier`。
- **L293 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`.
  **L293 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L295 EN**: Declares TableGen def `int_dx_all_memory_barrier_with_group_sync`.
  **L295 CN**: 声明 TableGen def `int_dx_all_memory_barrier_with_group_sync`。
- **L296 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`.
  **L296 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L298 EN**: Declares TableGen def `int_dx_device_memory_barrier`.
  **L298 CN**: 声明 TableGen def `int_dx_device_memory_barrier`。
- **L299 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`.
  **L299 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`。
- **L300 EN**: Blank line separating nearby declarations or logic blocks.
  **L300 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 301-320

````tablegen
def int_dx_device_memory_barrier_with_group_sync
    : DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;

def int_dx_group_memory_barrier
    : DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;

def int_dx_group_memory_barrier_with_group_sync
    : DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;

def int_dx_load_input
    : DefaultAttrsIntrinsic<[llvm_any_ty],
                            [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i8_ty,
                             llvm_i32_ty],
                            [IntrConvergent]>;

def int_dx_store_output
    : DefaultAttrsIntrinsic<[],
                            [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i8_ty,
                             llvm_i32_ty, llvm_any_ty],
                            [IntrConvergent]>;
````
- **L301 EN**: Declares TableGen def `int_dx_device_memory_barrier_with_group_sync`.
  **L301 CN**: 声明 TableGen def `int_dx_device_memory_barrier_with_group_sync`。
- **L302 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`.
  **L302 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`。
- **L303 EN**: Blank line separating nearby declarations or logic blocks.
  **L303 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L304 EN**: Declares TableGen def `int_dx_group_memory_barrier`.
  **L304 CN**: 声明 TableGen def `int_dx_group_memory_barrier`。
- **L305 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`.
  **L305 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`。
- **L306 EN**: Blank line separating nearby declarations or logic blocks.
  **L306 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L307 EN**: Declares TableGen def `int_dx_group_memory_barrier_with_group_sync`.
  **L307 CN**: 声明 TableGen def `int_dx_group_memory_barrier_with_group_sync`。
- **L308 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`.
  **L308 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [], [IntrConvergent]>;`。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L310 EN**: Declares TableGen def `int_dx_load_input`.
  **L310 CN**: 声明 TableGen def `int_dx_load_input`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_any_ty],`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_any_ty],`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i8_ty,`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i8_ty,`。
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L314 EN**: Executes a standalone statement or declaration: `[IntrConvergent]>;`.
  **L314 CN**: 执行一条独立语句或声明：`[IntrConvergent]>;`。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L316 EN**: Declares TableGen def `int_dx_store_output`.
  **L316 CN**: 声明 TableGen def `int_dx_store_output`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i8_ty,`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i8_ty,`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_any_ty],`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_any_ty],`。
- **L320 EN**: Executes a standalone statement or declaration: `[IntrConvergent]>;`.
  **L320 CN**: 执行一条独立语句或声明：`[IntrConvergent]>;`。

### Lines 321-321

````tablegen
}
````
- **L321 EN**: Closes the current lexical scope or compound statement.
  **L321 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **Function-level IR management / 函数级 IR 管理**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
