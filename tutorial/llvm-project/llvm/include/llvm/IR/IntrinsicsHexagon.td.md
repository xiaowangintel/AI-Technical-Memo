# IntrinsicsHexagon.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsHexagon.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the Hexagon-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsHexagon` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````tablegen
//===- IntrinsicsHexagon.td - Defines Hexagon intrinsics ---*- tablegen -*-===//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the Hexagon-specific intrinsics.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Definitions for all Hexagon intrinsics.
//
// All Hexagon intrinsics start with "llvm.hexagon.".
let TargetPrefix = "hexagon" in {
  /// Hexagon_Intrinsic - Base class for the majority of Hexagon intrinsics.
  class Hexagon_Intrinsic<string GCCIntSuffix, list<LLVMType> ret_types,
                              list<LLVMType> param_types,
                              list<IntrinsicProperty> properties>
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L2 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L5 EN**: Separator comment used for visual grouping.
  **L5 CN**: 用于视觉分组的分隔注释。
- **L6 EN**: Banner comment marking a file or section boundary.
  **L6 CN**: 横幅注释，用于标记文件或章节边界。
- **L7 EN**: Separator comment used for visual grouping.
  **L7 CN**: 用于视觉分组的分隔注释。
- **L8 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the Hexagon-specific intrinsics.`.
  **L8 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the Hexagon-specific intrinsics.`。
- **L9 EN**: Separator comment used for visual grouping.
  **L9 CN**: 用于视觉分组的分隔注释。
- **L10 EN**: Banner comment marking a file or section boundary.
  **L10 CN**: 横幅注释，用于标记文件或章节边界。
- **L11 EN**: Blank line separating nearby declarations or logic blocks.
  **L11 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L12 EN**: Banner comment marking a file or section boundary.
  **L12 CN**: 横幅注释，用于标记文件或章节边界。
- **L13 EN**: Comment explains nearby logic, invariants, or intent: `Definitions for all Hexagon intrinsics.`.
  **L13 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Definitions for all Hexagon intrinsics.`。
- **L14 EN**: Separator comment used for visual grouping.
  **L14 CN**: 用于视觉分组的分隔注释。
- **L15 EN**: Comment explains nearby logic, invariants, or intent: `All Hexagon intrinsics start with "llvm.hexagon.".`.
  **L15 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All Hexagon intrinsics start with "llvm.hexagon.".`。
- **L16 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L16 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L17 EN**: Comment explains nearby logic, invariants, or intent: `Hexagon_Intrinsic - Base class for the majority of Hexagon intrinsics.`.
  **L17 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Hexagon_Intrinsic - Base class for the majority of Hexagon intrinsics.`。
- **L18 EN**: Declares class `Hexagon_Intrinsic<string`.
  **L18 CN**: 声明 class `Hexagon_Intrinsic<string`。
- **L19 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> param_types,`.
  **L19 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> param_types,`。
- **L20 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> properties>`.
  **L20 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> properties>`。

### Lines 21-40

````tablegen
    : ClangBuiltin<!strconcat("__builtin_", GCCIntSuffix)>,
      DefaultAttrsIntrinsic<ret_types, param_types, properties>;

  /// Hexagon_NonGCC_Intrinsic - Base class for bitcode convertible Hexagon
  /// intrinsics.
  class Hexagon_NonGCC_Intrinsic<list<LLVMType> ret_types,
                                 list<LLVMType> param_types,
                                 list<IntrinsicProperty> properties>
    : DefaultAttrsIntrinsic<ret_types, param_types, properties>;
}

class Hexagon_mem_memmemsi_Intrinsic<string GCCIntSuffix>
  : Hexagon_Intrinsic<GCCIntSuffix,
                          [llvm_ptr_ty], [llvm_ptr_ty, llvm_ptr_ty,
                           llvm_i32_ty],
                          [IntrArgMemOnly]>;

class Hexagon_mem_memsisi_Intrinsic<string GCCIntSuffix>
  : Hexagon_Intrinsic<GCCIntSuffix,
                          [llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty,
````
- **L21 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<!strconcat("__builtin_", GCCIntSuffix)>,`.
  **L21 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<!strconcat("__builtin_", GCCIntSuffix)>,`。
- **L22 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<ret_types, param_types, properties>;`.
  **L22 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<ret_types, param_types, properties>;`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Comment explains nearby logic, invariants, or intent: `Hexagon_NonGCC_Intrinsic - Base class for bitcode convertible Hexagon`.
  **L24 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Hexagon_NonGCC_Intrinsic - Base class for bitcode convertible Hexagon`。
- **L25 EN**: Comment explains nearby logic, invariants, or intent: `intrinsics.`.
  **L25 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`intrinsics.`。
- **L26 EN**: Declares class `Hexagon_NonGCC_Intrinsic<list<LLVMType>`.
  **L26 CN**: 声明 class `Hexagon_NonGCC_Intrinsic<list<LLVMType>`。
- **L27 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> param_types,`.
  **L27 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> param_types,`。
- **L28 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> properties>`.
  **L28 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> properties>`。
- **L29 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<ret_types, param_types, properties>;`.
  **L29 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<ret_types, param_types, properties>;`。
- **L30 EN**: Closes the current lexical scope or compound statement.
  **L30 CN**: 结束当前词法作用域或复合语句块。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L32 EN**: Declares class `Hexagon_mem_memmemsi_Intrinsic<string`.
  **L32 CN**: 声明 class `Hexagon_mem_memmemsi_Intrinsic<string`。
- **L33 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L33 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L34 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty], [llvm_ptr_ty, llvm_ptr_ty,`.
  **L34 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty], [llvm_ptr_ty, llvm_ptr_ty,`。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L36 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly]>;`.
  **L36 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly]>;`。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Declares class `Hexagon_mem_memsisi_Intrinsic<string`.
  **L38 CN**: 声明 class `Hexagon_mem_memsisi_Intrinsic<string`。
- **L39 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L39 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L40 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty,`.
  **L40 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty,`。

### Lines 41-60

````tablegen
                           llvm_i32_ty],
                          [IntrWriteMem]>;

class Hexagon_mem_memdisi_Intrinsic<string GCCIntSuffix>
  : Hexagon_Intrinsic<GCCIntSuffix,
                          [llvm_ptr_ty], [llvm_ptr_ty, llvm_i64_ty,
                           llvm_i32_ty],
                          [IntrWriteMem]>;

class Hexagon_mem_memmemsisi_Intrinsic<string GCCIntSuffix>
  : Hexagon_Intrinsic<GCCIntSuffix,
                          [llvm_ptr_ty], [llvm_ptr_ty, llvm_ptr_ty,
                           llvm_i32_ty, llvm_i32_ty],
                          [IntrArgMemOnly, ImmArg<ArgIndex<3>>]>;

class Hexagon_mem_memsisisi_Intrinsic<string GCCIntSuffix>
  : Hexagon_Intrinsic<GCCIntSuffix,
                          [llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty,
                           llvm_i32_ty, llvm_i32_ty],
                          [IntrWriteMem, ImmArg<ArgIndex<3>>]>;
````
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L42 EN**: Executes a standalone statement or declaration: `[IntrWriteMem]>;`.
  **L42 CN**: 执行一条独立语句或声明：`[IntrWriteMem]>;`。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Declares class `Hexagon_mem_memdisi_Intrinsic<string`.
  **L44 CN**: 声明 class `Hexagon_mem_memdisi_Intrinsic<string`。
- **L45 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L45 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L46 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty], [llvm_ptr_ty, llvm_i64_ty,`.
  **L46 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty], [llvm_ptr_ty, llvm_i64_ty,`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L48 EN**: Executes a standalone statement or declaration: `[IntrWriteMem]>;`.
  **L48 CN**: 执行一条独立语句或声明：`[IntrWriteMem]>;`。
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Declares class `Hexagon_mem_memmemsisi_Intrinsic<string`.
  **L50 CN**: 声明 class `Hexagon_mem_memmemsisi_Intrinsic<string`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L52 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty], [llvm_ptr_ty, llvm_ptr_ty,`.
  **L52 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty], [llvm_ptr_ty, llvm_ptr_ty,`。
- **L53 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L53 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。
- **L54 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, ImmArg<ArgIndex<3>>]>;`.
  **L54 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, ImmArg<ArgIndex<3>>]>;`。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L56 EN**: Declares class `Hexagon_mem_memsisisi_Intrinsic<string`.
  **L56 CN**: 声明 class `Hexagon_mem_memsisisi_Intrinsic<string`。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty,`.
  **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty,`。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。
- **L60 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, ImmArg<ArgIndex<3>>]>;`.
  **L60 CN**: 执行一条独立语句或声明：`[IntrWriteMem, ImmArg<ArgIndex<3>>]>;`。

### Lines 61-80

````tablegen

class Hexagon_mem_memdisisi_Intrinsic<string GCCIntSuffix>
  : Hexagon_Intrinsic<GCCIntSuffix,
                          [llvm_ptr_ty], [llvm_ptr_ty, llvm_i64_ty,
                           llvm_i32_ty, llvm_i32_ty],
                          [IntrWriteMem, ImmArg<ArgIndex<3>>]>;

//
// BUILTIN_INFO_NONCONST(circ_ldd,PTR_ftype_PTRPTRSISI,4)
//
def int_hexagon_circ_ldd :
Hexagon_mem_memmemsisi_Intrinsic<"circ_ldd">;
//
// BUILTIN_INFO_NONCONST(circ_ldw,PTR_ftype_PTRPTRSISI,4)
//
def int_hexagon_circ_ldw :
Hexagon_mem_memmemsisi_Intrinsic<"circ_ldw">;
//
// BUILTIN_INFO_NONCONST(circ_ldh,PTR_ftype_PTRPTRSISI,4)
//
````
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L62 EN**: Declares class `Hexagon_mem_memdisisi_Intrinsic<string`.
  **L62 CN**: 声明 class `Hexagon_mem_memdisisi_Intrinsic<string`。
- **L63 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L63 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty], [llvm_ptr_ty, llvm_i64_ty,`.
  **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty], [llvm_ptr_ty, llvm_i64_ty,`。
- **L65 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L65 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。
- **L66 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, ImmArg<ArgIndex<3>>]>;`.
  **L66 CN**: 执行一条独立语句或声明：`[IntrWriteMem, ImmArg<ArgIndex<3>>]>;`。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L68 EN**: Separator comment used for visual grouping.
  **L68 CN**: 用于视觉分组的分隔注释。
- **L69 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_ldd,PTR_ftype_PTRPTRSISI,4)`.
  **L69 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_ldd,PTR_ftype_PTRPTRSISI,4)`。
- **L70 EN**: Separator comment used for visual grouping.
  **L70 CN**: 用于视觉分组的分隔注释。
- **L71 EN**: Declares TableGen def `int_hexagon_circ_ldd`.
  **L71 CN**: 声明 TableGen def `int_hexagon_circ_ldd`。
- **L72 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memmemsisi_Intrinsic<"circ_ldd">;`.
  **L72 CN**: 执行一条独立语句或声明：`Hexagon_mem_memmemsisi_Intrinsic<"circ_ldd">;`。
- **L73 EN**: Separator comment used for visual grouping.
  **L73 CN**: 用于视觉分组的分隔注释。
- **L74 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_ldw,PTR_ftype_PTRPTRSISI,4)`.
  **L74 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_ldw,PTR_ftype_PTRPTRSISI,4)`。
- **L75 EN**: Separator comment used for visual grouping.
  **L75 CN**: 用于视觉分组的分隔注释。
- **L76 EN**: Declares TableGen def `int_hexagon_circ_ldw`.
  **L76 CN**: 声明 TableGen def `int_hexagon_circ_ldw`。
- **L77 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memmemsisi_Intrinsic<"circ_ldw">;`.
  **L77 CN**: 执行一条独立语句或声明：`Hexagon_mem_memmemsisi_Intrinsic<"circ_ldw">;`。
- **L78 EN**: Separator comment used for visual grouping.
  **L78 CN**: 用于视觉分组的分隔注释。
- **L79 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_ldh,PTR_ftype_PTRPTRSISI,4)`.
  **L79 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_ldh,PTR_ftype_PTRPTRSISI,4)`。
- **L80 EN**: Separator comment used for visual grouping.
  **L80 CN**: 用于视觉分组的分隔注释。

### Lines 81-100

````tablegen
def int_hexagon_circ_ldh :
Hexagon_mem_memmemsisi_Intrinsic<"circ_ldh">;
//
// BUILTIN_INFO_NONCONST(circ_lduh,PTR_ftype_PTRPTRSISI,4)
//
def int_hexagon_circ_lduh :
Hexagon_mem_memmemsisi_Intrinsic<"circ_lduh">;
//
// BUILTIN_INFO_NONCONST(circ_ldb,PTR_ftype_PTRPTRSISI,4)
//
def int_hexagon_circ_ldb :
Hexagon_mem_memmemsisi_Intrinsic<"circ_ldb">;
//
// BUILTIN_INFO_NONCONST(circ_ldub,PTR_ftype_PTRPTRSISI,4)
//
def int_hexagon_circ_ldub :
Hexagon_mem_memmemsisi_Intrinsic<"circ_ldub">;

//
// BUILTIN_INFO_NONCONST(circ_std,PTR_ftype_PTRDISISI,4)
````
- **L81 EN**: Declares TableGen def `int_hexagon_circ_ldh`.
  **L81 CN**: 声明 TableGen def `int_hexagon_circ_ldh`。
- **L82 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memmemsisi_Intrinsic<"circ_ldh">;`.
  **L82 CN**: 执行一条独立语句或声明：`Hexagon_mem_memmemsisi_Intrinsic<"circ_ldh">;`。
- **L83 EN**: Separator comment used for visual grouping.
  **L83 CN**: 用于视觉分组的分隔注释。
- **L84 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_lduh,PTR_ftype_PTRPTRSISI,4)`.
  **L84 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_lduh,PTR_ftype_PTRPTRSISI,4)`。
- **L85 EN**: Separator comment used for visual grouping.
  **L85 CN**: 用于视觉分组的分隔注释。
- **L86 EN**: Declares TableGen def `int_hexagon_circ_lduh`.
  **L86 CN**: 声明 TableGen def `int_hexagon_circ_lduh`。
- **L87 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memmemsisi_Intrinsic<"circ_lduh">;`.
  **L87 CN**: 执行一条独立语句或声明：`Hexagon_mem_memmemsisi_Intrinsic<"circ_lduh">;`。
- **L88 EN**: Separator comment used for visual grouping.
  **L88 CN**: 用于视觉分组的分隔注释。
- **L89 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_ldb,PTR_ftype_PTRPTRSISI,4)`.
  **L89 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_ldb,PTR_ftype_PTRPTRSISI,4)`。
- **L90 EN**: Separator comment used for visual grouping.
  **L90 CN**: 用于视觉分组的分隔注释。
- **L91 EN**: Declares TableGen def `int_hexagon_circ_ldb`.
  **L91 CN**: 声明 TableGen def `int_hexagon_circ_ldb`。
- **L92 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memmemsisi_Intrinsic<"circ_ldb">;`.
  **L92 CN**: 执行一条独立语句或声明：`Hexagon_mem_memmemsisi_Intrinsic<"circ_ldb">;`。
- **L93 EN**: Separator comment used for visual grouping.
  **L93 CN**: 用于视觉分组的分隔注释。
- **L94 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_ldub,PTR_ftype_PTRPTRSISI,4)`.
  **L94 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_ldub,PTR_ftype_PTRPTRSISI,4)`。
- **L95 EN**: Separator comment used for visual grouping.
  **L95 CN**: 用于视觉分组的分隔注释。
- **L96 EN**: Declares TableGen def `int_hexagon_circ_ldub`.
  **L96 CN**: 声明 TableGen def `int_hexagon_circ_ldub`。
- **L97 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memmemsisi_Intrinsic<"circ_ldub">;`.
  **L97 CN**: 执行一条独立语句或声明：`Hexagon_mem_memmemsisi_Intrinsic<"circ_ldub">;`。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L99 EN**: Separator comment used for visual grouping.
  **L99 CN**: 用于视觉分组的分隔注释。
- **L100 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_std,PTR_ftype_PTRDISISI,4)`.
  **L100 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_std,PTR_ftype_PTRDISISI,4)`。

### Lines 101-120

````tablegen
//
def int_hexagon_circ_std :
Hexagon_mem_memdisisi_Intrinsic<"circ_std">;
//
// BUILTIN_INFO_NONCONST(circ_stw,PTR_ftype_PTRSISISI,4)
//
def int_hexagon_circ_stw :
Hexagon_mem_memsisisi_Intrinsic<"circ_stw">;
//
// BUILTIN_INFO_NONCONST(circ_sth,PTR_ftype_PTRSISISI,4)
//
def int_hexagon_circ_sth :
Hexagon_mem_memsisisi_Intrinsic<"circ_sth">;
//
// BUILTIN_INFO_NONCONST(circ_sthhi,PTR_ftype_PTRSISISI,4)
//
def int_hexagon_circ_sthhi :
Hexagon_mem_memsisisi_Intrinsic<"circ_sthhi">;
//
// BUILTIN_INFO_NONCONST(circ_stb,PTR_ftype_PTRSISISI,4)
````
- **L101 EN**: Separator comment used for visual grouping.
  **L101 CN**: 用于视觉分组的分隔注释。
- **L102 EN**: Declares TableGen def `int_hexagon_circ_std`.
  **L102 CN**: 声明 TableGen def `int_hexagon_circ_std`。
- **L103 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memdisisi_Intrinsic<"circ_std">;`.
  **L103 CN**: 执行一条独立语句或声明：`Hexagon_mem_memdisisi_Intrinsic<"circ_std">;`。
- **L104 EN**: Separator comment used for visual grouping.
  **L104 CN**: 用于视觉分组的分隔注释。
- **L105 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_stw,PTR_ftype_PTRSISISI,4)`.
  **L105 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_stw,PTR_ftype_PTRSISISI,4)`。
- **L106 EN**: Separator comment used for visual grouping.
  **L106 CN**: 用于视觉分组的分隔注释。
- **L107 EN**: Declares TableGen def `int_hexagon_circ_stw`.
  **L107 CN**: 声明 TableGen def `int_hexagon_circ_stw`。
- **L108 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memsisisi_Intrinsic<"circ_stw">;`.
  **L108 CN**: 执行一条独立语句或声明：`Hexagon_mem_memsisisi_Intrinsic<"circ_stw">;`。
- **L109 EN**: Separator comment used for visual grouping.
  **L109 CN**: 用于视觉分组的分隔注释。
- **L110 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_sth,PTR_ftype_PTRSISISI,4)`.
  **L110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_sth,PTR_ftype_PTRSISISI,4)`。
- **L111 EN**: Separator comment used for visual grouping.
  **L111 CN**: 用于视觉分组的分隔注释。
- **L112 EN**: Declares TableGen def `int_hexagon_circ_sth`.
  **L112 CN**: 声明 TableGen def `int_hexagon_circ_sth`。
- **L113 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memsisisi_Intrinsic<"circ_sth">;`.
  **L113 CN**: 执行一条独立语句或声明：`Hexagon_mem_memsisisi_Intrinsic<"circ_sth">;`。
- **L114 EN**: Separator comment used for visual grouping.
  **L114 CN**: 用于视觉分组的分隔注释。
- **L115 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_sthhi,PTR_ftype_PTRSISISI,4)`.
  **L115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_sthhi,PTR_ftype_PTRSISISI,4)`。
- **L116 EN**: Separator comment used for visual grouping.
  **L116 CN**: 用于视觉分组的分隔注释。
- **L117 EN**: Declares TableGen def `int_hexagon_circ_sthhi`.
  **L117 CN**: 声明 TableGen def `int_hexagon_circ_sthhi`。
- **L118 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memsisisi_Intrinsic<"circ_sthhi">;`.
  **L118 CN**: 执行一条独立语句或声明：`Hexagon_mem_memsisisi_Intrinsic<"circ_sthhi">;`。
- **L119 EN**: Separator comment used for visual grouping.
  **L119 CN**: 用于视觉分组的分隔注释。
- **L120 EN**: Comment explains nearby logic, invariants, or intent: `BUILTIN_INFO_NONCONST(circ_stb,PTR_ftype_PTRSISISI,4)`.
  **L120 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BUILTIN_INFO_NONCONST(circ_stb,PTR_ftype_PTRSISISI,4)`。

### Lines 121-140

````tablegen
//
def int_hexagon_circ_stb :
Hexagon_mem_memsisisi_Intrinsic<"circ_stb">;

def int_hexagon_prefetch :
Hexagon_Intrinsic<"HEXAGON_prefetch", [], [llvm_ptr_ty], []>;

// Mark locked loads as read/write to prevent any accidental reordering.
// These don't use Hexagon_Intrinsic, because they are not nosync, and as such
// cannot use default attributes.
let TargetPrefix = "hexagon" in {
  def int_hexagon_L2_loadw_locked :
  ClangBuiltin<"__builtin_HEXAGON_L2_loadw_locked">,
  Intrinsic<[llvm_i32_ty], [llvm_ptr_ty],
        [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
  def int_hexagon_L4_loadd_locked :
  ClangBuiltin<"__builtin__HEXAGON_L4_loadd_locked">,
  Intrinsic<[llvm_i64_ty], [llvm_ptr_ty],
        [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;

````
- **L121 EN**: Separator comment used for visual grouping.
  **L121 CN**: 用于视觉分组的分隔注释。
- **L122 EN**: Declares TableGen def `int_hexagon_circ_stb`.
  **L122 CN**: 声明 TableGen def `int_hexagon_circ_stb`。
- **L123 EN**: Executes a standalone statement or declaration: `Hexagon_mem_memsisisi_Intrinsic<"circ_stb">;`.
  **L123 CN**: 执行一条独立语句或声明：`Hexagon_mem_memsisisi_Intrinsic<"circ_stb">;`。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L125 EN**: Declares TableGen def `int_hexagon_prefetch`.
  **L125 CN**: 声明 TableGen def `int_hexagon_prefetch`。
- **L126 EN**: Executes a standalone statement or declaration: `Hexagon_Intrinsic<"HEXAGON_prefetch", [], [llvm_ptr_ty], []>;`.
  **L126 CN**: 执行一条独立语句或声明：`Hexagon_Intrinsic<"HEXAGON_prefetch", [], [llvm_ptr_ty], []>;`。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L128 EN**: Comment explains nearby logic, invariants, or intent: `Mark locked loads as read/write to prevent any accidental reordering.`.
  **L128 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Mark locked loads as read/write to prevent any accidental reordering.`。
- **L129 EN**: Comment explains nearby logic, invariants, or intent: `These don't use Hexagon_Intrinsic, because they are not nosync, and as such`.
  **L129 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These don't use Hexagon_Intrinsic, because they are not nosync, and as such`。
- **L130 EN**: Comment explains nearby logic, invariants, or intent: `cannot use default attributes.`.
  **L130 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`cannot use default attributes.`。
- **L131 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L131 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L132 EN**: Declares TableGen def `int_hexagon_L2_loadw_locked`.
  **L132 CN**: 声明 TableGen def `int_hexagon_L2_loadw_locked`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_HEXAGON_L2_loadw_locked">,`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_HEXAGON_L2_loadw_locked">,`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty],`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty],`。
- **L135 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L135 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L136 EN**: Declares TableGen def `int_hexagon_L4_loadd_locked`.
  **L136 CN**: 声明 TableGen def `int_hexagon_L4_loadd_locked`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin__HEXAGON_L4_loadd_locked">,`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin__HEXAGON_L4_loadd_locked">,`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_ptr_ty],`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_ptr_ty],`。
- **L139 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L139 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 141-160

````tablegen
  def int_hexagon_S2_storew_locked :
  ClangBuiltin<"__builtin_HEXAGON_S2_storew_locked">,
  Intrinsic<[llvm_i32_ty],
        [llvm_ptr_ty, llvm_i32_ty], [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
  def int_hexagon_S4_stored_locked :
  ClangBuiltin<"__builtin_HEXAGON_S4_stored_locked">,
  Intrinsic<[llvm_i32_ty],
        [llvm_ptr_ty, llvm_i64_ty], [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
}

def int_hexagon_vmemcpy : Hexagon_Intrinsic<"hexagon_vmemcpy",
    [], [llvm_ptr_ty, llvm_ptr_ty, llvm_i32_ty],
    [IntrArgMemOnly, NoCapture<ArgIndex<0>>, NoCapture<ArgIndex<1>>, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;

def int_hexagon_vmemset : Hexagon_Intrinsic<"hexagon_vmemset",
    [], [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
    [IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;

multiclass Hexagon_custom_circ_ld_Intrinsic<LLVMType ElTy> {
  def NAME#_pci : Hexagon_NonGCC_Intrinsic<
````
- **L141 EN**: Declares TableGen def `int_hexagon_S2_storew_locked`.
  **L141 CN**: 声明 TableGen def `int_hexagon_S2_storew_locked`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_HEXAGON_S2_storew_locked">,`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_HEXAGON_S2_storew_locked">,`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty],`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty],`。
- **L144 EN**: Executes a standalone statement or declaration: `[llvm_ptr_ty, llvm_i32_ty], [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L144 CN**: 执行一条独立语句或声明：`[llvm_ptr_ty, llvm_i32_ty], [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L145 EN**: Declares TableGen def `int_hexagon_S4_stored_locked`.
  **L145 CN**: 声明 TableGen def `int_hexagon_S4_stored_locked`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_HEXAGON_S4_stored_locked">,`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_HEXAGON_S4_stored_locked">,`。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty],`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty],`。
- **L148 EN**: Executes a standalone statement or declaration: `[llvm_ptr_ty, llvm_i64_ty], [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L148 CN**: 执行一条独立语句或声明：`[llvm_ptr_ty, llvm_i64_ty], [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L149 EN**: Closes the current lexical scope or compound statement.
  **L149 CN**: 结束当前词法作用域或复合语句块。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L151 EN**: Declares TableGen def `int_hexagon_vmemcpy`.
  **L151 CN**: 声明 TableGen def `int_hexagon_vmemcpy`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_ptr_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_ptr_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L153 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>, NoCapture<ArgIndex<1>>, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`.
  **L153 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>, NoCapture<ArgIndex<1>>, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`。
- **L154 EN**: Blank line separating nearby declarations or logic blocks.
  **L154 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L155 EN**: Declares TableGen def `int_hexagon_vmemset`.
  **L155 CN**: 声明 TableGen def `int_hexagon_vmemset`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L157 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`.
  **L157 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L159 EN**: Declares TableGen multiclass `Hexagon_custom_circ_ld_Intrinsic`.
  **L159 CN**: 声明 TableGen multiclass `Hexagon_custom_circ_ld_Intrinsic`。
- **L160 EN**: Declares TableGen def `NAME#_pci`.
  **L160 CN**: 声明 TableGen def `NAME#_pci`。

### Lines 161-180

````tablegen
    [ElTy, llvm_ptr_ty],
    [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty],
    [IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;
  def NAME#_pcr : Hexagon_NonGCC_Intrinsic<
    [ElTy, llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty, llvm_ptr_ty],
    [IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;
}

defm int_hexagon_L2_loadrub : Hexagon_custom_circ_ld_Intrinsic<llvm_i32_ty>;
defm int_hexagon_L2_loadrb : Hexagon_custom_circ_ld_Intrinsic<llvm_i32_ty>;
defm int_hexagon_L2_loadruh : Hexagon_custom_circ_ld_Intrinsic<llvm_i32_ty>;
defm int_hexagon_L2_loadrh : Hexagon_custom_circ_ld_Intrinsic<llvm_i32_ty>;
defm int_hexagon_L2_loadri : Hexagon_custom_circ_ld_Intrinsic<llvm_i32_ty>;
defm int_hexagon_L2_loadrd : Hexagon_custom_circ_ld_Intrinsic<llvm_i64_ty>;

multiclass Hexagon_custom_circ_st_Intrinsic<LLVMType ElTy> {
  def NAME#_pci : Hexagon_NonGCC_Intrinsic<
    [llvm_ptr_ty],
    [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, ElTy, llvm_ptr_ty],
    [IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;
````
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ElTy, llvm_ptr_ty],`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ElTy, llvm_ptr_ty],`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty],`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty],`。
- **L163 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`.
  **L163 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`。
- **L164 EN**: Declares TableGen def `NAME#_pcr`.
  **L164 CN**: 声明 TableGen def `NAME#_pcr`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ElTy, llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty, llvm_ptr_ty],`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ElTy, llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty, llvm_ptr_ty],`。
- **L166 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;`.
  **L166 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;`。
- **L167 EN**: Closes the current lexical scope or compound statement.
  **L167 CN**: 结束当前词法作用域或复合语句块。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L169 EN**: Declares TableGen defm `int_hexagon_L2_loadrub`.
  **L169 CN**: 声明 TableGen defm `int_hexagon_L2_loadrub`。
- **L170 EN**: Declares TableGen defm `int_hexagon_L2_loadrb`.
  **L170 CN**: 声明 TableGen defm `int_hexagon_L2_loadrb`。
- **L171 EN**: Declares TableGen defm `int_hexagon_L2_loadruh`.
  **L171 CN**: 声明 TableGen defm `int_hexagon_L2_loadruh`。
- **L172 EN**: Declares TableGen defm `int_hexagon_L2_loadrh`.
  **L172 CN**: 声明 TableGen defm `int_hexagon_L2_loadrh`。
- **L173 EN**: Declares TableGen defm `int_hexagon_L2_loadri`.
  **L173 CN**: 声明 TableGen defm `int_hexagon_L2_loadri`。
- **L174 EN**: Declares TableGen defm `int_hexagon_L2_loadrd`.
  **L174 CN**: 声明 TableGen defm `int_hexagon_L2_loadrd`。
- **L175 EN**: Blank line separating nearby declarations or logic blocks.
  **L175 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L176 EN**: Declares TableGen multiclass `Hexagon_custom_circ_st_Intrinsic`.
  **L176 CN**: 声明 TableGen multiclass `Hexagon_custom_circ_st_Intrinsic`。
- **L177 EN**: Declares TableGen def `NAME#_pci`.
  **L177 CN**: 声明 TableGen def `NAME#_pci`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty],`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty],`。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, ElTy, llvm_ptr_ty],`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, ElTy, llvm_ptr_ty],`。
- **L180 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;`.
  **L180 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;`。

### Lines 181-200

````tablegen
  def NAME#_pcr : Hexagon_NonGCC_Intrinsic<
    [llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty, ElTy, llvm_ptr_ty],
    [IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;
}

defm int_hexagon_S2_storerb : Hexagon_custom_circ_st_Intrinsic<llvm_i32_ty>;
defm int_hexagon_S2_storerh : Hexagon_custom_circ_st_Intrinsic<llvm_i32_ty>;
defm int_hexagon_S2_storerf : Hexagon_custom_circ_st_Intrinsic<llvm_i32_ty>;
defm int_hexagon_S2_storeri : Hexagon_custom_circ_st_Intrinsic<llvm_i32_ty>;
defm int_hexagon_S2_storerd : Hexagon_custom_circ_st_Intrinsic<llvm_i64_ty>;

// The front-end emits the intrinsic call with only two arguments. The third
// argument from the builtin is already used by front-end to write to memory
// by generating a store.
class Hexagon_custom_brev_ld_Intrinsic<LLVMType ElTy>
 : Hexagon_NonGCC_Intrinsic<
    [ElTy, llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty],
    [IntrReadMem]>;

def int_hexagon_L2_loadrub_pbr : Hexagon_custom_brev_ld_Intrinsic<llvm_i32_ty>;
````
- **L181 EN**: Declares TableGen def `NAME#_pcr`.
  **L181 CN**: 声明 TableGen def `NAME#_pcr`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty, ElTy, llvm_ptr_ty],`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty, ElTy, llvm_ptr_ty],`。
- **L183 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`.
  **L183 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L186 EN**: Declares TableGen defm `int_hexagon_S2_storerb`.
  **L186 CN**: 声明 TableGen defm `int_hexagon_S2_storerb`。
- **L187 EN**: Declares TableGen defm `int_hexagon_S2_storerh`.
  **L187 CN**: 声明 TableGen defm `int_hexagon_S2_storerh`。
- **L188 EN**: Declares TableGen defm `int_hexagon_S2_storerf`.
  **L188 CN**: 声明 TableGen defm `int_hexagon_S2_storerf`。
- **L189 EN**: Declares TableGen defm `int_hexagon_S2_storeri`.
  **L189 CN**: 声明 TableGen defm `int_hexagon_S2_storeri`。
- **L190 EN**: Declares TableGen defm `int_hexagon_S2_storerd`.
  **L190 CN**: 声明 TableGen defm `int_hexagon_S2_storerd`。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L192 EN**: Comment explains nearby logic, invariants, or intent: `The front-end emits the intrinsic call with only two arguments. The third`.
  **L192 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The front-end emits the intrinsic call with only two arguments. The third`。
- **L193 EN**: Comment explains nearby logic, invariants, or intent: `argument from the builtin is already used by front-end to write to memory`.
  **L193 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`argument from the builtin is already used by front-end to write to memory`。
- **L194 EN**: Comment explains nearby logic, invariants, or intent: `by generating a store.`.
  **L194 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`by generating a store.`。
- **L195 EN**: Declares class `Hexagon_custom_brev_ld_Intrinsic<LLVMType`.
  **L195 CN**: 声明 class `Hexagon_custom_brev_ld_Intrinsic<LLVMType`。
- **L196 EN**: Continues the surrounding expression or declaration: `: Hexagon_NonGCC_Intrinsic<`.
  **L196 CN**: 继续构造周围的表达式或声明：`: Hexagon_NonGCC_Intrinsic<`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ElTy, llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ElTy, llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L198 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L198 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L200 EN**: Declares TableGen def `int_hexagon_L2_loadrub_pbr`.
  **L200 CN**: 声明 TableGen def `int_hexagon_L2_loadrub_pbr`。

### Lines 201-220

````tablegen
def int_hexagon_L2_loadrb_pbr : Hexagon_custom_brev_ld_Intrinsic<llvm_i32_ty>;
def int_hexagon_L2_loadruh_pbr : Hexagon_custom_brev_ld_Intrinsic<llvm_i32_ty>;
def int_hexagon_L2_loadrh_pbr : Hexagon_custom_brev_ld_Intrinsic<llvm_i32_ty>;
def int_hexagon_L2_loadri_pbr : Hexagon_custom_brev_ld_Intrinsic<llvm_i32_ty>;
def int_hexagon_L2_loadrd_pbr : Hexagon_custom_brev_ld_Intrinsic<llvm_i64_ty>;

def int_hexagon_S2_storerb_pbr : Hexagon_mem_memsisi_Intrinsic<"brev_stb">;
def int_hexagon_S2_storerh_pbr : Hexagon_mem_memsisi_Intrinsic<"brev_sth">;
def int_hexagon_S2_storerf_pbr : Hexagon_mem_memsisi_Intrinsic<"brev_sthhi">;
def int_hexagon_S2_storeri_pbr : Hexagon_mem_memsisi_Intrinsic<"brev_stw">;
def int_hexagon_S2_storerd_pbr : Hexagon_mem_memdisi_Intrinsic<"brev_std">;

// tag : V6_vrmpybub_rtt
class Hexagon_v32i32_v16i32i64_rtt_Intrinsic<string GCCIntSuffix>
  : Hexagon_Intrinsic<GCCIntSuffix,
       [llvm_v32i32_ty], [llvm_v16i32_ty,llvm_i64_ty],
       [IntrNoMem]>;

// tag : V6_vrmpybub_rtt_128B
class Hexagon_v64i32_v32i32i64_rtt_Intrinsic<string GCCIntSuffix>
````
- **L201 EN**: Declares TableGen def `int_hexagon_L2_loadrb_pbr`.
  **L201 CN**: 声明 TableGen def `int_hexagon_L2_loadrb_pbr`。
- **L202 EN**: Declares TableGen def `int_hexagon_L2_loadruh_pbr`.
  **L202 CN**: 声明 TableGen def `int_hexagon_L2_loadruh_pbr`。
- **L203 EN**: Declares TableGen def `int_hexagon_L2_loadrh_pbr`.
  **L203 CN**: 声明 TableGen def `int_hexagon_L2_loadrh_pbr`。
- **L204 EN**: Declares TableGen def `int_hexagon_L2_loadri_pbr`.
  **L204 CN**: 声明 TableGen def `int_hexagon_L2_loadri_pbr`。
- **L205 EN**: Declares TableGen def `int_hexagon_L2_loadrd_pbr`.
  **L205 CN**: 声明 TableGen def `int_hexagon_L2_loadrd_pbr`。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L207 EN**: Declares TableGen def `int_hexagon_S2_storerb_pbr`.
  **L207 CN**: 声明 TableGen def `int_hexagon_S2_storerb_pbr`。
- **L208 EN**: Declares TableGen def `int_hexagon_S2_storerh_pbr`.
  **L208 CN**: 声明 TableGen def `int_hexagon_S2_storerh_pbr`。
- **L209 EN**: Declares TableGen def `int_hexagon_S2_storerf_pbr`.
  **L209 CN**: 声明 TableGen def `int_hexagon_S2_storerf_pbr`。
- **L210 EN**: Declares TableGen def `int_hexagon_S2_storeri_pbr`.
  **L210 CN**: 声明 TableGen def `int_hexagon_S2_storeri_pbr`。
- **L211 EN**: Declares TableGen def `int_hexagon_S2_storerd_pbr`.
  **L211 CN**: 声明 TableGen def `int_hexagon_S2_storerd_pbr`。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L213 EN**: Comment explains nearby logic, invariants, or intent: `tag : V6_vrmpybub_rtt`.
  **L213 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tag : V6_vrmpybub_rtt`。
- **L214 EN**: Declares class `Hexagon_v32i32_v16i32i64_rtt_Intrinsic<string`.
  **L214 CN**: 声明 class `Hexagon_v32i32_v16i32i64_rtt_Intrinsic<string`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v32i32_ty], [llvm_v16i32_ty,llvm_i64_ty],`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v32i32_ty], [llvm_v16i32_ty,llvm_i64_ty],`。
- **L217 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L217 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L219 EN**: Comment explains nearby logic, invariants, or intent: `tag : V6_vrmpybub_rtt_128B`.
  **L219 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tag : V6_vrmpybub_rtt_128B`。
- **L220 EN**: Declares class `Hexagon_v64i32_v32i32i64_rtt_Intrinsic<string`.
  **L220 CN**: 声明 class `Hexagon_v64i32_v32i32i64_rtt_Intrinsic<string`。

### Lines 221-240

````tablegen
  : Hexagon_Intrinsic<GCCIntSuffix,
       [llvm_v64i32_ty], [llvm_v32i32_ty,llvm_i64_ty],
       [IntrNoMem]>;

// tag : V6_vrmpybub_rtt_acc
class Hexagon_v32i32_v32i32v16i32i64_rtt_Intrinsic<string GCCIntSuffix>
  : Hexagon_Intrinsic<GCCIntSuffix,
       [llvm_v32i32_ty], [llvm_v32i32_ty,llvm_v16i32_ty,llvm_i64_ty],
       [IntrNoMem]>;

// tag : V6_vrmpybub_rtt_acc_128B
class Hexagon_v64i32_v64i32v32i32i64_rtt_Intrinsic<string GCCIntSuffix>
  : Hexagon_Intrinsic<GCCIntSuffix,
       [llvm_v64i32_ty], [llvm_v64i32_ty,llvm_v32i32_ty,llvm_i64_ty],
       [IntrNoMem]>;

def int_hexagon_V6_vrmpybub_rtt :
Hexagon_v32i32_v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt">;

def int_hexagon_V6_vrmpybub_rtt_128B :
````
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v64i32_ty], [llvm_v32i32_ty,llvm_i64_ty],`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v64i32_ty], [llvm_v32i32_ty,llvm_i64_ty],`。
- **L223 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L223 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L224 EN**: Blank line separating nearby declarations or logic blocks.
  **L224 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L225 EN**: Comment explains nearby logic, invariants, or intent: `tag : V6_vrmpybub_rtt_acc`.
  **L225 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tag : V6_vrmpybub_rtt_acc`。
- **L226 EN**: Declares class `Hexagon_v32i32_v32i32v16i32i64_rtt_Intrinsic<string`.
  **L226 CN**: 声明 class `Hexagon_v32i32_v32i32v16i32i64_rtt_Intrinsic<string`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v32i32_ty], [llvm_v32i32_ty,llvm_v16i32_ty,llvm_i64_ty],`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v32i32_ty], [llvm_v32i32_ty,llvm_v16i32_ty,llvm_i64_ty],`。
- **L229 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L229 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L231 EN**: Comment explains nearby logic, invariants, or intent: `tag : V6_vrmpybub_rtt_acc_128B`.
  **L231 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tag : V6_vrmpybub_rtt_acc_128B`。
- **L232 EN**: Declares class `Hexagon_v64i32_v64i32v32i32i64_rtt_Intrinsic<string`.
  **L232 CN**: 声明 class `Hexagon_v64i32_v64i32v32i32i64_rtt_Intrinsic<string`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v64i32_ty], [llvm_v64i32_ty,llvm_v32i32_ty,llvm_i64_ty],`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v64i32_ty], [llvm_v64i32_ty,llvm_v32i32_ty,llvm_i64_ty],`。
- **L235 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L235 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L237 EN**: Declares TableGen def `int_hexagon_V6_vrmpybub_rtt`.
  **L237 CN**: 声明 TableGen def `int_hexagon_V6_vrmpybub_rtt`。
- **L238 EN**: Executes a standalone statement or declaration: `Hexagon_v32i32_v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt">;`.
  **L238 CN**: 执行一条独立语句或声明：`Hexagon_v32i32_v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt">;`。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L240 EN**: Declares TableGen def `int_hexagon_V6_vrmpybub_rtt_128B`.
  **L240 CN**: 声明 TableGen def `int_hexagon_V6_vrmpybub_rtt_128B`。

### Lines 241-260

````tablegen
Hexagon_v64i32_v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt_128B">;

def int_hexagon_V6_vrmpybub_rtt_acc :
Hexagon_v32i32_v32i32v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt_acc">;

def int_hexagon_V6_vrmpybub_rtt_acc_128B :
Hexagon_v64i32_v64i32v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt_acc_128B">;

def int_hexagon_V6_vrmpyub_rtt :
Hexagon_v32i32_v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt">;

def int_hexagon_V6_vrmpyub_rtt_128B :
Hexagon_v64i32_v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt_128B">;

def int_hexagon_V6_vrmpyub_rtt_acc :
Hexagon_v32i32_v32i32v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt_acc">;

def int_hexagon_V6_vrmpyub_rtt_acc_128B :
Hexagon_v64i32_v64i32v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt_acc_128B">;

````
- **L241 EN**: Executes a standalone statement or declaration: `Hexagon_v64i32_v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt_128B">;`.
  **L241 CN**: 执行一条独立语句或声明：`Hexagon_v64i32_v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt_128B">;`。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L243 EN**: Declares TableGen def `int_hexagon_V6_vrmpybub_rtt_acc`.
  **L243 CN**: 声明 TableGen def `int_hexagon_V6_vrmpybub_rtt_acc`。
- **L244 EN**: Executes a standalone statement or declaration: `Hexagon_v32i32_v32i32v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt_acc">;`.
  **L244 CN**: 执行一条独立语句或声明：`Hexagon_v32i32_v32i32v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt_acc">;`。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L246 EN**: Declares TableGen def `int_hexagon_V6_vrmpybub_rtt_acc_128B`.
  **L246 CN**: 声明 TableGen def `int_hexagon_V6_vrmpybub_rtt_acc_128B`。
- **L247 EN**: Executes a standalone statement or declaration: `Hexagon_v64i32_v64i32v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt_acc_128B">;`.
  **L247 CN**: 执行一条独立语句或声明：`Hexagon_v64i32_v64i32v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpybub_rtt_acc_128B">;`。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L249 EN**: Declares TableGen def `int_hexagon_V6_vrmpyub_rtt`.
  **L249 CN**: 声明 TableGen def `int_hexagon_V6_vrmpyub_rtt`。
- **L250 EN**: Executes a standalone statement or declaration: `Hexagon_v32i32_v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt">;`.
  **L250 CN**: 执行一条独立语句或声明：`Hexagon_v32i32_v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt">;`。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L252 EN**: Declares TableGen def `int_hexagon_V6_vrmpyub_rtt_128B`.
  **L252 CN**: 声明 TableGen def `int_hexagon_V6_vrmpyub_rtt_128B`。
- **L253 EN**: Executes a standalone statement or declaration: `Hexagon_v64i32_v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt_128B">;`.
  **L253 CN**: 执行一条独立语句或声明：`Hexagon_v64i32_v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt_128B">;`。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L255 EN**: Declares TableGen def `int_hexagon_V6_vrmpyub_rtt_acc`.
  **L255 CN**: 声明 TableGen def `int_hexagon_V6_vrmpyub_rtt_acc`。
- **L256 EN**: Executes a standalone statement or declaration: `Hexagon_v32i32_v32i32v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt_acc">;`.
  **L256 CN**: 执行一条独立语句或声明：`Hexagon_v32i32_v32i32v16i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt_acc">;`。
- **L257 EN**: Blank line separating nearby declarations or logic blocks.
  **L257 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L258 EN**: Declares TableGen def `int_hexagon_V6_vrmpyub_rtt_acc_128B`.
  **L258 CN**: 声明 TableGen def `int_hexagon_V6_vrmpyub_rtt_acc_128B`。
- **L259 EN**: Executes a standalone statement or declaration: `Hexagon_v64i32_v64i32v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt_acc_128B">;`.
  **L259 CN**: 执行一条独立语句或声明：`Hexagon_v64i32_v64i32v32i32i64_rtt_Intrinsic<"HEXAGON_V6_vrmpyub_rtt_acc_128B">;`。
- **L260 EN**: Blank line separating nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 261-280

````tablegen
// HVX conditional loads/stores

class Hexagon_pred_vload_imm<LLVMType ValTy>
  : Hexagon_NonGCC_Intrinsic<
      [ValTy],
      [llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty],
      [IntrReadMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,
       ImmArg<ArgIndex<2>>]>;

class Hexagon_pred_vload_imm_64B:  Hexagon_pred_vload_imm<llvm_v16i32_ty>;
class Hexagon_pred_vload_imm_128B: Hexagon_pred_vload_imm<llvm_v32i32_ty>;

def int_hexagon_V6_vL32b_pred_ai:            Hexagon_pred_vload_imm_64B;
def int_hexagon_V6_vL32b_npred_ai:           Hexagon_pred_vload_imm_64B;
def int_hexagon_V6_vL32b_nt_pred_ai:         Hexagon_pred_vload_imm_64B;
def int_hexagon_V6_vL32b_nt_npred_ai:        Hexagon_pred_vload_imm_64B;
def int_hexagon_V6_vL32b_pred_ai_128B:      Hexagon_pred_vload_imm_128B;
def int_hexagon_V6_vL32b_npred_ai_128B:     Hexagon_pred_vload_imm_128B;
def int_hexagon_V6_vL32b_nt_pred_ai_128B:   Hexagon_pred_vload_imm_128B;
def int_hexagon_V6_vL32b_nt_npred_ai_128B:  Hexagon_pred_vload_imm_128B;
````
- **L261 EN**: Comment explains nearby logic, invariants, or intent: `HVX conditional loads/stores`.
  **L261 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`HVX conditional loads/stores`。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L263 EN**: Declares class `Hexagon_pred_vload_imm<LLVMType`.
  **L263 CN**: 声明 class `Hexagon_pred_vload_imm<LLVMType`。
- **L264 EN**: Continues the surrounding expression or declaration: `: Hexagon_NonGCC_Intrinsic<`.
  **L264 CN**: 继续构造周围的表达式或声明：`: Hexagon_NonGCC_Intrinsic<`。
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ValTy],`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ValTy],`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,`。
- **L268 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>]>;`.
  **L268 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>]>;`。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L270 EN**: Declares class `Hexagon_pred_vload_imm_64B`.
  **L270 CN**: 声明 class `Hexagon_pred_vload_imm_64B`。
- **L271 EN**: Declares class `Hexagon_pred_vload_imm_128B`.
  **L271 CN**: 声明 class `Hexagon_pred_vload_imm_128B`。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L273 EN**: Declares TableGen def `int_hexagon_V6_vL32b_pred_ai`.
  **L273 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_pred_ai`。
- **L274 EN**: Declares TableGen def `int_hexagon_V6_vL32b_npred_ai`.
  **L274 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_npred_ai`。
- **L275 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_pred_ai`.
  **L275 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_pred_ai`。
- **L276 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_npred_ai`.
  **L276 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_npred_ai`。
- **L277 EN**: Declares TableGen def `int_hexagon_V6_vL32b_pred_ai_128B`.
  **L277 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_pred_ai_128B`。
- **L278 EN**: Declares TableGen def `int_hexagon_V6_vL32b_npred_ai_128B`.
  **L278 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_npred_ai_128B`。
- **L279 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_pred_ai_128B`.
  **L279 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_pred_ai_128B`。
- **L280 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_npred_ai_128B`.
  **L280 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_npred_ai_128B`。

### Lines 281-300

````tablegen

class Hexagom_pred_vload_upd<LLVMType ValTy, bit TakesImm>
  : Hexagon_NonGCC_Intrinsic<
      [ValTy, llvm_ptr_ty],
      [llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty],
      !if(TakesImm,
          [IntrReadMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,
           ImmArg<ArgIndex<2>>],
          [IntrReadMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>])>;

class Hexagom_pred_vload_upd_64B<bit TakesImm>
  : Hexagom_pred_vload_upd<llvm_v16i32_ty, TakesImm>;
class Hexagom_pred_vload_upd_128B<bit TakesImm>
  : Hexagom_pred_vload_upd<llvm_v32i32_ty, TakesImm>;

def int_hexagon_V6_vL32b_pred_pi:            Hexagom_pred_vload_upd_64B<1>;
def int_hexagon_V6_vL32b_npred_pi:           Hexagom_pred_vload_upd_64B<1>;
def int_hexagon_V6_vL32b_nt_pred_pi:         Hexagom_pred_vload_upd_64B<1>;
def int_hexagon_V6_vL32b_nt_npred_pi:        Hexagom_pred_vload_upd_64B<1>;
def int_hexagon_V6_vL32b_pred_pi_128B:      Hexagom_pred_vload_upd_128B<1>;
````
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L282 EN**: Declares class `Hexagom_pred_vload_upd<LLVMType`.
  **L282 CN**: 声明 class `Hexagom_pred_vload_upd<LLVMType`。
- **L283 EN**: Continues the surrounding expression or declaration: `: Hexagon_NonGCC_Intrinsic<`.
  **L283 CN**: 继续构造周围的表达式或声明：`: Hexagon_NonGCC_Intrinsic<`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ValTy, llvm_ptr_ty],`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ValTy, llvm_ptr_ty],`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(TakesImm,`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(TakesImm,`。
- **L287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,`.
  **L287 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,`。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<2>>],`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<2>>],`。
- **L289 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>])>;`.
  **L289 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>])>;`。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L291 EN**: Declares class `Hexagom_pred_vload_upd_64B<bit`.
  **L291 CN**: 声明 class `Hexagom_pred_vload_upd_64B<bit`。
- **L292 EN**: Executes a standalone statement or declaration: `: Hexagom_pred_vload_upd<llvm_v16i32_ty, TakesImm>;`.
  **L292 CN**: 执行一条独立语句或声明：`: Hexagom_pred_vload_upd<llvm_v16i32_ty, TakesImm>;`。
- **L293 EN**: Declares class `Hexagom_pred_vload_upd_128B<bit`.
  **L293 CN**: 声明 class `Hexagom_pred_vload_upd_128B<bit`。
- **L294 EN**: Executes a standalone statement or declaration: `: Hexagom_pred_vload_upd<llvm_v32i32_ty, TakesImm>;`.
  **L294 CN**: 执行一条独立语句或声明：`: Hexagom_pred_vload_upd<llvm_v32i32_ty, TakesImm>;`。
- **L295 EN**: Blank line separating nearby declarations or logic blocks.
  **L295 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L296 EN**: Declares TableGen def `int_hexagon_V6_vL32b_pred_pi`.
  **L296 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_pred_pi`。
- **L297 EN**: Declares TableGen def `int_hexagon_V6_vL32b_npred_pi`.
  **L297 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_npred_pi`。
- **L298 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_pred_pi`.
  **L298 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_pred_pi`。
- **L299 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_npred_pi`.
  **L299 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_npred_pi`。
- **L300 EN**: Declares TableGen def `int_hexagon_V6_vL32b_pred_pi_128B`.
  **L300 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_pred_pi_128B`。

### Lines 301-320

````tablegen
def int_hexagon_V6_vL32b_npred_pi_128B:     Hexagom_pred_vload_upd_128B<1>;
def int_hexagon_V6_vL32b_nt_pred_pi_128B:   Hexagom_pred_vload_upd_128B<1>;
def int_hexagon_V6_vL32b_nt_npred_pi_128B:  Hexagom_pred_vload_upd_128B<1>;

def int_hexagon_V6_vL32b_pred_ppu:           Hexagom_pred_vload_upd_64B<0>;
def int_hexagon_V6_vL32b_npred_ppu:          Hexagom_pred_vload_upd_64B<0>;
def int_hexagon_V6_vL32b_nt_pred_ppu:        Hexagom_pred_vload_upd_64B<0>;
def int_hexagon_V6_vL32b_nt_npred_ppu:       Hexagom_pred_vload_upd_64B<0>;
def int_hexagon_V6_vL32b_pred_ppu_128B:     Hexagom_pred_vload_upd_128B<0>;
def int_hexagon_V6_vL32b_npred_ppu_128B:    Hexagom_pred_vload_upd_128B<0>;
def int_hexagon_V6_vL32b_nt_pred_ppu_128B:  Hexagom_pred_vload_upd_128B<0>;
def int_hexagon_V6_vL32b_nt_npred_ppu_128B: Hexagom_pred_vload_upd_128B<0>;


class Hexagon_pred_vstore_imm<LLVMType ValTy>
  : Hexagon_NonGCC_Intrinsic<
      [],
      [llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty, ValTy],
      [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,
       ImmArg<ArgIndex<2>>]>;
````
- **L301 EN**: Declares TableGen def `int_hexagon_V6_vL32b_npred_pi_128B`.
  **L301 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_npred_pi_128B`。
- **L302 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_pred_pi_128B`.
  **L302 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_pred_pi_128B`。
- **L303 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_npred_pi_128B`.
  **L303 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_npred_pi_128B`。
- **L304 EN**: Blank line separating nearby declarations or logic blocks.
  **L304 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L305 EN**: Declares TableGen def `int_hexagon_V6_vL32b_pred_ppu`.
  **L305 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_pred_ppu`。
- **L306 EN**: Declares TableGen def `int_hexagon_V6_vL32b_npred_ppu`.
  **L306 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_npred_ppu`。
- **L307 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_pred_ppu`.
  **L307 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_pred_ppu`。
- **L308 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_npred_ppu`.
  **L308 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_npred_ppu`。
- **L309 EN**: Declares TableGen def `int_hexagon_V6_vL32b_pred_ppu_128B`.
  **L309 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_pred_ppu_128B`。
- **L310 EN**: Declares TableGen def `int_hexagon_V6_vL32b_npred_ppu_128B`.
  **L310 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_npred_ppu_128B`。
- **L311 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_pred_ppu_128B`.
  **L311 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_pred_ppu_128B`。
- **L312 EN**: Declares TableGen def `int_hexagon_V6_vL32b_nt_npred_ppu_128B`.
  **L312 CN**: 声明 TableGen def `int_hexagon_V6_vL32b_nt_npred_ppu_128B`。
- **L313 EN**: Blank line separating nearby declarations or logic blocks.
  **L313 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L315 EN**: Declares class `Hexagon_pred_vstore_imm<LLVMType`.
  **L315 CN**: 声明 class `Hexagon_pred_vstore_imm<LLVMType`。
- **L316 EN**: Continues the surrounding expression or declaration: `: Hexagon_NonGCC_Intrinsic<`.
  **L316 CN**: 继续构造周围的表达式或声明：`: Hexagon_NonGCC_Intrinsic<`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty, ValTy],`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty, ValTy],`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,`。
- **L320 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>]>;`.
  **L320 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>]>;`。

### Lines 321-340

````tablegen

class Hexagon_pred_vstore_imm_64B:  Hexagon_pred_vstore_imm<llvm_v16i32_ty>;
class Hexagon_pred_vstore_imm_128B: Hexagon_pred_vstore_imm<llvm_v32i32_ty>;

def int_hexagon_V6_vS32b_pred_ai:            Hexagon_pred_vstore_imm_64B;
def int_hexagon_V6_vS32b_npred_ai:           Hexagon_pred_vstore_imm_64B;
def int_hexagon_V6_vS32Ub_pred_ai:           Hexagon_pred_vstore_imm_64B;
def int_hexagon_V6_vS32Ub_npred_ai:          Hexagon_pred_vstore_imm_64B;
def int_hexagon_V6_vS32b_nt_pred_ai:         Hexagon_pred_vstore_imm_64B;
def int_hexagon_V6_vS32b_nt_npred_ai:        Hexagon_pred_vstore_imm_64B;
def int_hexagon_V6_vS32b_pred_ai_128B:      Hexagon_pred_vstore_imm_128B;
def int_hexagon_V6_vS32b_npred_ai_128B:     Hexagon_pred_vstore_imm_128B;
def int_hexagon_V6_vS32Ub_pred_ai_128B:     Hexagon_pred_vstore_imm_128B;
def int_hexagon_V6_vS32Ub_npred_ai_128B:    Hexagon_pred_vstore_imm_128B;
def int_hexagon_V6_vS32b_nt_pred_ai_128B:   Hexagon_pred_vstore_imm_128B;
def int_hexagon_V6_vS32b_nt_npred_ai_128B:  Hexagon_pred_vstore_imm_128B;

class Hexagon_pred_vstore_upd<LLVMType ValTy, bit TakesImm>
  : Hexagon_NonGCC_Intrinsic<
      [llvm_ptr_ty],
````
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L322 EN**: Declares class `Hexagon_pred_vstore_imm_64B`.
  **L322 CN**: 声明 class `Hexagon_pred_vstore_imm_64B`。
- **L323 EN**: Declares class `Hexagon_pred_vstore_imm_128B`.
  **L323 CN**: 声明 class `Hexagon_pred_vstore_imm_128B`。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L325 EN**: Declares TableGen def `int_hexagon_V6_vS32b_pred_ai`.
  **L325 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_pred_ai`。
- **L326 EN**: Declares TableGen def `int_hexagon_V6_vS32b_npred_ai`.
  **L326 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_npred_ai`。
- **L327 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_pred_ai`.
  **L327 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_pred_ai`。
- **L328 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_npred_ai`.
  **L328 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_npred_ai`。
- **L329 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_pred_ai`.
  **L329 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_pred_ai`。
- **L330 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_npred_ai`.
  **L330 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_npred_ai`。
- **L331 EN**: Declares TableGen def `int_hexagon_V6_vS32b_pred_ai_128B`.
  **L331 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_pred_ai_128B`。
- **L332 EN**: Declares TableGen def `int_hexagon_V6_vS32b_npred_ai_128B`.
  **L332 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_npred_ai_128B`。
- **L333 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_pred_ai_128B`.
  **L333 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_pred_ai_128B`。
- **L334 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_npred_ai_128B`.
  **L334 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_npred_ai_128B`。
- **L335 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_pred_ai_128B`.
  **L335 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_pred_ai_128B`。
- **L336 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_npred_ai_128B`.
  **L336 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_npred_ai_128B`。
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L338 EN**: Declares class `Hexagon_pred_vstore_upd<LLVMType`.
  **L338 CN**: 声明 class `Hexagon_pred_vstore_upd<LLVMType`。
- **L339 EN**: Continues the surrounding expression or declaration: `: Hexagon_NonGCC_Intrinsic<`.
  **L339 CN**: 继续构造周围的表达式或声明：`: Hexagon_NonGCC_Intrinsic<`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty],`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty],`。

### Lines 341-360

````tablegen
      [llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty, ValTy],
      !if(TakesImm,
          [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,
           ImmArg<ArgIndex<2>>],
          [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>])>;

class Hexagon_pred_vstore_upd_64B<bit TakesImm>
  : Hexagon_pred_vstore_upd<llvm_v16i32_ty, TakesImm>;
class Hexagon_pred_vstore_upd_128B<bit TakesImm>
  : Hexagon_pred_vstore_upd<llvm_v32i32_ty, TakesImm>;

def int_hexagon_V6_vS32b_pred_pi:            Hexagon_pred_vstore_upd_64B<1>;
def int_hexagon_V6_vS32b_npred_pi:           Hexagon_pred_vstore_upd_64B<1>;
def int_hexagon_V6_vS32Ub_pred_pi:           Hexagon_pred_vstore_upd_64B<1>;
def int_hexagon_V6_vS32Ub_npred_pi:          Hexagon_pred_vstore_upd_64B<1>;
def int_hexagon_V6_vS32b_nt_pred_pi:         Hexagon_pred_vstore_upd_64B<1>;
def int_hexagon_V6_vS32b_nt_npred_pi:        Hexagon_pred_vstore_upd_64B<1>;
def int_hexagon_V6_vS32b_pred_pi_128B:      Hexagon_pred_vstore_upd_128B<1>;
def int_hexagon_V6_vS32b_npred_pi_128B:     Hexagon_pred_vstore_upd_128B<1>;
def int_hexagon_V6_vS32Ub_pred_pi_128B:     Hexagon_pred_vstore_upd_128B<1>;
````
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty, ValTy],`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i1_ty, llvm_ptr_ty, llvm_i32_ty, ValTy],`。
- **L342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(TakesImm,`.
  **L342 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(TakesImm,`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>,`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<2>>],`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<2>>],`。
- **L345 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>])>;`.
  **L345 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<1>>])>;`。
- **L346 EN**: Blank line separating nearby declarations or logic blocks.
  **L346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L347 EN**: Declares class `Hexagon_pred_vstore_upd_64B<bit`.
  **L347 CN**: 声明 class `Hexagon_pred_vstore_upd_64B<bit`。
- **L348 EN**: Executes a standalone statement or declaration: `: Hexagon_pred_vstore_upd<llvm_v16i32_ty, TakesImm>;`.
  **L348 CN**: 执行一条独立语句或声明：`: Hexagon_pred_vstore_upd<llvm_v16i32_ty, TakesImm>;`。
- **L349 EN**: Declares class `Hexagon_pred_vstore_upd_128B<bit`.
  **L349 CN**: 声明 class `Hexagon_pred_vstore_upd_128B<bit`。
- **L350 EN**: Executes a standalone statement or declaration: `: Hexagon_pred_vstore_upd<llvm_v32i32_ty, TakesImm>;`.
  **L350 CN**: 执行一条独立语句或声明：`: Hexagon_pred_vstore_upd<llvm_v32i32_ty, TakesImm>;`。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L352 EN**: Declares TableGen def `int_hexagon_V6_vS32b_pred_pi`.
  **L352 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_pred_pi`。
- **L353 EN**: Declares TableGen def `int_hexagon_V6_vS32b_npred_pi`.
  **L353 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_npred_pi`。
- **L354 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_pred_pi`.
  **L354 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_pred_pi`。
- **L355 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_npred_pi`.
  **L355 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_npred_pi`。
- **L356 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_pred_pi`.
  **L356 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_pred_pi`。
- **L357 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_npred_pi`.
  **L357 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_npred_pi`。
- **L358 EN**: Declares TableGen def `int_hexagon_V6_vS32b_pred_pi_128B`.
  **L358 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_pred_pi_128B`。
- **L359 EN**: Declares TableGen def `int_hexagon_V6_vS32b_npred_pi_128B`.
  **L359 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_npred_pi_128B`。
- **L360 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_pred_pi_128B`.
  **L360 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_pred_pi_128B`。

### Lines 361-380

````tablegen
def int_hexagon_V6_vS32Ub_npred_pi_128B:    Hexagon_pred_vstore_upd_128B<1>;
def int_hexagon_V6_vS32b_nt_pred_pi_128B:   Hexagon_pred_vstore_upd_128B<1>;
def int_hexagon_V6_vS32b_nt_npred_pi_128B:  Hexagon_pred_vstore_upd_128B<1>;

def int_hexagon_V6_vS32b_pred_ppu:           Hexagon_pred_vstore_upd_64B<0>;
def int_hexagon_V6_vS32b_npred_ppu:          Hexagon_pred_vstore_upd_64B<0>;
def int_hexagon_V6_vS32Ub_pred_ppu:          Hexagon_pred_vstore_upd_64B<0>;
def int_hexagon_V6_vS32Ub_npred_ppu:         Hexagon_pred_vstore_upd_64B<0>;
def int_hexagon_V6_vS32b_nt_pred_ppu:        Hexagon_pred_vstore_upd_64B<0>;
def int_hexagon_V6_vS32b_nt_npred_ppu:       Hexagon_pred_vstore_upd_64B<0>;
def int_hexagon_V6_vS32b_pred_ppu_128B:     Hexagon_pred_vstore_upd_128B<0>;
def int_hexagon_V6_vS32b_npred_ppu_128B:    Hexagon_pred_vstore_upd_128B<0>;
def int_hexagon_V6_vS32Ub_pred_ppu_128B:    Hexagon_pred_vstore_upd_128B<0>;
def int_hexagon_V6_vS32Ub_npred_ppu_128B:   Hexagon_pred_vstore_upd_128B<0>;
def int_hexagon_V6_vS32b_nt_pred_ppu_128B:  Hexagon_pred_vstore_upd_128B<0>;
def int_hexagon_V6_vS32b_nt_npred_ppu_128B: Hexagon_pred_vstore_upd_128B<0>;


// HVX Vector predicate casts.
// These intrinsics do not emit (nor do they correspond to) any instructions,
````
- **L361 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_npred_pi_128B`.
  **L361 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_npred_pi_128B`。
- **L362 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_pred_pi_128B`.
  **L362 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_pred_pi_128B`。
- **L363 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_npred_pi_128B`.
  **L363 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_npred_pi_128B`。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L365 EN**: Declares TableGen def `int_hexagon_V6_vS32b_pred_ppu`.
  **L365 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_pred_ppu`。
- **L366 EN**: Declares TableGen def `int_hexagon_V6_vS32b_npred_ppu`.
  **L366 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_npred_ppu`。
- **L367 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_pred_ppu`.
  **L367 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_pred_ppu`。
- **L368 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_npred_ppu`.
  **L368 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_npred_ppu`。
- **L369 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_pred_ppu`.
  **L369 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_pred_ppu`。
- **L370 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_npred_ppu`.
  **L370 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_npred_ppu`。
- **L371 EN**: Declares TableGen def `int_hexagon_V6_vS32b_pred_ppu_128B`.
  **L371 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_pred_ppu_128B`。
- **L372 EN**: Declares TableGen def `int_hexagon_V6_vS32b_npred_ppu_128B`.
  **L372 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_npred_ppu_128B`。
- **L373 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_pred_ppu_128B`.
  **L373 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_pred_ppu_128B`。
- **L374 EN**: Declares TableGen def `int_hexagon_V6_vS32Ub_npred_ppu_128B`.
  **L374 CN**: 声明 TableGen def `int_hexagon_V6_vS32Ub_npred_ppu_128B`。
- **L375 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_pred_ppu_128B`.
  **L375 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_pred_ppu_128B`。
- **L376 EN**: Declares TableGen def `int_hexagon_V6_vS32b_nt_npred_ppu_128B`.
  **L376 CN**: 声明 TableGen def `int_hexagon_V6_vS32b_nt_npred_ppu_128B`。
- **L377 EN**: Blank line separating nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L379 EN**: Comment explains nearby logic, invariants, or intent: `HVX Vector predicate casts.`.
  **L379 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`HVX Vector predicate casts.`。
- **L380 EN**: Comment explains nearby logic, invariants, or intent: `These intrinsics do not emit (nor do they correspond to) any instructions,`.
  **L380 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These intrinsics do not emit (nor do they correspond to) any instructions,`。

### Lines 381-400

````tablegen
// they are no-ops.

def int_hexagon_V6_pred_typecast :
Hexagon_NonGCC_Intrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;

def int_hexagon_V6_pred_typecast_128B :
Hexagon_NonGCC_Intrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;

// HVX full-precision multiplication.
// V6_vmpyss_parts(Vu,Vv) = (MulHS(Vu,Vv),  Mul(Vu,Vv))
// V6_vmpyuu_parts(Vu,Vv) = (MulHU(Vu,Vv),  Mul(Vu,Vv))
// V6_vmpyus_parts(Vu,Vv) = (MulHUS(Vu,Vv), Mul(Vu,Vv))
//
// Both, the (purportedly) 64b and the _128B versions are exactly equivalent
// regardless of the HVX mode, they are both defined for consistency.
// The purpose of these intrinsics is to have a uniform way of multiplying two
// integer vectors in the LLVM IR. Many HVX multiply operations interleave
// the even-odd results, except for 32x32 multiplications. Also, different
// HVX versions have different instructions that can be used, so defer the
// instruction choice to the isel.
````
- **L381 EN**: Comment explains nearby logic, invariants, or intent: `they are no-ops.`.
  **L381 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`they are no-ops.`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L383 EN**: Declares TableGen def `int_hexagon_V6_pred_typecast`.
  **L383 CN**: 声明 TableGen def `int_hexagon_V6_pred_typecast`。
- **L384 EN**: Executes a standalone statement or declaration: `Hexagon_NonGCC_Intrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`.
  **L384 CN**: 执行一条独立语句或声明：`Hexagon_NonGCC_Intrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`。
- **L385 EN**: Blank line separating nearby declarations or logic blocks.
  **L385 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L386 EN**: Declares TableGen def `int_hexagon_V6_pred_typecast_128B`.
  **L386 CN**: 声明 TableGen def `int_hexagon_V6_pred_typecast_128B`。
- **L387 EN**: Executes a standalone statement or declaration: `Hexagon_NonGCC_Intrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`.
  **L387 CN**: 执行一条独立语句或声明：`Hexagon_NonGCC_Intrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`。
- **L388 EN**: Blank line separating nearby declarations or logic blocks.
  **L388 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L389 EN**: Comment explains nearby logic, invariants, or intent: `HVX full-precision multiplication.`.
  **L389 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`HVX full-precision multiplication.`。
- **L390 EN**: Comment explains nearby logic, invariants, or intent: `V6_vmpyss_parts(Vu,Vv) = (MulHS(Vu,Vv),  Mul(Vu,Vv))`.
  **L390 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`V6_vmpyss_parts(Vu,Vv) = (MulHS(Vu,Vv),  Mul(Vu,Vv))`。
- **L391 EN**: Comment explains nearby logic, invariants, or intent: `V6_vmpyuu_parts(Vu,Vv) = (MulHU(Vu,Vv),  Mul(Vu,Vv))`.
  **L391 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`V6_vmpyuu_parts(Vu,Vv) = (MulHU(Vu,Vv),  Mul(Vu,Vv))`。
- **L392 EN**: Comment explains nearby logic, invariants, or intent: `V6_vmpyus_parts(Vu,Vv) = (MulHUS(Vu,Vv), Mul(Vu,Vv))`.
  **L392 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`V6_vmpyus_parts(Vu,Vv) = (MulHUS(Vu,Vv), Mul(Vu,Vv))`。
- **L393 EN**: Separator comment used for visual grouping.
  **L393 CN**: 用于视觉分组的分隔注释。
- **L394 EN**: Comment explains nearby logic, invariants, or intent: `Both, the (purportedly) 64b and the _128B versions are exactly equivalent`.
  **L394 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Both, the (purportedly) 64b and the _128B versions are exactly equivalent`。
- **L395 EN**: Comment explains nearby logic, invariants, or intent: `regardless of the HVX mode, they are both defined for consistency.`.
  **L395 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`regardless of the HVX mode, they are both defined for consistency.`。
- **L396 EN**: Comment explains nearby logic, invariants, or intent: `The purpose of these intrinsics is to have a uniform way of multiplying two`.
  **L396 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The purpose of these intrinsics is to have a uniform way of multiplying two`。
- **L397 EN**: Comment explains nearby logic, invariants, or intent: `integer vectors in the LLVM IR. Many HVX multiply operations interleave`.
  **L397 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`integer vectors in the LLVM IR. Many HVX multiply operations interleave`。
- **L398 EN**: Comment explains nearby logic, invariants, or intent: `the even-odd results, except for 32x32 multiplications. Also, different`.
  **L398 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the even-odd results, except for 32x32 multiplications. Also, different`。
- **L399 EN**: Comment explains nearby logic, invariants, or intent: `HVX versions have different instructions that can be used, so defer the`.
  **L399 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`HVX versions have different instructions that can be used, so defer the`。
- **L400 EN**: Comment explains nearby logic, invariants, or intent: `instruction choice to the isel.`.
  **L400 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`instruction choice to the isel.`。

### Lines 401-420

````tablegen
class Hexagon_vv_vv_pure:
  Hexagon_NonGCC_Intrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>],
    [LLVMMatchType<0>, LLVMMatchType<0>],
    [IntrNoMem]>;

def int_hexagon_V6_vmpyss_parts:      Hexagon_vv_vv_pure;
def int_hexagon_V6_vmpyss_parts_128B: Hexagon_vv_vv_pure;
def int_hexagon_V6_vmpyuu_parts:      Hexagon_vv_vv_pure;
def int_hexagon_V6_vmpyuu_parts_128B: Hexagon_vv_vv_pure;
def int_hexagon_V6_vmpyus_parts:      Hexagon_vv_vv_pure;
def int_hexagon_V6_vmpyus_parts_128B: Hexagon_vv_vv_pure;


// Masked vector stores
//
// These are all deprecated, the intrinsics matching instruction names
// should be used instead, e.g. int_hexagon_V6_vS32b_qpred_ai, etc.

class Hexagon_custom_vms_Intrinsic
````
- **L401 EN**: Declares class `Hexagon_vv_vv_pure`.
  **L401 CN**: 声明 class `Hexagon_vv_vv_pure`。
- **L402 EN**: Continues the surrounding expression or declaration: `Hexagon_NonGCC_Intrinsic<`.
  **L402 CN**: 继续构造周围的表达式或声明：`Hexagon_NonGCC_Intrinsic<`。
- **L403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L403 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L405 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L405 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L406 EN**: Blank line separating nearby declarations or logic blocks.
  **L406 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L407 EN**: Declares TableGen def `int_hexagon_V6_vmpyss_parts`.
  **L407 CN**: 声明 TableGen def `int_hexagon_V6_vmpyss_parts`。
- **L408 EN**: Declares TableGen def `int_hexagon_V6_vmpyss_parts_128B`.
  **L408 CN**: 声明 TableGen def `int_hexagon_V6_vmpyss_parts_128B`。
- **L409 EN**: Declares TableGen def `int_hexagon_V6_vmpyuu_parts`.
  **L409 CN**: 声明 TableGen def `int_hexagon_V6_vmpyuu_parts`。
- **L410 EN**: Declares TableGen def `int_hexagon_V6_vmpyuu_parts_128B`.
  **L410 CN**: 声明 TableGen def `int_hexagon_V6_vmpyuu_parts_128B`。
- **L411 EN**: Declares TableGen def `int_hexagon_V6_vmpyus_parts`.
  **L411 CN**: 声明 TableGen def `int_hexagon_V6_vmpyus_parts`。
- **L412 EN**: Declares TableGen def `int_hexagon_V6_vmpyus_parts_128B`.
  **L412 CN**: 声明 TableGen def `int_hexagon_V6_vmpyus_parts_128B`。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L415 EN**: Comment explains nearby logic, invariants, or intent: `Masked vector stores`.
  **L415 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Masked vector stores`。
- **L416 EN**: Separator comment used for visual grouping.
  **L416 CN**: 用于视觉分组的分隔注释。
- **L417 EN**: Comment explains nearby logic, invariants, or intent: `These are all deprecated, the intrinsics matching instruction names`.
  **L417 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These are all deprecated, the intrinsics matching instruction names`。
- **L418 EN**: Comment explains nearby logic, invariants, or intent: `should be used instead, e.g. int_hexagon_V6_vS32b_qpred_ai, etc.`.
  **L418 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`should be used instead, e.g. int_hexagon_V6_vS32b_qpred_ai, etc.`。
- **L419 EN**: Blank line separating nearby declarations or logic blocks.
  **L419 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L420 EN**: Declares class `Hexagon_custom_vms_Intrinsic`.
  **L420 CN**: 声明 class `Hexagon_custom_vms_Intrinsic`。

### Lines 421-440

````tablegen
  : Hexagon_NonGCC_Intrinsic<
       [], [llvm_v64i1_ty,llvm_ptr_ty,llvm_v16i32_ty], [IntrWriteMem]>;

class Hexagon_custom_vms_Intrinsic_128B
  : Hexagon_NonGCC_Intrinsic<
       [], [llvm_v128i1_ty,llvm_ptr_ty,llvm_v32i32_ty], [IntrWriteMem]>;

def int_hexagon_V6_vmaskedstoreq: Hexagon_custom_vms_Intrinsic;
def int_hexagon_V6_vmaskedstorenq: Hexagon_custom_vms_Intrinsic;
def int_hexagon_V6_vmaskedstorentq: Hexagon_custom_vms_Intrinsic;
def int_hexagon_V6_vmaskedstorentnq: Hexagon_custom_vms_Intrinsic;

def int_hexagon_V6_vmaskedstoreq_128B: Hexagon_custom_vms_Intrinsic_128B;
def int_hexagon_V6_vmaskedstorenq_128B: Hexagon_custom_vms_Intrinsic_128B;
def int_hexagon_V6_vmaskedstorentq_128B: Hexagon_custom_vms_Intrinsic_128B;
def int_hexagon_V6_vmaskedstorentnq_128B: Hexagon_custom_vms_Intrinsic_128B;


// Intrinsic for instrumentation based profiling using a custom handler. The
// name of the handler is passed as the first operand to the intrinsic. The
````
- **L421 EN**: Continues the surrounding expression or declaration: `: Hexagon_NonGCC_Intrinsic<`.
  **L421 CN**: 继续构造周围的表达式或声明：`: Hexagon_NonGCC_Intrinsic<`。
- **L422 EN**: Executes a standalone statement or declaration: `[], [llvm_v64i1_ty,llvm_ptr_ty,llvm_v16i32_ty], [IntrWriteMem]>;`.
  **L422 CN**: 执行一条独立语句或声明：`[], [llvm_v64i1_ty,llvm_ptr_ty,llvm_v16i32_ty], [IntrWriteMem]>;`。
- **L423 EN**: Blank line separating nearby declarations or logic blocks.
  **L423 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L424 EN**: Declares class `Hexagon_custom_vms_Intrinsic_128B`.
  **L424 CN**: 声明 class `Hexagon_custom_vms_Intrinsic_128B`。
- **L425 EN**: Continues the surrounding expression or declaration: `: Hexagon_NonGCC_Intrinsic<`.
  **L425 CN**: 继续构造周围的表达式或声明：`: Hexagon_NonGCC_Intrinsic<`。
- **L426 EN**: Executes a standalone statement or declaration: `[], [llvm_v128i1_ty,llvm_ptr_ty,llvm_v32i32_ty], [IntrWriteMem]>;`.
  **L426 CN**: 执行一条独立语句或声明：`[], [llvm_v128i1_ty,llvm_ptr_ty,llvm_v32i32_ty], [IntrWriteMem]>;`。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L428 EN**: Declares TableGen def `int_hexagon_V6_vmaskedstoreq`.
  **L428 CN**: 声明 TableGen def `int_hexagon_V6_vmaskedstoreq`。
- **L429 EN**: Declares TableGen def `int_hexagon_V6_vmaskedstorenq`.
  **L429 CN**: 声明 TableGen def `int_hexagon_V6_vmaskedstorenq`。
- **L430 EN**: Declares TableGen def `int_hexagon_V6_vmaskedstorentq`.
  **L430 CN**: 声明 TableGen def `int_hexagon_V6_vmaskedstorentq`。
- **L431 EN**: Declares TableGen def `int_hexagon_V6_vmaskedstorentnq`.
  **L431 CN**: 声明 TableGen def `int_hexagon_V6_vmaskedstorentnq`。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L433 EN**: Declares TableGen def `int_hexagon_V6_vmaskedstoreq_128B`.
  **L433 CN**: 声明 TableGen def `int_hexagon_V6_vmaskedstoreq_128B`。
- **L434 EN**: Declares TableGen def `int_hexagon_V6_vmaskedstorenq_128B`.
  **L434 CN**: 声明 TableGen def `int_hexagon_V6_vmaskedstorenq_128B`。
- **L435 EN**: Declares TableGen def `int_hexagon_V6_vmaskedstorentq_128B`.
  **L435 CN**: 声明 TableGen def `int_hexagon_V6_vmaskedstorentq_128B`。
- **L436 EN**: Declares TableGen def `int_hexagon_V6_vmaskedstorentnq_128B`.
  **L436 CN**: 声明 TableGen def `int_hexagon_V6_vmaskedstorentnq_128B`。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L439 EN**: Comment explains nearby logic, invariants, or intent: `Intrinsic for instrumentation based profiling using a custom handler. The`.
  **L439 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intrinsic for instrumentation based profiling using a custom handler. The`。
- **L440 EN**: Comment explains nearby logic, invariants, or intent: `name of the handler is passed as the first operand to the intrinsic. The`.
  **L440 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`name of the handler is passed as the first operand to the intrinsic. The`。

### Lines 441-460

````tablegen
// handler can take only one int32 input which is passed as the second
// operand to the intrinsic.
def int_hexagon_instrprof_custom
    : Hexagon_NonGCC_Intrinsic<[],
                               [llvm_ptr_ty, llvm_i32_ty],
                               [IntrInaccessibleMemOnly]>;


include "llvm/IR/IntrinsicsHexagonDep.td"

class Hexagon__ptri32i32v64i16_Intrinsic<string GCCIntSuffix,
      list<IntrinsicProperty> intr_properties = [IntrNoMem]>
  : Hexagon_Intrinsic<GCCIntSuffix,
       [], [llvm_ptr_ty,llvm_i32_ty,llvm_i32_ty,llvm_v64i16_ty],
       intr_properties>;

def int_hexagon_V6_vgather_vscattermh :
Hexagon__ptri32i32v64i16_Intrinsic<"HEXAGON_V6_vgather_vscattermh", [IntrArgMemOnly]>;

def int_hexagon_V6_vgather_vscattermh_128B :
````
- **L441 EN**: Comment explains nearby logic, invariants, or intent: `handler can take only one int32 input which is passed as the second`.
  **L441 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`handler can take only one int32 input which is passed as the second`。
- **L442 EN**: Comment explains nearby logic, invariants, or intent: `operand to the intrinsic.`.
  **L442 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operand to the intrinsic.`。
- **L443 EN**: Declares TableGen def `int_hexagon_instrprof_custom`.
  **L443 CN**: 声明 TableGen def `int_hexagon_instrprof_custom`。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_NonGCC_Intrinsic<[],`.
  **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_NonGCC_Intrinsic<[],`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty],`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty],`。
- **L446 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L446 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L449 EN**: Imports TableGen file "llvm/IR/IntrinsicsHexagonDep.td" to reuse LLVM IR core types, instructions, and ownership utilities.
  **L449 CN**: 导入 TableGen 文件 "llvm/IR/IntrinsicsHexagonDep.td" 以复用LLVM IR 核心类型、指令与所有权辅助组件。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L451 EN**: Declares class `Hexagon__ptri32i32v64i16_Intrinsic<string`.
  **L451 CN**: 声明 class `Hexagon__ptri32i32v64i16_Intrinsic<string`。
- **L452 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> intr_properties = [IntrNoMem]>`.
  **L452 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> intr_properties = [IntrNoMem]>`。
- **L453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Hexagon_Intrinsic<GCCIntSuffix,`.
  **L453 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Hexagon_Intrinsic<GCCIntSuffix,`。
- **L454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_ptr_ty,llvm_i32_ty,llvm_i32_ty,llvm_v64i16_ty],`.
  **L454 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_ptr_ty,llvm_i32_ty,llvm_i32_ty,llvm_v64i16_ty],`。
- **L455 EN**: Executes a standalone statement or declaration: `intr_properties>;`.
  **L455 CN**: 执行一条独立语句或声明：`intr_properties>;`。
- **L456 EN**: Blank line separating nearby declarations or logic blocks.
  **L456 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L457 EN**: Declares TableGen def `int_hexagon_V6_vgather_vscattermh`.
  **L457 CN**: 声明 TableGen def `int_hexagon_V6_vgather_vscattermh`。
- **L458 EN**: Executes a standalone statement or declaration: `Hexagon__ptri32i32v64i16_Intrinsic<"HEXAGON_V6_vgather_vscattermh", [IntrArgMemOnly]>;`.
  **L458 CN**: 执行一条独立语句或声明：`Hexagon__ptri32i32v64i16_Intrinsic<"HEXAGON_V6_vgather_vscattermh", [IntrArgMemOnly]>;`。
- **L459 EN**: Blank line separating nearby declarations or logic blocks.
  **L459 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L460 EN**: Declares TableGen def `int_hexagon_V6_vgather_vscattermh_128B`.
  **L460 CN**: 声明 TableGen def `int_hexagon_V6_vgather_vscattermh_128B`。

### Lines 461-461

````tablegen
Hexagon__ptri32i32v32i32_Intrinsic<"HEXAGON_V6_vgather_vscattermh_128B", [IntrArgMemOnly]>;
````
- **L461 EN**: Executes a standalone statement or declaration: `Hexagon__ptri32i32v32i32_Intrinsic<"HEXAGON_V6_vgather_vscattermh_128B", [IntrArgMemOnly]>;`.
  **L461 CN**: 执行一条独立语句或声明：`Hexagon__ptri32i32v32i32_Intrinsic<"HEXAGON_V6_vgather_vscattermh_128B", [IntrArgMemOnly]>;`。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- `llvm/IR/IntrinsicsHexagonDep.td`: Provides LLVM IR core types, instructions, and ownership utilities. / 提供LLVM IR 核心类型、指令与所有权辅助组件。
