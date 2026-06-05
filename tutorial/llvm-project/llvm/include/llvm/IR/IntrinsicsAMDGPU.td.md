# IntrinsicsAMDGPU.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsAMDGPU.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the R600-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsAMDGPU` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

````tablegen
//===- IntrinsicsAMDGPU.td - Defines AMDGPU intrinsics -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the R600-specific intrinsics.
//
//===----------------------------------------------------------------------===//

def flat_ptr_ty : LLVMQualPointerType<0>;
def global_ptr_ty : LLVMQualPointerType<1>;
def local_ptr_ty : LLVMQualPointerType<3>;

// The amdgpu-no-* attributes (ex amdgpu-no-workitem-id-z) typically inferred
// by the backend cause whole-program undefined behavior when violated, such as
// by causing all other preload register intrinsics to return arbitrarily incorrect
// values. In non-entry-point functions, attempting to call a function that needs
// some preloaded register from a function that is known to not need it is a violation
// of the calling convention and also program-level UB. Outside of such IR-level UB,
// these preloaded registers are always set to a well-defined value and are thus `noundef`.
class AMDGPUReadPreloadRegisterIntrinsic<
    list<IntrinsicProperty> ExtraAttrs = []>
    : DefaultAttrsIntrinsic<[llvm_i32_ty], [],
                            !listconcat([NoUndef<RetIndex>, IntrNoMem,
                                         IntrSpeculatable],
                                        ExtraAttrs)>;

class AMDGPUReadPreloadRegisterIntrinsicNamed<
    string name, list<IntrinsicProperty> ExtraAttrs = []>
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
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the R600-specific intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the R600-specific intrinsics.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Declares TableGen def `flat_ptr_ty`.
  **L13 CN**: 声明 TableGen def `flat_ptr_ty`。
- **L14 EN**: Declares TableGen def `global_ptr_ty`.
  **L14 CN**: 声明 TableGen def `global_ptr_ty`。
- **L15 EN**: Declares TableGen def `local_ptr_ty`.
  **L15 CN**: 声明 TableGen def `local_ptr_ty`。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Comment explains nearby logic, invariants, or intent: `The amdgpu-no-* attributes (ex amdgpu-no-workitem-id-z) typically inferred`.
  **L17 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The amdgpu-no-* attributes (ex amdgpu-no-workitem-id-z) typically inferred`。
- **L18 EN**: Comment explains nearby logic, invariants, or intent: `by the backend cause whole-program undefined behavior when violated, such as`.
  **L18 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`by the backend cause whole-program undefined behavior when violated, such as`。
- **L19 EN**: Comment explains nearby logic, invariants, or intent: `by causing all other preload register intrinsics to return arbitrarily incorrect`.
  **L19 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`by causing all other preload register intrinsics to return arbitrarily incorrect`。
- **L20 EN**: Comment explains nearby logic, invariants, or intent: `values. In non-entry-point functions, attempting to call a function that needs`.
  **L20 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`values. In non-entry-point functions, attempting to call a function that needs`。
- **L21 EN**: Comment explains nearby logic, invariants, or intent: `some preloaded register from a function that is known to not need it is a violation`.
  **L21 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`some preloaded register from a function that is known to not need it is a violation`。
- **L22 EN**: Comment explains nearby logic, invariants, or intent: `of the calling convention and also program-level UB. Outside of such IR-level UB,`.
  **L22 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`of the calling convention and also program-level UB. Outside of such IR-level UB,`。
- **L23 EN**: Comment explains nearby logic, invariants, or intent: `these preloaded registers are always set to a well-defined value and are thus `noundef`.`.
  **L23 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`these preloaded registers are always set to a well-defined value and are thus `noundef`.`。
- **L24 EN**: Declares class `AMDGPUReadPreloadRegisterIntrinsic<`.
  **L24 CN**: 声明 class `AMDGPUReadPreloadRegisterIntrinsic<`。
- **L25 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> ExtraAttrs = []>`.
  **L25 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> ExtraAttrs = []>`。
- **L26 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [],`.
  **L26 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [],`。
- **L27 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([NoUndef<RetIndex>, IntrNoMem,`.
  **L27 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([NoUndef<RetIndex>, IntrNoMem,`。
- **L28 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrSpeculatable],`.
  **L28 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrSpeculatable],`。
- **L29 EN**: Executes a standalone statement or declaration: `ExtraAttrs)>;`.
  **L29 CN**: 执行一条独立语句或声明：`ExtraAttrs)>;`。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L31 EN**: Declares class `AMDGPUReadPreloadRegisterIntrinsicNamed<`.
  **L31 CN**: 声明 class `AMDGPUReadPreloadRegisterIntrinsicNamed<`。
- **L32 EN**: Continues the surrounding expression or declaration: `string name, list<IntrinsicProperty> ExtraAttrs = []>`.
  **L32 CN**: 继续构造周围的表达式或声明：`string name, list<IntrinsicProperty> ExtraAttrs = []>`。

### Lines 33-64

````tablegen
    : AMDGPUReadPreloadRegisterIntrinsic<ExtraAttrs>, ClangBuiltin<name>;

// Used to tag image and resource intrinsics with information used to generate
// mem operands.
class AMDGPURsrcIntrinsic<int rsrcarg, bit isimage = false> {
  int RsrcArg = rsrcarg;
  bit IsImage = isimage;
}

let TargetPrefix = "r600" in {

  multiclass AMDGPUReadPreloadRegisterIntrinsic_xyz<
      list<IntrinsicProperty> ExtraAttrs = []> {
    def _x : AMDGPUReadPreloadRegisterIntrinsic<ExtraAttrs>;
    def _y : AMDGPUReadPreloadRegisterIntrinsic<ExtraAttrs>;
    def _z : AMDGPUReadPreloadRegisterIntrinsic<ExtraAttrs>;
  }

  multiclass AMDGPUReadPreloadRegisterIntrinsic_xyz_named<
      string prefix, list<IntrinsicProperty> ExtraAttrs = []> {
    def _x : AMDGPUReadPreloadRegisterIntrinsicNamed<!strconcat(prefix, "_x"),
                                                     ExtraAttrs>;
    def _y : AMDGPUReadPreloadRegisterIntrinsicNamed<!strconcat(prefix, "_y"),
                                                     ExtraAttrs>;
    def _z : AMDGPUReadPreloadRegisterIntrinsicNamed<!strconcat(prefix, "_z"),
                                                     ExtraAttrs>;
  }

defm int_r600_read_global_size : AMDGPUReadPreloadRegisterIntrinsic_xyz_named
                                 <"__builtin_r600_read_global_size">;
defm int_r600_read_ngroups : AMDGPUReadPreloadRegisterIntrinsic_xyz_named
                             <"__builtin_r600_read_ngroups">;
````
- **L33 EN**: Executes a standalone statement or declaration: `: AMDGPUReadPreloadRegisterIntrinsic<ExtraAttrs>, ClangBuiltin<name>;`.
  **L33 CN**: 执行一条独立语句或声明：`: AMDGPUReadPreloadRegisterIntrinsic<ExtraAttrs>, ClangBuiltin<name>;`。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Comment explains nearby logic, invariants, or intent: `Used to tag image and resource intrinsics with information used to generate`.
  **L35 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Used to tag image and resource intrinsics with information used to generate`。
- **L36 EN**: Comment explains nearby logic, invariants, or intent: `mem operands.`.
  **L36 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mem operands.`。
- **L37 EN**: Declares class `AMDGPURsrcIntrinsic<int`.
  **L37 CN**: 声明 class `AMDGPURsrcIntrinsic<int`。
- **L38 EN**: Initializes variable `RsrcArg` from the right-hand expression.
  **L38 CN**: 使用右侧表达式初始化变量 `RsrcArg`。
- **L39 EN**: Initializes variable `IsImage` from the right-hand expression.
  **L39 CN**: 使用右侧表达式初始化变量 `IsImage`。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L42 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Declares TableGen multiclass `AMDGPUReadPreloadRegisterIntrinsic_xyz`.
  **L44 CN**: 声明 TableGen multiclass `AMDGPUReadPreloadRegisterIntrinsic_xyz`。
- **L45 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> ExtraAttrs = []> {`.
  **L45 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> ExtraAttrs = []> {`。
- **L46 EN**: Declares TableGen def `_x`.
  **L46 CN**: 声明 TableGen def `_x`。
- **L47 EN**: Declares TableGen def `_y`.
  **L47 CN**: 声明 TableGen def `_y`。
- **L48 EN**: Declares TableGen def `_z`.
  **L48 CN**: 声明 TableGen def `_z`。
- **L49 EN**: Closes the current lexical scope or compound statement.
  **L49 CN**: 结束当前词法作用域或复合语句块。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L51 EN**: Declares TableGen multiclass `AMDGPUReadPreloadRegisterIntrinsic_xyz_named`.
  **L51 CN**: 声明 TableGen multiclass `AMDGPUReadPreloadRegisterIntrinsic_xyz_named`。
- **L52 EN**: Continues the surrounding expression or declaration: `string prefix, list<IntrinsicProperty> ExtraAttrs = []> {`.
  **L52 CN**: 继续构造周围的表达式或声明：`string prefix, list<IntrinsicProperty> ExtraAttrs = []> {`。
- **L53 EN**: Declares TableGen def `_x`.
  **L53 CN**: 声明 TableGen def `_x`。
- **L54 EN**: Executes a standalone statement or declaration: `ExtraAttrs>;`.
  **L54 CN**: 执行一条独立语句或声明：`ExtraAttrs>;`。
- **L55 EN**: Declares TableGen def `_y`.
  **L55 CN**: 声明 TableGen def `_y`。
- **L56 EN**: Executes a standalone statement or declaration: `ExtraAttrs>;`.
  **L56 CN**: 执行一条独立语句或声明：`ExtraAttrs>;`。
- **L57 EN**: Declares TableGen def `_z`.
  **L57 CN**: 声明 TableGen def `_z`。
- **L58 EN**: Executes a standalone statement or declaration: `ExtraAttrs>;`.
  **L58 CN**: 执行一条独立语句或声明：`ExtraAttrs>;`。
- **L59 EN**: Closes the current lexical scope or compound statement.
  **L59 CN**: 结束当前词法作用域或复合语句块。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L61 EN**: Declares TableGen defm `int_r600_read_global_size`.
  **L61 CN**: 声明 TableGen defm `int_r600_read_global_size`。
- **L62 EN**: Executes a standalone statement or declaration: `<"__builtin_r600_read_global_size">;`.
  **L62 CN**: 执行一条独立语句或声明：`<"__builtin_r600_read_global_size">;`。
- **L63 EN**: Declares TableGen defm `int_r600_read_ngroups`.
  **L63 CN**: 声明 TableGen defm `int_r600_read_ngroups`。
- **L64 EN**: Executes a standalone statement or declaration: `<"__builtin_r600_read_ngroups">;`.
  **L64 CN**: 执行一条独立语句或声明：`<"__builtin_r600_read_ngroups">;`。

### Lines 65-96

````tablegen
defm int_r600_read_tgid : AMDGPUReadPreloadRegisterIntrinsic_xyz_named
                          <"__builtin_r600_read_tgid">;

defm int_r600_read_local_size : AMDGPUReadPreloadRegisterIntrinsic_xyz;
defm int_r600_read_tidig
    : AMDGPUReadPreloadRegisterIntrinsic_xyz_named<
          "__builtin_r600_read_tidig", [Range<RetIndex, 0, 1024>]>;

def int_r600_group_barrier : ClangBuiltin<"__builtin_r600_group_barrier">,
  Intrinsic<[], [], [IntrConvergent, IntrWillReturn]>;

// AS 7 is PARAM_I_ADDRESS, used for kernel arguments
def int_r600_implicitarg_ptr :
  ClangBuiltin<"__builtin_r600_implicitarg_ptr">,
  DefaultAttrsIntrinsic<[LLVMQualPointerType<7>], [],
  [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;

def int_r600_rat_store_typed :
  // 1st parameter: Data
  // 2nd parameter: Index
  // 3rd parameter: Constant RAT ID
  DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty], []>,
  ClangBuiltin<"__builtin_r600_rat_store_typed">;

def int_r600_recipsqrt_ieee :  DefaultAttrsIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable]
>;

def int_r600_recipsqrt_clamped : DefaultAttrsIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable]
>;

````
- **L65 EN**: Declares TableGen defm `int_r600_read_tgid`.
  **L65 CN**: 声明 TableGen defm `int_r600_read_tgid`。
- **L66 EN**: Executes a standalone statement or declaration: `<"__builtin_r600_read_tgid">;`.
  **L66 CN**: 执行一条独立语句或声明：`<"__builtin_r600_read_tgid">;`。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L68 EN**: Declares TableGen defm `int_r600_read_local_size`.
  **L68 CN**: 声明 TableGen defm `int_r600_read_local_size`。
- **L69 EN**: Declares TableGen defm `int_r600_read_tidig`.
  **L69 CN**: 声明 TableGen defm `int_r600_read_tidig`。
- **L70 EN**: Continues the surrounding expression or declaration: `: AMDGPUReadPreloadRegisterIntrinsic_xyz_named<`.
  **L70 CN**: 继续构造周围的表达式或声明：`: AMDGPUReadPreloadRegisterIntrinsic_xyz_named<`。
- **L71 EN**: Executes a standalone statement or declaration: `"__builtin_r600_read_tidig", [Range<RetIndex, 0, 1024>]>;`.
  **L71 CN**: 执行一条独立语句或声明：`"__builtin_r600_read_tidig", [Range<RetIndex, 0, 1024>]>;`。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L73 EN**: Declares TableGen def `int_r600_group_barrier`.
  **L73 CN**: 声明 TableGen def `int_r600_group_barrier`。
- **L74 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], [IntrConvergent, IntrWillReturn]>;`.
  **L74 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], [IntrConvergent, IntrWillReturn]>;`。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L76 EN**: Comment explains nearby logic, invariants, or intent: `AS 7 is PARAM_I_ADDRESS, used for kernel arguments`.
  **L76 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AS 7 is PARAM_I_ADDRESS, used for kernel arguments`。
- **L77 EN**: Declares TableGen def `int_r600_implicitarg_ptr`.
  **L77 CN**: 声明 TableGen def `int_r600_implicitarg_ptr`。
- **L78 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_r600_implicitarg_ptr">,`.
  **L78 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_r600_implicitarg_ptr">,`。
- **L79 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[LLVMQualPointerType<7>], [],`.
  **L79 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[LLVMQualPointerType<7>], [],`。
- **L80 EN**: Executes a standalone statement or declaration: `[NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`.
  **L80 CN**: 执行一条独立语句或声明：`[NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Declares TableGen def `int_r600_rat_store_typed`.
  **L82 CN**: 声明 TableGen def `int_r600_rat_store_typed`。
- **L83 EN**: Comment explains nearby logic, invariants, or intent: `1st parameter: Data`.
  **L83 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`1st parameter: Data`。
- **L84 EN**: Comment explains nearby logic, invariants, or intent: `2nd parameter: Index`.
  **L84 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2nd parameter: Index`。
- **L85 EN**: Comment explains nearby logic, invariants, or intent: `3rd parameter: Constant RAT ID`.
  **L85 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`3rd parameter: Constant RAT ID`。
- **L86 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty], []>,`.
  **L86 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty], []>,`。
- **L87 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_r600_rat_store_typed">;`.
  **L87 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_r600_rat_store_typed">;`。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L89 EN**: Declares TableGen def `int_r600_recipsqrt_ieee`.
  **L89 CN**: 声明 TableGen def `int_r600_recipsqrt_ieee`。
- **L90 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable]`.
  **L90 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable]`。
- **L91 EN**: Executes a standalone statement or declaration: `>;`.
  **L91 CN**: 执行一条独立语句或声明：`>;`。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Declares TableGen def `int_r600_recipsqrt_clamped`.
  **L93 CN**: 声明 TableGen def `int_r600_recipsqrt_clamped`。
- **L94 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable]`.
  **L94 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable]`。
- **L95 EN**: Executes a standalone statement or declaration: `>;`.
  **L95 CN**: 执行一条独立语句或声明：`>;`。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 97-128

````tablegen
def int_r600_cube : DefaultAttrsIntrinsic<
  [llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem, IntrSpeculatable]
>;

def int_r600_store_stream_output : DefaultAttrsIntrinsic<
  [], [llvm_v4f32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []
>;

class TextureIntrinsicFloatInput : DefaultAttrsIntrinsic<[llvm_v4f32_ty], [
  llvm_v4f32_ty, // Coord
  llvm_i32_ty,   // offset_x
  llvm_i32_ty,   // offset_y,
  llvm_i32_ty,   // offset_z,
  llvm_i32_ty,   // resource_id
  llvm_i32_ty,   // samplerid
  llvm_i32_ty,   // coord_type_x
  llvm_i32_ty,   // coord_type_y
  llvm_i32_ty,   // coord_type_z
  llvm_i32_ty],  // coord_type_w
  [IntrNoMem]
>;

class TextureIntrinsicInt32Input : DefaultAttrsIntrinsic<[llvm_v4i32_ty], [
    llvm_v4i32_ty, // Coord
    llvm_i32_ty,   // offset_x
    llvm_i32_ty,   // offset_y,
    llvm_i32_ty,   // offset_z,
    llvm_i32_ty,   // resource_id
    llvm_i32_ty,   // samplerid
    llvm_i32_ty,   // coord_type_x
    llvm_i32_ty,   // coord_type_y
    llvm_i32_ty,   // coord_type_z
````
- **L97 EN**: Declares TableGen def `int_r600_cube`.
  **L97 CN**: 声明 TableGen def `int_r600_cube`。
- **L98 EN**: Continues the surrounding expression or declaration: `[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem, IntrSpeculatable]`.
  **L98 CN**: 继续构造周围的表达式或声明：`[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem, IntrSpeculatable]`。
- **L99 EN**: Executes a standalone statement or declaration: `>;`.
  **L99 CN**: 执行一条独立语句或声明：`>;`。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L101 EN**: Declares TableGen def `int_r600_store_stream_output`.
  **L101 CN**: 声明 TableGen def `int_r600_store_stream_output`。
- **L102 EN**: Continues the surrounding expression or declaration: `[], [llvm_v4f32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []`.
  **L102 CN**: 继续构造周围的表达式或声明：`[], [llvm_v4f32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []`。
- **L103 EN**: Executes a standalone statement or declaration: `>;`.
  **L103 CN**: 执行一条独立语句或声明：`>;`。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L105 EN**: Declares class `TextureIntrinsicFloatInput`.
  **L105 CN**: 声明 class `TextureIntrinsicFloatInput`。
- **L106 EN**: Continues the surrounding expression or declaration: `llvm_v4f32_ty, // Coord`.
  **L106 CN**: 继续构造周围的表达式或声明：`llvm_v4f32_ty, // Coord`。
- **L107 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // offset_x`.
  **L107 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // offset_x`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,   // offset_y,`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,   // offset_y,`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,   // offset_z,`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,   // offset_z,`。
- **L110 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // resource_id`.
  **L110 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // resource_id`。
- **L111 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // samplerid`.
  **L111 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // samplerid`。
- **L112 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // coord_type_x`.
  **L112 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // coord_type_x`。
- **L113 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // coord_type_y`.
  **L113 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // coord_type_y`。
- **L114 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // coord_type_z`.
  **L114 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // coord_type_z`。
- **L115 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],  // coord_type_w`.
  **L115 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],  // coord_type_w`。
- **L116 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]`.
  **L116 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]`。
- **L117 EN**: Executes a standalone statement or declaration: `>;`.
  **L117 CN**: 执行一条独立语句或声明：`>;`。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L119 EN**: Declares class `TextureIntrinsicInt32Input`.
  **L119 CN**: 声明 class `TextureIntrinsicInt32Input`。
- **L120 EN**: Continues the surrounding expression or declaration: `llvm_v4i32_ty, // Coord`.
  **L120 CN**: 继续构造周围的表达式或声明：`llvm_v4i32_ty, // Coord`。
- **L121 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // offset_x`.
  **L121 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // offset_x`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,   // offset_y,`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,   // offset_y,`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,   // offset_z,`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,   // offset_z,`。
- **L124 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // resource_id`.
  **L124 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // resource_id`。
- **L125 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // samplerid`.
  **L125 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // samplerid`。
- **L126 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // coord_type_x`.
  **L126 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // coord_type_x`。
- **L127 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // coord_type_y`.
  **L127 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // coord_type_y`。
- **L128 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // coord_type_z`.
  **L128 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // coord_type_z`。

### Lines 129-160

````tablegen
    llvm_i32_ty],  // coord_type_w
    [IntrNoMem]
>;

def int_r600_store_swizzle :
  Intrinsic<[], [llvm_v4f32_ty, llvm_i32_ty, llvm_i32_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

def int_r600_tex : TextureIntrinsicFloatInput;
def int_r600_texc : TextureIntrinsicFloatInput;
def int_r600_txl : TextureIntrinsicFloatInput;
def int_r600_txlc : TextureIntrinsicFloatInput;
def int_r600_txb : TextureIntrinsicFloatInput;
def int_r600_txbc : TextureIntrinsicFloatInput;
def int_r600_txf : TextureIntrinsicInt32Input;
def int_r600_txq : TextureIntrinsicInt32Input;
def int_r600_ddx : TextureIntrinsicFloatInput;
def int_r600_ddy : TextureIntrinsicFloatInput;

def int_r600_dot4 : DefaultAttrsIntrinsic<[llvm_float_ty],
  [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem, IntrSpeculatable]
>;

def int_r600_kill : DefaultAttrsIntrinsic<[], [llvm_float_ty], []>;

} // End TargetPrefix = "r600"

let TargetPrefix = "amdgcn" in {

//===----------------------------------------------------------------------===//
// ABI Special Intrinsics
//===----------------------------------------------------------------------===//
````
- **L129 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],  // coord_type_w`.
  **L129 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],  // coord_type_w`。
- **L130 EN**: Continues the surrounding expression or declaration: `[IntrNoMem]`.
  **L130 CN**: 继续构造周围的表达式或声明：`[IntrNoMem]`。
- **L131 EN**: Executes a standalone statement or declaration: `>;`.
  **L131 CN**: 执行一条独立语句或声明：`>;`。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L133 EN**: Declares TableGen def `int_r600_store_swizzle`.
  **L133 CN**: 声明 TableGen def `int_r600_store_swizzle`。
- **L134 EN**: Continues the surrounding expression or declaration: `Intrinsic<[], [llvm_v4f32_ty, llvm_i32_ty, llvm_i32_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L134 CN**: 继续构造周围的表达式或声明：`Intrinsic<[], [llvm_v4f32_ty, llvm_i32_ty, llvm_i32_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L135 EN**: Executes a standalone statement or declaration: `>;`.
  **L135 CN**: 执行一条独立语句或声明：`>;`。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L137 EN**: Declares TableGen def `int_r600_tex`.
  **L137 CN**: 声明 TableGen def `int_r600_tex`。
- **L138 EN**: Declares TableGen def `int_r600_texc`.
  **L138 CN**: 声明 TableGen def `int_r600_texc`。
- **L139 EN**: Declares TableGen def `int_r600_txl`.
  **L139 CN**: 声明 TableGen def `int_r600_txl`。
- **L140 EN**: Declares TableGen def `int_r600_txlc`.
  **L140 CN**: 声明 TableGen def `int_r600_txlc`。
- **L141 EN**: Declares TableGen def `int_r600_txb`.
  **L141 CN**: 声明 TableGen def `int_r600_txb`。
- **L142 EN**: Declares TableGen def `int_r600_txbc`.
  **L142 CN**: 声明 TableGen def `int_r600_txbc`。
- **L143 EN**: Declares TableGen def `int_r600_txf`.
  **L143 CN**: 声明 TableGen def `int_r600_txf`。
- **L144 EN**: Declares TableGen def `int_r600_txq`.
  **L144 CN**: 声明 TableGen def `int_r600_txq`。
- **L145 EN**: Declares TableGen def `int_r600_ddx`.
  **L145 CN**: 声明 TableGen def `int_r600_ddx`。
- **L146 EN**: Declares TableGen def `int_r600_ddy`.
  **L146 CN**: 声明 TableGen def `int_r600_ddy`。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L148 EN**: Declares TableGen def `int_r600_dot4`.
  **L148 CN**: 声明 TableGen def `int_r600_dot4`。
- **L149 EN**: Continues the surrounding expression or declaration: `[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem, IntrSpeculatable]`.
  **L149 CN**: 继续构造周围的表达式或声明：`[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem, IntrSpeculatable]`。
- **L150 EN**: Executes a standalone statement or declaration: `>;`.
  **L150 CN**: 执行一条独立语句或声明：`>;`。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L152 EN**: Declares TableGen def `int_r600_kill`.
  **L152 CN**: 声明 TableGen def `int_r600_kill`。
- **L153 EN**: Blank line separating nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L154 EN**: Continues the surrounding expression or declaration: `} // End TargetPrefix = "r600"`.
  **L154 CN**: 继续构造周围的表达式或声明：`} // End TargetPrefix = "r600"`。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L156 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L156 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L158 EN**: Banner comment marking a file or section boundary.
  **L158 CN**: 横幅注释，用于标记文件或章节边界。
- **L159 EN**: Comment explains nearby logic, invariants, or intent: `ABI Special Intrinsics`.
  **L159 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ABI Special Intrinsics`。
- **L160 EN**: Banner comment marking a file or section boundary.
  **L160 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 161-192

````tablegen

defm int_amdgcn_workitem_id
    : AMDGPUReadPreloadRegisterIntrinsic_xyz_named<
          "__builtin_amdgcn_workitem_id", [Range<RetIndex, 0, 1024>]>;

defm int_amdgcn_workgroup_id : AMDGPUReadPreloadRegisterIntrinsic_xyz_named
                               <"__builtin_amdgcn_workgroup_id">;
defm int_amdgcn_cluster_id : AMDGPUReadPreloadRegisterIntrinsic_xyz_named
                             <"__builtin_amdgcn_cluster_id">;
defm int_amdgcn_cluster_workgroup_id : AMDGPUReadPreloadRegisterIntrinsic_xyz_named
                                       <"__builtin_amdgcn_cluster_workgroup_id">;
def int_amdgcn_cluster_workgroup_flat_id:
  ClangBuiltin<"__builtin_amdgcn_cluster_workgroup_flat_id">,
  Intrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrSpeculatable]>;
defm int_amdgcn_cluster_workgroup_max_id : AMDGPUReadPreloadRegisterIntrinsic_xyz_named
                                           <"__builtin_amdgcn_cluster_workgroup_max_id">;
def int_amdgcn_cluster_workgroup_max_flat_id:
  ClangBuiltin<"__builtin_amdgcn_cluster_workgroup_max_flat_id">,
  Intrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrSpeculatable]>;

def int_amdgcn_dispatch_ptr :
  DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],
  [Align<RetIndex, 4>, Dereferenceable<RetIndex, 64>, NoUndef<RetIndex>, NonNull<RetIndex>, IntrNoMem, IntrSpeculatable]>;

def int_amdgcn_queue_ptr :
  ClangBuiltin<"__builtin_amdgcn_queue_ptr">,
  DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],
  [Align<RetIndex, 4>, NoUndef<RetIndex>, NonNull<RetIndex>, IntrNoMem, IntrSpeculatable]>;

def int_amdgcn_kernarg_segment_ptr :
  ClangBuiltin<"__builtin_amdgcn_kernarg_segment_ptr">,
  DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],
````
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L162 EN**: Declares TableGen defm `int_amdgcn_workitem_id`.
  **L162 CN**: 声明 TableGen defm `int_amdgcn_workitem_id`。
- **L163 EN**: Continues the surrounding expression or declaration: `: AMDGPUReadPreloadRegisterIntrinsic_xyz_named<`.
  **L163 CN**: 继续构造周围的表达式或声明：`: AMDGPUReadPreloadRegisterIntrinsic_xyz_named<`。
- **L164 EN**: Executes a standalone statement or declaration: `"__builtin_amdgcn_workitem_id", [Range<RetIndex, 0, 1024>]>;`.
  **L164 CN**: 执行一条独立语句或声明：`"__builtin_amdgcn_workitem_id", [Range<RetIndex, 0, 1024>]>;`。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L166 EN**: Declares TableGen defm `int_amdgcn_workgroup_id`.
  **L166 CN**: 声明 TableGen defm `int_amdgcn_workgroup_id`。
- **L167 EN**: Executes a standalone statement or declaration: `<"__builtin_amdgcn_workgroup_id">;`.
  **L167 CN**: 执行一条独立语句或声明：`<"__builtin_amdgcn_workgroup_id">;`。
- **L168 EN**: Declares TableGen defm `int_amdgcn_cluster_id`.
  **L168 CN**: 声明 TableGen defm `int_amdgcn_cluster_id`。
- **L169 EN**: Executes a standalone statement or declaration: `<"__builtin_amdgcn_cluster_id">;`.
  **L169 CN**: 执行一条独立语句或声明：`<"__builtin_amdgcn_cluster_id">;`。
- **L170 EN**: Declares TableGen defm `int_amdgcn_cluster_workgroup_id`.
  **L170 CN**: 声明 TableGen defm `int_amdgcn_cluster_workgroup_id`。
- **L171 EN**: Executes a standalone statement or declaration: `<"__builtin_amdgcn_cluster_workgroup_id">;`.
  **L171 CN**: 执行一条独立语句或声明：`<"__builtin_amdgcn_cluster_workgroup_id">;`。
- **L172 EN**: Declares TableGen def `int_amdgcn_cluster_workgroup_flat_id`.
  **L172 CN**: 声明 TableGen def `int_amdgcn_cluster_workgroup_flat_id`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_cluster_workgroup_flat_id">,`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_cluster_workgroup_flat_id">,`。
- **L174 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrSpeculatable]>;`.
  **L174 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrSpeculatable]>;`。
- **L175 EN**: Declares TableGen defm `int_amdgcn_cluster_workgroup_max_id`.
  **L175 CN**: 声明 TableGen defm `int_amdgcn_cluster_workgroup_max_id`。
- **L176 EN**: Executes a standalone statement or declaration: `<"__builtin_amdgcn_cluster_workgroup_max_id">;`.
  **L176 CN**: 执行一条独立语句或声明：`<"__builtin_amdgcn_cluster_workgroup_max_id">;`。
- **L177 EN**: Declares TableGen def `int_amdgcn_cluster_workgroup_max_flat_id`.
  **L177 CN**: 声明 TableGen def `int_amdgcn_cluster_workgroup_max_flat_id`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_cluster_workgroup_max_flat_id">,`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_cluster_workgroup_max_flat_id">,`。
- **L179 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrSpeculatable]>;`.
  **L179 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrSpeculatable]>;`。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L181 EN**: Declares TableGen def `int_amdgcn_dispatch_ptr`.
  **L181 CN**: 声明 TableGen def `int_amdgcn_dispatch_ptr`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`。
- **L183 EN**: Executes a standalone statement or declaration: `[Align<RetIndex, 4>, Dereferenceable<RetIndex, 64>, NoUndef<RetIndex>, NonNull<RetIndex>, IntrNoMem, IntrSpeculatable]>;`.
  **L183 CN**: 执行一条独立语句或声明：`[Align<RetIndex, 4>, Dereferenceable<RetIndex, 64>, NoUndef<RetIndex>, NonNull<RetIndex>, IntrNoMem, IntrSpeculatable]>;`。
- **L184 EN**: Blank line separating nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L185 EN**: Declares TableGen def `int_amdgcn_queue_ptr`.
  **L185 CN**: 声明 TableGen def `int_amdgcn_queue_ptr`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_queue_ptr">,`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_queue_ptr">,`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`。
- **L188 EN**: Executes a standalone statement or declaration: `[Align<RetIndex, 4>, NoUndef<RetIndex>, NonNull<RetIndex>, IntrNoMem, IntrSpeculatable]>;`.
  **L188 CN**: 执行一条独立语句或声明：`[Align<RetIndex, 4>, NoUndef<RetIndex>, NonNull<RetIndex>, IntrNoMem, IntrSpeculatable]>;`。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L190 EN**: Declares TableGen def `int_amdgcn_kernarg_segment_ptr`.
  **L190 CN**: 声明 TableGen def `int_amdgcn_kernarg_segment_ptr`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_kernarg_segment_ptr">,`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_kernarg_segment_ptr">,`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`。

### Lines 193-224

````tablegen
  [Align<RetIndex, 4>, NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;

def int_amdgcn_implicitarg_ptr :
  ClangBuiltin<"__builtin_amdgcn_implicitarg_ptr">,
  DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],
  [Align<RetIndex, 4>, NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;

// Returns the amount of LDS statically allocated for this program.
// This is no longer guaranteed to be a compile-time constant due to linking
// support.
def int_amdgcn_groupstaticsize :
  ClangBuiltin<"__builtin_amdgcn_groupstaticsize">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;

def int_amdgcn_dispatch_id :
  ClangBuiltin<"__builtin_amdgcn_dispatch_id">,
  DefaultAttrsIntrinsic<[llvm_i64_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;

// For internal use. Coordinates LDS lowering between IR transform and backend.
def int_amdgcn_lds_kernel_id :
  DefaultAttrsIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;

def int_amdgcn_implicit_buffer_ptr :
  ClangBuiltin<"__builtin_amdgcn_implicit_buffer_ptr">,
  DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],
  [Align<RetIndex, 4>, NoUndef<RetIndex>,
    IntrNoMem, IntrSpeculatable]>;

// Set EXEC to the 64-bit value given.
// This is always moved to the beginning of the basic block.
// FIXME: Should be mangled for wave size.
def int_amdgcn_init_exec : Intrinsic<[],
````
- **L193 EN**: Executes a standalone statement or declaration: `[Align<RetIndex, 4>, NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`.
  **L193 CN**: 执行一条独立语句或声明：`[Align<RetIndex, 4>, NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L195 EN**: Declares TableGen def `int_amdgcn_implicitarg_ptr`.
  **L195 CN**: 声明 TableGen def `int_amdgcn_implicitarg_ptr`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_implicitarg_ptr">,`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_implicitarg_ptr">,`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`。
- **L198 EN**: Executes a standalone statement or declaration: `[Align<RetIndex, 4>, NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`.
  **L198 CN**: 执行一条独立语句或声明：`[Align<RetIndex, 4>, NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L200 EN**: Comment explains nearby logic, invariants, or intent: `Returns the amount of LDS statically allocated for this program.`.
  **L200 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns the amount of LDS statically allocated for this program.`。
- **L201 EN**: Comment explains nearby logic, invariants, or intent: `This is no longer guaranteed to be a compile-time constant due to linking`.
  **L201 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is no longer guaranteed to be a compile-time constant due to linking`。
- **L202 EN**: Comment explains nearby logic, invariants, or intent: `support.`.
  **L202 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`support.`。
- **L203 EN**: Declares TableGen def `int_amdgcn_groupstaticsize`.
  **L203 CN**: 声明 TableGen def `int_amdgcn_groupstaticsize`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_groupstaticsize">,`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_groupstaticsize">,`。
- **L205 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`.
  **L205 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L207 EN**: Declares TableGen def `int_amdgcn_dispatch_id`.
  **L207 CN**: 声明 TableGen def `int_amdgcn_dispatch_id`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_dispatch_id">,`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_dispatch_id">,`。
- **L209 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i64_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`.
  **L209 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i64_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Comment explains nearby logic, invariants, or intent: `For internal use. Coordinates LDS lowering between IR transform and backend.`.
  **L211 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For internal use. Coordinates LDS lowering between IR transform and backend.`。
- **L212 EN**: Declares TableGen def `int_amdgcn_lds_kernel_id`.
  **L212 CN**: 声明 TableGen def `int_amdgcn_lds_kernel_id`。
- **L213 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`.
  **L213 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L215 EN**: Declares TableGen def `int_amdgcn_implicit_buffer_ptr`.
  **L215 CN**: 声明 TableGen def `int_amdgcn_implicit_buffer_ptr`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_implicit_buffer_ptr">,`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_implicit_buffer_ptr">,`。
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[LLVMQualPointerType<4>], [],`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Align<RetIndex, 4>, NoUndef<RetIndex>,`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Align<RetIndex, 4>, NoUndef<RetIndex>,`。
- **L219 EN**: Executes a standalone statement or declaration: `IntrNoMem, IntrSpeculatable]>;`.
  **L219 CN**: 执行一条独立语句或声明：`IntrNoMem, IntrSpeculatable]>;`。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L221 EN**: Comment explains nearby logic, invariants, or intent: `Set EXEC to the 64-bit value given.`.
  **L221 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Set EXEC to the 64-bit value given.`。
- **L222 EN**: Comment explains nearby logic, invariants, or intent: `This is always moved to the beginning of the basic block.`.
  **L222 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is always moved to the beginning of the basic block.`。
- **L223 EN**: Comment records a pending task or caution: `FIXME: Should be mangled for wave size.`.
  **L223 CN**: 注释记录了待办事项或注意点：`FIXME: Should be mangled for wave size.`。
- **L224 EN**: Declares TableGen def `int_amdgcn_init_exec`.
  **L224 CN**: 声明 TableGen def `int_amdgcn_init_exec`。

### Lines 225-256

````tablegen
  [llvm_i64_ty],      // 64-bit literal constant
  [IntrConvergent, IntrNoMem, IntrHasSideEffects, IntrNoCallback,
   IntrNoFree, IntrWillReturn, ImmArg<ArgIndex<0>>]>;

// Set EXEC according to a thread count packed in an SGPR input:
//    thread_count = (input >> bitoffset) & 0x7f;
// This is always moved to the beginning of the basic block.
// Note: only inreg arguments to the parent function are valid as
// inputs to this intrinsic, computed values cannot be used.
def int_amdgcn_init_exec_from_input : Intrinsic<[],
  [llvm_i32_ty,       // 32-bit SGPR input
   llvm_i32_ty],      // bit offset of the thread count
  [IntrConvergent, IntrHasSideEffects, IntrNoMem, IntrNoCallback,
   IntrNoFree, IntrWillReturn, ImmArg<ArgIndex<1>>]>;

// Sets the function into whole-wave-mode and returns whether the lane was
// active when entering the function. A branch depending on this return will
// revert the EXEC mask to what it was when entering the function, thus
// resulting in a no-op. This pattern is used to optimize branches when function
// tails need to be run in whole-wave-mode. It may also have other consequences
// (mostly related to WWM CSR handling) that differentiate it from using
// a plain `amdgcn.init.exec -1`.
def int_amdgcn_init_whole_wave : Intrinsic<[llvm_i1_ty], [], [
    IntrHasSideEffects, IntrNoMem, IntrConvergent]>;

def int_amdgcn_wavefrontsize
    : ClangBuiltin<"__builtin_amdgcn_wavefrontsize">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [],
                            [NoUndef<RetIndex>, Range<RetIndex, 32, 65>,
                             IntrNoMem, IntrSpeculatable]>;

// Represent a relocation constant.
````
- **L225 EN**: Continues the surrounding expression or declaration: `[llvm_i64_ty],      // 64-bit literal constant`.
  **L225 CN**: 继续构造周围的表达式或声明：`[llvm_i64_ty],      // 64-bit literal constant`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrNoMem, IntrHasSideEffects, IntrNoCallback,`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrNoMem, IntrHasSideEffects, IntrNoCallback,`。
- **L227 EN**: Executes a standalone statement or declaration: `IntrNoFree, IntrWillReturn, ImmArg<ArgIndex<0>>]>;`.
  **L227 CN**: 执行一条独立语句或声明：`IntrNoFree, IntrWillReturn, ImmArg<ArgIndex<0>>]>;`。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L229 EN**: Comment explains nearby logic, invariants, or intent: `Set EXEC according to a thread count packed in an SGPR input:`.
  **L229 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Set EXEC according to a thread count packed in an SGPR input:`。
- **L230 EN**: Comment explains nearby logic, invariants, or intent: `thread_count = (input >> bitoffset) & 0x7f;`.
  **L230 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`thread_count = (input >> bitoffset) & 0x7f;`。
- **L231 EN**: Comment explains nearby logic, invariants, or intent: `This is always moved to the beginning of the basic block.`.
  **L231 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is always moved to the beginning of the basic block.`。
- **L232 EN**: Comment explains nearby logic, invariants, or intent: `Note: only inreg arguments to the parent function are valid as`.
  **L232 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: only inreg arguments to the parent function are valid as`。
- **L233 EN**: Comment explains nearby logic, invariants, or intent: `inputs to this intrinsic, computed values cannot be used.`.
  **L233 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`inputs to this intrinsic, computed values cannot be used.`。
- **L234 EN**: Declares TableGen def `int_amdgcn_init_exec_from_input`.
  **L234 CN**: 声明 TableGen def `int_amdgcn_init_exec_from_input`。
- **L235 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty,       // 32-bit SGPR input`.
  **L235 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty,       // 32-bit SGPR input`。
- **L236 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],      // bit offset of the thread count`.
  **L236 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],      // bit offset of the thread count`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrHasSideEffects, IntrNoMem, IntrNoCallback,`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrHasSideEffects, IntrNoMem, IntrNoCallback,`。
- **L238 EN**: Executes a standalone statement or declaration: `IntrNoFree, IntrWillReturn, ImmArg<ArgIndex<1>>]>;`.
  **L238 CN**: 执行一条独立语句或声明：`IntrNoFree, IntrWillReturn, ImmArg<ArgIndex<1>>]>;`。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L240 EN**: Comment explains nearby logic, invariants, or intent: `Sets the function into whole-wave-mode and returns whether the lane was`.
  **L240 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sets the function into whole-wave-mode and returns whether the lane was`。
- **L241 EN**: Comment explains nearby logic, invariants, or intent: `active when entering the function. A branch depending on this return will`.
  **L241 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`active when entering the function. A branch depending on this return will`。
- **L242 EN**: Comment explains nearby logic, invariants, or intent: `revert the EXEC mask to what it was when entering the function, thus`.
  **L242 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`revert the EXEC mask to what it was when entering the function, thus`。
- **L243 EN**: Comment explains nearby logic, invariants, or intent: `resulting in a no-op. This pattern is used to optimize branches when function`.
  **L243 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`resulting in a no-op. This pattern is used to optimize branches when function`。
- **L244 EN**: Comment explains nearby logic, invariants, or intent: `tails need to be run in whole-wave-mode. It may also have other consequences`.
  **L244 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tails need to be run in whole-wave-mode. It may also have other consequences`。
- **L245 EN**: Comment explains nearby logic, invariants, or intent: `(mostly related to WWM CSR handling) that differentiate it from using`.
  **L245 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(mostly related to WWM CSR handling) that differentiate it from using`。
- **L246 EN**: Comment explains nearby logic, invariants, or intent: `a plain `amdgcn.init.exec -1`.`.
  **L246 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a plain `amdgcn.init.exec -1`.`。
- **L247 EN**: Declares TableGen def `int_amdgcn_init_whole_wave`.
  **L247 CN**: 声明 TableGen def `int_amdgcn_init_whole_wave`。
- **L248 EN**: Executes a standalone statement or declaration: `IntrHasSideEffects, IntrNoMem, IntrConvergent]>;`.
  **L248 CN**: 执行一条独立语句或声明：`IntrHasSideEffects, IntrNoMem, IntrConvergent]>;`。
- **L249 EN**: Blank line separating nearby declarations or logic blocks.
  **L249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L250 EN**: Declares TableGen def `int_amdgcn_wavefrontsize`.
  **L250 CN**: 声明 TableGen def `int_amdgcn_wavefrontsize`。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_amdgcn_wavefrontsize">,`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_amdgcn_wavefrontsize">,`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [],`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [],`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoUndef<RetIndex>, Range<RetIndex, 32, 65>,`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoUndef<RetIndex>, Range<RetIndex, 32, 65>,`。
- **L254 EN**: Executes a standalone statement or declaration: `IntrNoMem, IntrSpeculatable]>;`.
  **L254 CN**: 执行一条独立语句或声明：`IntrNoMem, IntrSpeculatable]>;`。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L256 EN**: Comment explains nearby logic, invariants, or intent: `Represent a relocation constant.`.
  **L256 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Represent a relocation constant.`。

### Lines 257-288

````tablegen
def int_amdgcn_reloc_constant : DefaultAttrsIntrinsic<
  [llvm_i32_ty], [llvm_metadata_ty],
  [IntrNoMem, IntrSpeculatable]
>;

//===----------------------------------------------------------------------===//
// Instruction Intrinsics
//===----------------------------------------------------------------------===//

// The first parameter is s_sendmsg immediate (i16),
// the second one is copied to m0
def int_amdgcn_s_sendmsg : ClangBuiltin<"__builtin_amdgcn_s_sendmsg">,
  Intrinsic <[], [llvm_i32_ty, llvm_i32_ty],
  [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback]>;
def int_amdgcn_s_sendmsghalt : ClangBuiltin<"__builtin_amdgcn_s_sendmsghalt">,
  Intrinsic <[], [llvm_i32_ty, llvm_i32_ty],
  [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrNoCallback]>;


// gfx11 intrinsic
// The first parameter is s_sendmsg immediate (i16). Return type is i32 or i64.
def int_amdgcn_s_sendmsg_rtn : Intrinsic <[llvm_anyint_ty], [llvm_i32_ty],
  [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback]>;

// Vanilla workgroup sync-barrier
def int_amdgcn_s_barrier : ClangBuiltin<"__builtin_amdgcn_s_barrier">,
  Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// Lower-level split-barrier intrinsics

// void @llvm.amdgcn.s.barrier.signal(i32 %barrierType)
// only for non-named barrier
````
- **L257 EN**: Declares TableGen def `int_amdgcn_reloc_constant`.
  **L257 CN**: 声明 TableGen def `int_amdgcn_reloc_constant`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty], [llvm_metadata_ty],`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty], [llvm_metadata_ty],`。
- **L259 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrSpeculatable]`.
  **L259 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrSpeculatable]`。
- **L260 EN**: Executes a standalone statement or declaration: `>;`.
  **L260 CN**: 执行一条独立语句或声明：`>;`。
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L262 EN**: Banner comment marking a file or section boundary.
  **L262 CN**: 横幅注释，用于标记文件或章节边界。
- **L263 EN**: Comment explains nearby logic, invariants, or intent: `Instruction Intrinsics`.
  **L263 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Instruction Intrinsics`。
- **L264 EN**: Banner comment marking a file or section boundary.
  **L264 CN**: 横幅注释，用于标记文件或章节边界。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Comment explains nearby logic, invariants, or intent: `The first parameter is s_sendmsg immediate (i16),`.
  **L266 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The first parameter is s_sendmsg immediate (i16),`。
- **L267 EN**: Comment explains nearby logic, invariants, or intent: `the second one is copied to m0`.
  **L267 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the second one is copied to m0`。
- **L268 EN**: Declares TableGen def `int_amdgcn_s_sendmsg`.
  **L268 CN**: 声明 TableGen def `int_amdgcn_s_sendmsg`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic <[], [llvm_i32_ty, llvm_i32_ty],`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic <[], [llvm_i32_ty, llvm_i32_ty],`。
- **L270 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback]>;`.
  **L270 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback]>;`。
- **L271 EN**: Declares TableGen def `int_amdgcn_s_sendmsghalt`.
  **L271 CN**: 声明 TableGen def `int_amdgcn_s_sendmsghalt`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic <[], [llvm_i32_ty, llvm_i32_ty],`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic <[], [llvm_i32_ty, llvm_i32_ty],`。
- **L273 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrNoCallback]>;`.
  **L273 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrNoCallback]>;`。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L275 EN**: Blank line separating nearby declarations or logic blocks.
  **L275 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L276 EN**: Comment explains nearby logic, invariants, or intent: `gfx11 intrinsic`.
  **L276 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx11 intrinsic`。
- **L277 EN**: Comment explains nearby logic, invariants, or intent: `The first parameter is s_sendmsg immediate (i16). Return type is i32 or i64.`.
  **L277 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The first parameter is s_sendmsg immediate (i16). Return type is i32 or i64.`。
- **L278 EN**: Declares TableGen def `int_amdgcn_s_sendmsg_rtn`.
  **L278 CN**: 声明 TableGen def `int_amdgcn_s_sendmsg_rtn`。
- **L279 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback]>;`.
  **L279 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback]>;`。
- **L280 EN**: Blank line separating nearby declarations or logic blocks.
  **L280 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L281 EN**: Comment explains nearby logic, invariants, or intent: `Vanilla workgroup sync-barrier`.
  **L281 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vanilla workgroup sync-barrier`。
- **L282 EN**: Declares TableGen def `int_amdgcn_s_barrier`.
  **L282 CN**: 声明 TableGen def `int_amdgcn_s_barrier`。
- **L283 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L283 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L285 EN**: Comment explains nearby logic, invariants, or intent: `Lower-level split-barrier intrinsics`.
  **L285 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Lower-level split-barrier intrinsics`。
- **L286 EN**: Blank line separating nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L287 EN**: Comment explains nearby logic, invariants, or intent: `void @llvm.amdgcn.s.barrier.signal(i32 %barrierType)`.
  **L287 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`void @llvm.amdgcn.s.barrier.signal(i32 %barrierType)`。
- **L288 EN**: Comment explains nearby logic, invariants, or intent: `only for non-named barrier`.
  **L288 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`only for non-named barrier`。

### Lines 289-320

````tablegen
def int_amdgcn_s_barrier_signal : ClangBuiltin<"__builtin_amdgcn_s_barrier_signal">,
  Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,
                                IntrNoCallback, IntrNoFree]>;

// void @llvm.amdgcn.s.barrier.signal.var(ptr addrspace(3) %barrier, i32 %memberCnt)
// The %barrier and %memberCnt argument must be uniform, otherwise behavior is undefined.
// If %memberCnt is 0, the member count is retained from the previous
// s_barrier_init or s_barrier_signal operation.
def int_amdgcn_s_barrier_signal_var : ClangBuiltin<"__builtin_amdgcn_s_barrier_signal_var">,
  Intrinsic<[], [local_ptr_ty, llvm_i32_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,
                                IntrNoCallback, IntrNoFree]>;

// bool @llvm.amdgcn.s.barrier.signal.isfirst(i32 %barrierType)
// only for non-named barrier
def int_amdgcn_s_barrier_signal_isfirst : ClangBuiltin<"__builtin_amdgcn_s_barrier_signal_isfirst">,
  Intrinsic<[llvm_i1_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,
                                IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// void @llvm.amdgcn.s.barrier.init(ptr addrspace(3) %barrier, i32 %memberCnt)
// The %barrier and %memberCnt argument must be uniform, otherwise behavior is undefined.
def int_amdgcn_s_barrier_init : ClangBuiltin<"__builtin_amdgcn_s_barrier_init">,
  Intrinsic<[], [local_ptr_ty, llvm_i32_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent,
                                IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// void @llvm.amdgcn.s.barrier.join(ptr addrspace(3) %barrier)
// The %barrier argument must be uniform, otherwise behavior is undefined.
def int_amdgcn_s_barrier_join : ClangBuiltin<"__builtin_amdgcn_s_barrier_join">,
  Intrinsic<[], [local_ptr_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,
                                IntrNoCallback, IntrNoFree]>;

// void @llvm.amdgcn.s.wakeup.barrier(ptr addrspace(3) %barrier)
// The %barrier argument must be uniform, otherwise behavior is undefined.
````
- **L289 EN**: Declares TableGen def `int_amdgcn_s_barrier_signal`.
  **L289 CN**: 声明 TableGen def `int_amdgcn_s_barrier_signal`。
- **L290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`.
  **L290 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`。
- **L291 EN**: Executes a standalone statement or declaration: `IntrNoCallback, IntrNoFree]>;`.
  **L291 CN**: 执行一条独立语句或声明：`IntrNoCallback, IntrNoFree]>;`。
- **L292 EN**: Blank line separating nearby declarations or logic blocks.
  **L292 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L293 EN**: Comment explains nearby logic, invariants, or intent: `void @llvm.amdgcn.s.barrier.signal.var(ptr addrspace(3) %barrier, i32 %memberCnt)`.
  **L293 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`void @llvm.amdgcn.s.barrier.signal.var(ptr addrspace(3) %barrier, i32 %memberCnt)`。
- **L294 EN**: Comment explains nearby logic, invariants, or intent: `The %barrier and %memberCnt argument must be uniform, otherwise behavior is undefined.`.
  **L294 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The %barrier and %memberCnt argument must be uniform, otherwise behavior is undefined.`。
- **L295 EN**: Comment explains nearby logic, invariants, or intent: `If %memberCnt is 0, the member count is retained from the previous`.
  **L295 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If %memberCnt is 0, the member count is retained from the previous`。
- **L296 EN**: Comment explains nearby logic, invariants, or intent: `s_barrier_init or s_barrier_signal operation.`.
  **L296 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`s_barrier_init or s_barrier_signal operation.`。
- **L297 EN**: Declares TableGen def `int_amdgcn_s_barrier_signal_var`.
  **L297 CN**: 声明 TableGen def `int_amdgcn_s_barrier_signal_var`。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [local_ptr_ty, llvm_i32_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [local_ptr_ty, llvm_i32_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`。
- **L299 EN**: Executes a standalone statement or declaration: `IntrNoCallback, IntrNoFree]>;`.
  **L299 CN**: 执行一条独立语句或声明：`IntrNoCallback, IntrNoFree]>;`。
- **L300 EN**: Blank line separating nearby declarations or logic blocks.
  **L300 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L301 EN**: Comment explains nearby logic, invariants, or intent: `bool @llvm.amdgcn.s.barrier.signal.isfirst(i32 %barrierType)`.
  **L301 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bool @llvm.amdgcn.s.barrier.signal.isfirst(i32 %barrierType)`。
- **L302 EN**: Comment explains nearby logic, invariants, or intent: `only for non-named barrier`.
  **L302 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`only for non-named barrier`。
- **L303 EN**: Declares TableGen def `int_amdgcn_s_barrier_signal_isfirst`.
  **L303 CN**: 声明 TableGen def `int_amdgcn_s_barrier_signal_isfirst`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i1_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i1_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`。
- **L305 EN**: Executes a standalone statement or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L305 CN**: 执行一条独立语句或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L306 EN**: Blank line separating nearby declarations or logic blocks.
  **L306 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L307 EN**: Comment explains nearby logic, invariants, or intent: `void @llvm.amdgcn.s.barrier.init(ptr addrspace(3) %barrier, i32 %memberCnt)`.
  **L307 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`void @llvm.amdgcn.s.barrier.init(ptr addrspace(3) %barrier, i32 %memberCnt)`。
- **L308 EN**: Comment explains nearby logic, invariants, or intent: `The %barrier and %memberCnt argument must be uniform, otherwise behavior is undefined.`.
  **L308 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The %barrier and %memberCnt argument must be uniform, otherwise behavior is undefined.`。
- **L309 EN**: Declares TableGen def `int_amdgcn_s_barrier_init`.
  **L309 CN**: 声明 TableGen def `int_amdgcn_s_barrier_init`。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [local_ptr_ty, llvm_i32_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent,`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [local_ptr_ty, llvm_i32_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent,`。
- **L311 EN**: Executes a standalone statement or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L311 CN**: 执行一条独立语句或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L313 EN**: Comment explains nearby logic, invariants, or intent: `void @llvm.amdgcn.s.barrier.join(ptr addrspace(3) %barrier)`.
  **L313 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`void @llvm.amdgcn.s.barrier.join(ptr addrspace(3) %barrier)`。
- **L314 EN**: Comment explains nearby logic, invariants, or intent: `The %barrier argument must be uniform, otherwise behavior is undefined.`.
  **L314 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The %barrier argument must be uniform, otherwise behavior is undefined.`。
- **L315 EN**: Declares TableGen def `int_amdgcn_s_barrier_join`.
  **L315 CN**: 声明 TableGen def `int_amdgcn_s_barrier_join`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [local_ptr_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`.
  **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [local_ptr_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`。
- **L317 EN**: Executes a standalone statement or declaration: `IntrNoCallback, IntrNoFree]>;`.
  **L317 CN**: 执行一条独立语句或声明：`IntrNoCallback, IntrNoFree]>;`。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L319 EN**: Comment explains nearby logic, invariants, or intent: `void @llvm.amdgcn.s.wakeup.barrier(ptr addrspace(3) %barrier)`.
  **L319 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`void @llvm.amdgcn.s.wakeup.barrier(ptr addrspace(3) %barrier)`。
- **L320 EN**: Comment explains nearby logic, invariants, or intent: `The %barrier argument must be uniform, otherwise behavior is undefined.`.
  **L320 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The %barrier argument must be uniform, otherwise behavior is undefined.`。

### Lines 321-352

````tablegen
def int_amdgcn_s_wakeup_barrier : ClangBuiltin<"__builtin_amdgcn_s_wakeup_barrier">,
  Intrinsic<[], [local_ptr_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,
                                IntrNoCallback, IntrNoFree]>;

// void @llvm.amdgcn.s.barrier.wait(i16 %barrierType)
def int_amdgcn_s_barrier_wait : ClangBuiltin<"__builtin_amdgcn_s_barrier_wait">,
  Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,
                                IntrWillReturn, IntrNoCallback, IntrNoFree]>;


// void @llvm.amdgcn.s.barrier.leave(i16 %barrierType)
def int_amdgcn_s_barrier_leave : ClangBuiltin<"__builtin_amdgcn_s_barrier_leave">,
  Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,
                                IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// uint32_t @llvm.amdgcn.s.get.barrier.state(i32 %barrierId)
// The %barrierType argument must be uniform, otherwise behavior is undefined.
def int_amdgcn_s_get_barrier_state : ClangBuiltin<"__builtin_amdgcn_s_get_barrier_state">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,
                                IntrNoCallback, IntrNoFree]>;

// uint32_t @llvm.amdgcn.s.get.named.barrier.state(ptr addrspace(3) %barrier)
// The %barrier argument must be uniform, otherwise behavior is undefined.
def int_amdgcn_s_get_named_barrier_state : ClangBuiltin<"__builtin_amdgcn_s_get_named_barrier_state">,
  Intrinsic<[llvm_i32_ty], [local_ptr_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,
                                IntrNoCallback, IntrNoFree]>;

def int_amdgcn_wave_barrier : ClangBuiltin<"__builtin_amdgcn_wave_barrier">,
  Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// The 1st parameter is a mask for the types of instructions that may be allowed
// to cross the SCHED_BARRIER during scheduling.
````
- **L321 EN**: Declares TableGen def `int_amdgcn_s_wakeup_barrier`.
  **L321 CN**: 声明 TableGen def `int_amdgcn_s_wakeup_barrier`。
- **L322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [local_ptr_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`.
  **L322 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [local_ptr_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`。
- **L323 EN**: Executes a standalone statement or declaration: `IntrNoCallback, IntrNoFree]>;`.
  **L323 CN**: 执行一条独立语句或声明：`IntrNoCallback, IntrNoFree]>;`。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L325 EN**: Comment explains nearby logic, invariants, or intent: `void @llvm.amdgcn.s.barrier.wait(i16 %barrierType)`.
  **L325 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`void @llvm.amdgcn.s.barrier.wait(i16 %barrierType)`。
- **L326 EN**: Declares TableGen def `int_amdgcn_s_barrier_wait`.
  **L326 CN**: 声明 TableGen def `int_amdgcn_s_barrier_wait`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`。
- **L328 EN**: Executes a standalone statement or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L328 CN**: 执行一条独立语句或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L331 EN**: Comment explains nearby logic, invariants, or intent: `void @llvm.amdgcn.s.barrier.leave(i16 %barrierType)`.
  **L331 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`void @llvm.amdgcn.s.barrier.leave(i16 %barrierType)`。
- **L332 EN**: Declares TableGen def `int_amdgcn_s_barrier_leave`.
  **L332 CN**: 声明 TableGen def `int_amdgcn_s_barrier_leave`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`。
- **L334 EN**: Executes a standalone statement or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L334 CN**: 执行一条独立语句或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L335 EN**: Blank line separating nearby declarations or logic blocks.
  **L335 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L336 EN**: Comment explains nearby logic, invariants, or intent: `uint32_t @llvm.amdgcn.s.get.barrier.state(i32 %barrierId)`.
  **L336 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`uint32_t @llvm.amdgcn.s.get.barrier.state(i32 %barrierId)`。
- **L337 EN**: Comment explains nearby logic, invariants, or intent: `The %barrierType argument must be uniform, otherwise behavior is undefined.`.
  **L337 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The %barrierType argument must be uniform, otherwise behavior is undefined.`。
- **L338 EN**: Declares TableGen def `int_amdgcn_s_get_barrier_state`.
  **L338 CN**: 声明 TableGen def `int_amdgcn_s_get_barrier_state`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`。
- **L340 EN**: Executes a standalone statement or declaration: `IntrNoCallback, IntrNoFree]>;`.
  **L340 CN**: 执行一条独立语句或声明：`IntrNoCallback, IntrNoFree]>;`。
- **L341 EN**: Blank line separating nearby declarations or logic blocks.
  **L341 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L342 EN**: Comment explains nearby logic, invariants, or intent: `uint32_t @llvm.amdgcn.s.get.named.barrier.state(ptr addrspace(3) %barrier)`.
  **L342 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`uint32_t @llvm.amdgcn.s.get.named.barrier.state(ptr addrspace(3) %barrier)`。
- **L343 EN**: Comment explains nearby logic, invariants, or intent: `The %barrier argument must be uniform, otherwise behavior is undefined.`.
  **L343 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The %barrier argument must be uniform, otherwise behavior is undefined.`。
- **L344 EN**: Declares TableGen def `int_amdgcn_s_get_named_barrier_state`.
  **L344 CN**: 声明 TableGen def `int_amdgcn_s_get_named_barrier_state`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [local_ptr_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [local_ptr_ty], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn,`。
- **L346 EN**: Executes a standalone statement or declaration: `IntrNoCallback, IntrNoFree]>;`.
  **L346 CN**: 执行一条独立语句或声明：`IntrNoCallback, IntrNoFree]>;`。
- **L347 EN**: Blank line separating nearby declarations or logic blocks.
  **L347 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L348 EN**: Declares TableGen def `int_amdgcn_wave_barrier`.
  **L348 CN**: 声明 TableGen def `int_amdgcn_wave_barrier`。
- **L349 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L349 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L351 EN**: Comment explains nearby logic, invariants, or intent: `The 1st parameter is a mask for the types of instructions that may be allowed`.
  **L351 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The 1st parameter is a mask for the types of instructions that may be allowed`。
- **L352 EN**: Comment explains nearby logic, invariants, or intent: `to cross the SCHED_BARRIER during scheduling.`.
  **L352 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to cross the SCHED_BARRIER during scheduling.`。

### Lines 353-384

````tablegen
//     MASK = 0x0000 0000: No instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0001: ALL, non-memory, non-side-effect producing instructions may be
//                         scheduled across SCHED_BARRIER, i.e. allow ALU instructions to pass.
//     MASK = 0x0000 0002: VALU instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0004: SALU instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0008: MFMA/WMMA instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0010: ALL VMEM instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0020: VMEM read instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0040: VMEM write instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0080: ALL DS instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0100: ALL DS read instructions may be scheduled accoss SCHED_BARRIER.
//     MASK = 0x0000 0200: ALL DS write instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0400: All Transcendental (e.g. V_EXP) instructions may be scheduled across SCHED_BARRIER.
//     MASK = 0x0000 0800: All LDSDMA instructions may be scheduled across SCHED_BARRIER.
def int_amdgcn_sched_barrier : ClangBuiltin<"__builtin_amdgcn_sched_barrier">,
  Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,
                                IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// The first parameter is a mask that determines the types of instructions that
// you would like to synchronize around and add to a scheduling group. The
// values of the mask are defined above for sched_barrier. These instructions
// will be selected from the bottom up starting from the sched_group_barrier's
// location during instruction scheduling. The second parameter is the number of
// matching instructions that will be associated with this sched_group_barrier.
// The third parameter is an identifier which is used to describe what other
// sched_group_barriers should be synchronized with.
def int_amdgcn_sched_group_barrier : ClangBuiltin<"__builtin_amdgcn_sched_group_barrier">,
  Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
  [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, IntrNoMem, IntrHasSideEffects,
   IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// Scheduler optimization hint.
````
- **L353 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0000: No instructions may be scheduled across SCHED_BARRIER.`.
  **L353 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0000: No instructions may be scheduled across SCHED_BARRIER.`。
- **L354 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0001: ALL, non-memory, non-side-effect producing instructions may be`.
  **L354 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0001: ALL, non-memory, non-side-effect producing instructions may be`。
- **L355 EN**: Comment explains nearby logic, invariants, or intent: `scheduled across SCHED_BARRIER, i.e. allow ALU instructions to pass.`.
  **L355 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scheduled across SCHED_BARRIER, i.e. allow ALU instructions to pass.`。
- **L356 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0002: VALU instructions may be scheduled across SCHED_BARRIER.`.
  **L356 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0002: VALU instructions may be scheduled across SCHED_BARRIER.`。
- **L357 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0004: SALU instructions may be scheduled across SCHED_BARRIER.`.
  **L357 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0004: SALU instructions may be scheduled across SCHED_BARRIER.`。
- **L358 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0008: MFMA/WMMA instructions may be scheduled across SCHED_BARRIER.`.
  **L358 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0008: MFMA/WMMA instructions may be scheduled across SCHED_BARRIER.`。
- **L359 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0010: ALL VMEM instructions may be scheduled across SCHED_BARRIER.`.
  **L359 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0010: ALL VMEM instructions may be scheduled across SCHED_BARRIER.`。
- **L360 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0020: VMEM read instructions may be scheduled across SCHED_BARRIER.`.
  **L360 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0020: VMEM read instructions may be scheduled across SCHED_BARRIER.`。
- **L361 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0040: VMEM write instructions may be scheduled across SCHED_BARRIER.`.
  **L361 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0040: VMEM write instructions may be scheduled across SCHED_BARRIER.`。
- **L362 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0080: ALL DS instructions may be scheduled across SCHED_BARRIER.`.
  **L362 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0080: ALL DS instructions may be scheduled across SCHED_BARRIER.`。
- **L363 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0100: ALL DS read instructions may be scheduled accoss SCHED_BARRIER.`.
  **L363 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0100: ALL DS read instructions may be scheduled accoss SCHED_BARRIER.`。
- **L364 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0200: ALL DS write instructions may be scheduled across SCHED_BARRIER.`.
  **L364 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0200: ALL DS write instructions may be scheduled across SCHED_BARRIER.`。
- **L365 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0400: All Transcendental (e.g. V_EXP) instructions may be scheduled across SCHED_BARRIER.`.
  **L365 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0400: All Transcendental (e.g. V_EXP) instructions may be scheduled across SCHED_BARRIER.`。
- **L366 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0x0000 0800: All LDSDMA instructions may be scheduled across SCHED_BARRIER.`.
  **L366 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0x0000 0800: All LDSDMA instructions may be scheduled across SCHED_BARRIER.`。
- **L367 EN**: Declares TableGen def `int_amdgcn_sched_barrier`.
  **L367 CN**: 声明 TableGen def `int_amdgcn_sched_barrier`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`。
- **L369 EN**: Executes a standalone statement or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L369 CN**: 执行一条独立语句或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L370 EN**: Blank line separating nearby declarations or logic blocks.
  **L370 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L371 EN**: Comment explains nearby logic, invariants, or intent: `The first parameter is a mask that determines the types of instructions that`.
  **L371 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The first parameter is a mask that determines the types of instructions that`。
- **L372 EN**: Comment explains nearby logic, invariants, or intent: `you would like to synchronize around and add to a scheduling group. The`.
  **L372 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`you would like to synchronize around and add to a scheduling group. The`。
- **L373 EN**: Comment explains nearby logic, invariants, or intent: `values of the mask are defined above for sched_barrier. These instructions`.
  **L373 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`values of the mask are defined above for sched_barrier. These instructions`。
- **L374 EN**: Comment explains nearby logic, invariants, or intent: `will be selected from the bottom up starting from the sched_group_barrier's`.
  **L374 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`will be selected from the bottom up starting from the sched_group_barrier's`。
- **L375 EN**: Comment explains nearby logic, invariants, or intent: `location during instruction scheduling. The second parameter is the number of`.
  **L375 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`location during instruction scheduling. The second parameter is the number of`。
- **L376 EN**: Comment explains nearby logic, invariants, or intent: `matching instructions that will be associated with this sched_group_barrier.`.
  **L376 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`matching instructions that will be associated with this sched_group_barrier.`。
- **L377 EN**: Comment explains nearby logic, invariants, or intent: `The third parameter is an identifier which is used to describe what other`.
  **L377 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The third parameter is an identifier which is used to describe what other`。
- **L378 EN**: Comment explains nearby logic, invariants, or intent: `sched_group_barriers should be synchronized with.`.
  **L378 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sched_group_barriers should be synchronized with.`。
- **L379 EN**: Declares TableGen def `int_amdgcn_sched_group_barrier`.
  **L379 CN**: 声明 TableGen def `int_amdgcn_sched_group_barrier`。
- **L380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L380 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, IntrNoMem, IntrHasSideEffects,`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, IntrNoMem, IntrHasSideEffects,`。
- **L382 EN**: Executes a standalone statement or declaration: `IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L382 CN**: 执行一条独立语句或声明：`IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L383 EN**: Blank line separating nearby declarations or logic blocks.
  **L383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L384 EN**: Comment explains nearby logic, invariants, or intent: `Scheduler optimization hint.`.
  **L384 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scheduler optimization hint.`。

### Lines 385-416

````tablegen
//     MASK = 0: Small gemm opt
def int_amdgcn_iglp_opt : ClangBuiltin<"__builtin_amdgcn_iglp_opt">,
  Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,
                                IntrWillReturn, IntrNoCallback, IntrNoFree]>;

def int_amdgcn_s_waitcnt : ClangBuiltin<"__builtin_amdgcn_s_waitcnt">,
  Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// GFX12 intrinsics
class AMDGPUWaitIntrinsic :
  Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;
def int_amdgcn_s_wait_bvhcnt         : AMDGPUWaitIntrinsic;
def int_amdgcn_s_wait_dscnt          : AMDGPUWaitIntrinsic;
def int_amdgcn_s_wait_expcnt         : AMDGPUWaitIntrinsic;
def int_amdgcn_s_wait_kmcnt          : AMDGPUWaitIntrinsic;
def int_amdgcn_s_wait_loadcnt        : AMDGPUWaitIntrinsic;
def int_amdgcn_s_wait_samplecnt      : AMDGPUWaitIntrinsic;
def int_amdgcn_s_wait_storecnt       : AMDGPUWaitIntrinsic;

// Request the hardware to allocate the given number of VGPRs. The actual number
// of allocated VGPRs may be rounded up to match hardware block boundaries.
// It is the responsibility of the calling code to ensure it does not allocate
// below the VGPR requirements of the current shader. This intrinsic is only
// available on targets that support dynamic VGPR mode.
def int_amdgcn_s_alloc_vgpr : DefaultAttrsIntrinsic<
    [llvm_i1_ty], // Returns true if the allocation succeeded, false otherwise.
    [llvm_i32_ty], // The number of VGPRs to allocate.
    [NoUndef<RetIndex>, NoUndef<ArgIndex<0>>,
     IntrNoMem, IntrHasSideEffects, IntrConvergent
    ]
>;

````
- **L385 EN**: Comment explains nearby logic, invariants, or intent: `MASK = 0: Small gemm opt`.
  **L385 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MASK = 0: Small gemm opt`。
- **L386 EN**: Declares TableGen def `int_amdgcn_iglp_opt`.
  **L386 CN**: 声明 TableGen def `int_amdgcn_iglp_opt`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrConvergent,`。
- **L388 EN**: Executes a standalone statement or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L388 CN**: 执行一条独立语句或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L390 EN**: Declares TableGen def `int_amdgcn_s_waitcnt`.
  **L390 CN**: 声明 TableGen def `int_amdgcn_s_waitcnt`。
- **L391 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L391 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L393 EN**: Comment explains nearby logic, invariants, or intent: `GFX12 intrinsics`.
  **L393 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GFX12 intrinsics`。
- **L394 EN**: Declares class `AMDGPUWaitIntrinsic`.
  **L394 CN**: 声明 class `AMDGPUWaitIntrinsic`。
- **L395 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L395 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L396 EN**: Declares TableGen def `int_amdgcn_s_wait_bvhcnt`.
  **L396 CN**: 声明 TableGen def `int_amdgcn_s_wait_bvhcnt`。
- **L397 EN**: Declares TableGen def `int_amdgcn_s_wait_dscnt`.
  **L397 CN**: 声明 TableGen def `int_amdgcn_s_wait_dscnt`。
- **L398 EN**: Declares TableGen def `int_amdgcn_s_wait_expcnt`.
  **L398 CN**: 声明 TableGen def `int_amdgcn_s_wait_expcnt`。
- **L399 EN**: Declares TableGen def `int_amdgcn_s_wait_kmcnt`.
  **L399 CN**: 声明 TableGen def `int_amdgcn_s_wait_kmcnt`。
- **L400 EN**: Declares TableGen def `int_amdgcn_s_wait_loadcnt`.
  **L400 CN**: 声明 TableGen def `int_amdgcn_s_wait_loadcnt`。
- **L401 EN**: Declares TableGen def `int_amdgcn_s_wait_samplecnt`.
  **L401 CN**: 声明 TableGen def `int_amdgcn_s_wait_samplecnt`。
- **L402 EN**: Declares TableGen def `int_amdgcn_s_wait_storecnt`.
  **L402 CN**: 声明 TableGen def `int_amdgcn_s_wait_storecnt`。
- **L403 EN**: Blank line separating nearby declarations or logic blocks.
  **L403 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L404 EN**: Comment explains nearby logic, invariants, or intent: `Request the hardware to allocate the given number of VGPRs. The actual number`.
  **L404 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Request the hardware to allocate the given number of VGPRs. The actual number`。
- **L405 EN**: Comment explains nearby logic, invariants, or intent: `of allocated VGPRs may be rounded up to match hardware block boundaries.`.
  **L405 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`of allocated VGPRs may be rounded up to match hardware block boundaries.`。
- **L406 EN**: Comment explains nearby logic, invariants, or intent: `It is the responsibility of the calling code to ensure it does not allocate`.
  **L406 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It is the responsibility of the calling code to ensure it does not allocate`。
- **L407 EN**: Comment explains nearby logic, invariants, or intent: `below the VGPR requirements of the current shader. This intrinsic is only`.
  **L407 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`below the VGPR requirements of the current shader. This intrinsic is only`。
- **L408 EN**: Comment explains nearby logic, invariants, or intent: `available on targets that support dynamic VGPR mode.`.
  **L408 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`available on targets that support dynamic VGPR mode.`。
- **L409 EN**: Declares TableGen def `int_amdgcn_s_alloc_vgpr`.
  **L409 CN**: 声明 TableGen def `int_amdgcn_s_alloc_vgpr`。
- **L410 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty], // Returns true if the allocation succeeded, false otherwise.`.
  **L410 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty], // Returns true if the allocation succeeded, false otherwise.`。
- **L411 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // The number of VGPRs to allocate.`.
  **L411 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // The number of VGPRs to allocate.`。
- **L412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NoUndef<RetIndex>, NoUndef<ArgIndex<0>>,`.
  **L412 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NoUndef<RetIndex>, NoUndef<ArgIndex<0>>,`。
- **L413 EN**: Continues the surrounding expression or declaration: `IntrNoMem, IntrHasSideEffects, IntrConvergent`.
  **L413 CN**: 继续构造周围的表达式或声明：`IntrNoMem, IntrHasSideEffects, IntrConvergent`。
- **L414 EN**: Continues the surrounding expression or declaration: `]`.
  **L414 CN**: 继续构造周围的表达式或声明：`]`。
- **L415 EN**: Executes a standalone statement or declaration: `>;`.
  **L415 CN**: 执行一条独立语句或声明：`>;`。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 417-448

````tablegen
def int_amdgcn_div_scale : PureIntrinsic<
  // 1st parameter: Numerator
  // 2nd parameter: Denominator
  // 3rd parameter: Select quotient. Must equal Numerator or Denominator.
  //                (0 = Denominator, 1 = Numerator).
  [llvm_anyfloat_ty, llvm_i1_ty],
  [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i1_ty],
  [ImmArg<ArgIndex<2>>]
>;

def int_amdgcn_div_fmas : PureIntrinsic<[llvm_anyfloat_ty],
  [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, llvm_i1_ty]
>;

def int_amdgcn_div_fixup : PureIntrinsic<[llvm_anyfloat_ty],
  [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]>;

// Look Up 2.0 / pi src0 with segment select src1[4:0]
def int_amdgcn_trig_preop : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>, llvm_i32_ty]
>;

def int_amdgcn_sin : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

def int_amdgcn_cos : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

// v_log_{f16|f32}, performs log2. f32 version does not handle
// denormals. There is no reason to use this for f16 as it does
````
- **L417 EN**: Declares TableGen def `int_amdgcn_div_scale`.
  **L417 CN**: 声明 TableGen def `int_amdgcn_div_scale`。
- **L418 EN**: Comment explains nearby logic, invariants, or intent: `1st parameter: Numerator`.
  **L418 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`1st parameter: Numerator`。
- **L419 EN**: Comment explains nearby logic, invariants, or intent: `2nd parameter: Denominator`.
  **L419 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2nd parameter: Denominator`。
- **L420 EN**: Comment explains nearby logic, invariants, or intent: `3rd parameter: Select quotient. Must equal Numerator or Denominator.`.
  **L420 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`3rd parameter: Select quotient. Must equal Numerator or Denominator.`。
- **L421 EN**: Comment explains nearby logic, invariants, or intent: `(0 = Denominator, 1 = Numerator).`.
  **L421 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(0 = Denominator, 1 = Numerator).`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyfloat_ty, llvm_i1_ty],`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyfloat_ty, llvm_i1_ty],`。
- **L423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i1_ty],`.
  **L423 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i1_ty],`。
- **L424 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<2>>]`.
  **L424 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<2>>]`。
- **L425 EN**: Executes a standalone statement or declaration: `>;`.
  **L425 CN**: 执行一条独立语句或声明：`>;`。
- **L426 EN**: Blank line separating nearby declarations or logic blocks.
  **L426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L427 EN**: Declares TableGen def `int_amdgcn_div_fmas`.
  **L427 CN**: 声明 TableGen def `int_amdgcn_div_fmas`。
- **L428 EN**: Continues the surrounding expression or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, llvm_i1_ty]`.
  **L428 CN**: 继续构造周围的表达式或声明：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, llvm_i1_ty]`。
- **L429 EN**: Executes a standalone statement or declaration: `>;`.
  **L429 CN**: 执行一条独立语句或声明：`>;`。
- **L430 EN**: Blank line separating nearby declarations or logic blocks.
  **L430 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L431 EN**: Declares TableGen def `int_amdgcn_div_fixup`.
  **L431 CN**: 声明 TableGen def `int_amdgcn_div_fixup`。
- **L432 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]>;`.
  **L432 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]>;`。
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L434 EN**: Comment explains nearby logic, invariants, or intent: `Look Up 2.0 / pi src0 with segment select src1[4:0]`.
  **L434 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Look Up 2.0 / pi src0 with segment select src1[4:0]`。
- **L435 EN**: Declares TableGen def `int_amdgcn_trig_preop`.
  **L435 CN**: 声明 TableGen def `int_amdgcn_trig_preop`。
- **L436 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>, llvm_i32_ty]`.
  **L436 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>, llvm_i32_ty]`。
- **L437 EN**: Executes a standalone statement or declaration: `>;`.
  **L437 CN**: 执行一条独立语句或声明：`>;`。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L439 EN**: Declares TableGen def `int_amdgcn_sin`.
  **L439 CN**: 声明 TableGen def `int_amdgcn_sin`。
- **L440 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L440 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L441 EN**: Executes a standalone statement or declaration: `>;`.
  **L441 CN**: 执行一条独立语句或声明：`>;`。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L443 EN**: Declares TableGen def `int_amdgcn_cos`.
  **L443 CN**: 声明 TableGen def `int_amdgcn_cos`。
- **L444 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L444 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L445 EN**: Executes a standalone statement or declaration: `>;`.
  **L445 CN**: 执行一条独立语句或声明：`>;`。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L447 EN**: Comment explains nearby logic, invariants, or intent: `v_log_{f16|f32}, performs log2. f32 version does not handle`.
  **L447 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v_log_{f16|f32}, performs log2. f32 version does not handle`。
- **L448 EN**: Comment explains nearby logic, invariants, or intent: `denormals. There is no reason to use this for f16 as it does`.
  **L448 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`denormals. There is no reason to use this for f16 as it does`。

### Lines 449-480

````tablegen
// support denormals, and the generic log2 intrinsic should be
// preferred.
def int_amdgcn_log : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

// v_exp_{f16|f32} (int_amdgcn_exp was taken by export
// already). Performs exp2. f32 version does not handle
// denormals. There is no reason to use this for f16 as it does
// support denormals, and the generic exp2 intrinsic should be
// preferred.
def int_amdgcn_exp2 : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

def int_amdgcn_log_clamp : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

def int_amdgcn_fmul_legacy : ClangBuiltin<"__builtin_amdgcn_fmul_legacy">,
  PureIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty],
  [Commutative]
>;

// Fused single-precision multiply-add with legacy behaviour for the multiply,
// which is that +/- 0.0 * anything (even NaN or infinity) is +0.0. This is
// intended for use on subtargets that have the v_fma_legacy_f32 and/or
// v_fmac_legacy_f32 instructions. (Note that v_fma_legacy_f16 is unrelated and
// has a completely different kind of legacy behaviour.)
def int_amdgcn_fma_legacy :
  PureIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty, llvm_float_ty],
  [Commutative]
````
- **L449 EN**: Comment explains nearby logic, invariants, or intent: `support denormals, and the generic log2 intrinsic should be`.
  **L449 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`support denormals, and the generic log2 intrinsic should be`。
- **L450 EN**: Comment explains nearby logic, invariants, or intent: `preferred.`.
  **L450 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`preferred.`。
- **L451 EN**: Declares TableGen def `int_amdgcn_log`.
  **L451 CN**: 声明 TableGen def `int_amdgcn_log`。
- **L452 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L452 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L453 EN**: Executes a standalone statement or declaration: `>;`.
  **L453 CN**: 执行一条独立语句或声明：`>;`。
- **L454 EN**: Blank line separating nearby declarations or logic blocks.
  **L454 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L455 EN**: Comment explains nearby logic, invariants, or intent: `v_exp_{f16|f32} (int_amdgcn_exp was taken by export`.
  **L455 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v_exp_{f16|f32} (int_amdgcn_exp was taken by export`。
- **L456 EN**: Comment explains nearby logic, invariants, or intent: `already). Performs exp2. f32 version does not handle`.
  **L456 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`already). Performs exp2. f32 version does not handle`。
- **L457 EN**: Comment explains nearby logic, invariants, or intent: `denormals. There is no reason to use this for f16 as it does`.
  **L457 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`denormals. There is no reason to use this for f16 as it does`。
- **L458 EN**: Comment explains nearby logic, invariants, or intent: `support denormals, and the generic exp2 intrinsic should be`.
  **L458 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`support denormals, and the generic exp2 intrinsic should be`。
- **L459 EN**: Comment explains nearby logic, invariants, or intent: `preferred.`.
  **L459 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`preferred.`。
- **L460 EN**: Declares TableGen def `int_amdgcn_exp2`.
  **L460 CN**: 声明 TableGen def `int_amdgcn_exp2`。
- **L461 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L461 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L462 EN**: Executes a standalone statement or declaration: `>;`.
  **L462 CN**: 执行一条独立语句或声明：`>;`。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L464 EN**: Declares TableGen def `int_amdgcn_log_clamp`.
  **L464 CN**: 声明 TableGen def `int_amdgcn_log_clamp`。
- **L465 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L465 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L466 EN**: Executes a standalone statement or declaration: `>;`.
  **L466 CN**: 执行一条独立语句或声明：`>;`。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L468 EN**: Declares TableGen def `int_amdgcn_fmul_legacy`.
  **L468 CN**: 声明 TableGen def `int_amdgcn_fmul_legacy`。
- **L469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty],`.
  **L469 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty],`。
- **L470 EN**: Continues the surrounding expression or declaration: `[Commutative]`.
  **L470 CN**: 继续构造周围的表达式或声明：`[Commutative]`。
- **L471 EN**: Executes a standalone statement or declaration: `>;`.
  **L471 CN**: 执行一条独立语句或声明：`>;`。
- **L472 EN**: Blank line separating nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L473 EN**: Comment explains nearby logic, invariants, or intent: `Fused single-precision multiply-add with legacy behaviour for the multiply,`.
  **L473 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Fused single-precision multiply-add with legacy behaviour for the multiply,`。
- **L474 EN**: Comment explains nearby logic, invariants, or intent: `which is that +/- 0.0 * anything (even NaN or infinity) is +0.0. This is`.
  **L474 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`which is that +/- 0.0 * anything (even NaN or infinity) is +0.0. This is`。
- **L475 EN**: Comment explains nearby logic, invariants, or intent: `intended for use on subtargets that have the v_fma_legacy_f32 and/or`.
  **L475 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`intended for use on subtargets that have the v_fma_legacy_f32 and/or`。
- **L476 EN**: Comment explains nearby logic, invariants, or intent: `v_fmac_legacy_f32 instructions. (Note that v_fma_legacy_f16 is unrelated and`.
  **L476 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v_fmac_legacy_f32 instructions. (Note that v_fma_legacy_f16 is unrelated and`。
- **L477 EN**: Comment explains nearby logic, invariants, or intent: `has a completely different kind of legacy behaviour.)`.
  **L477 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`has a completely different kind of legacy behaviour.)`。
- **L478 EN**: Declares TableGen def `int_amdgcn_fma_legacy`.
  **L478 CN**: 声明 TableGen def `int_amdgcn_fma_legacy`。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty, llvm_float_ty],`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty, llvm_float_ty],`。
- **L480 EN**: Continues the surrounding expression or declaration: `[Commutative]`.
  **L480 CN**: 继续构造周围的表达式或声明：`[Commutative]`。

### Lines 481-512

````tablegen
>;

def int_amdgcn_rcp : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

def int_amdgcn_rcp_legacy : ClangBuiltin<"__builtin_amdgcn_rcp_legacy">,
  PureIntrinsic<[llvm_float_ty], [llvm_float_ty]
>;

def int_amdgcn_sqrt :  PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

def int_amdgcn_rsq :  PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

def int_amdgcn_rsq_legacy :  ClangBuiltin<"__builtin_amdgcn_rsq_legacy">,
  PureIntrinsic<
  [llvm_float_ty], [llvm_float_ty]
>;

// out = 1.0 / sqrt(a) result clamped to +/- max_float.
def int_amdgcn_rsq_clamp : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

def int_amdgcn_frexp_mant : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

````
- **L481 EN**: Executes a standalone statement or declaration: `>;`.
  **L481 CN**: 执行一条独立语句或声明：`>;`。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L483 EN**: Declares TableGen def `int_amdgcn_rcp`.
  **L483 CN**: 声明 TableGen def `int_amdgcn_rcp`。
- **L484 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L484 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L485 EN**: Executes a standalone statement or declaration: `>;`.
  **L485 CN**: 执行一条独立语句或声明：`>;`。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L487 EN**: Declares TableGen def `int_amdgcn_rcp_legacy`.
  **L487 CN**: 声明 TableGen def `int_amdgcn_rcp_legacy`。
- **L488 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<[llvm_float_ty], [llvm_float_ty]`.
  **L488 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<[llvm_float_ty], [llvm_float_ty]`。
- **L489 EN**: Executes a standalone statement or declaration: `>;`.
  **L489 CN**: 执行一条独立语句或声明：`>;`。
- **L490 EN**: Blank line separating nearby declarations or logic blocks.
  **L490 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L491 EN**: Declares TableGen def `int_amdgcn_sqrt`.
  **L491 CN**: 声明 TableGen def `int_amdgcn_sqrt`。
- **L492 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L492 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L493 EN**: Executes a standalone statement or declaration: `>;`.
  **L493 CN**: 执行一条独立语句或声明：`>;`。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L495 EN**: Declares TableGen def `int_amdgcn_rsq`.
  **L495 CN**: 声明 TableGen def `int_amdgcn_rsq`。
- **L496 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L496 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L497 EN**: Executes a standalone statement or declaration: `>;`.
  **L497 CN**: 执行一条独立语句或声明：`>;`。
- **L498 EN**: Blank line separating nearby declarations or logic blocks.
  **L498 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L499 EN**: Declares TableGen def `int_amdgcn_rsq_legacy`.
  **L499 CN**: 声明 TableGen def `int_amdgcn_rsq_legacy`。
- **L500 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L500 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L501 EN**: Continues the surrounding expression or declaration: `[llvm_float_ty], [llvm_float_ty]`.
  **L501 CN**: 继续构造周围的表达式或声明：`[llvm_float_ty], [llvm_float_ty]`。
- **L502 EN**: Executes a standalone statement or declaration: `>;`.
  **L502 CN**: 执行一条独立语句或声明：`>;`。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L504 EN**: Comment explains nearby logic, invariants, or intent: `out = 1.0 / sqrt(a) result clamped to +/- max_float.`.
  **L504 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`out = 1.0 / sqrt(a) result clamped to +/- max_float.`。
- **L505 EN**: Declares TableGen def `int_amdgcn_rsq_clamp`.
  **L505 CN**: 声明 TableGen def `int_amdgcn_rsq_clamp`。
- **L506 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L506 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L507 EN**: Executes a standalone statement or declaration: `>;`.
  **L507 CN**: 执行一条独立语句或声明：`>;`。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L509 EN**: Declares TableGen def `int_amdgcn_frexp_mant`.
  **L509 CN**: 声明 TableGen def `int_amdgcn_frexp_mant`。
- **L510 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L510 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L511 EN**: Executes a standalone statement or declaration: `>;`.
  **L511 CN**: 执行一条独立语句或声明：`>;`。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 513-544

````tablegen
def int_amdgcn_frexp_exp : PureIntrinsic<
  [llvm_anyint_ty], [llvm_anyfloat_ty]
>;

// v_fract is buggy on SI/CI. It mishandles infinities, may return 1.0
// and always uses rtz, so is not suitable for implementing the OpenCL
// fract function. It should be ok on VI.
def int_amdgcn_fract : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

def int_amdgcn_cvt_pkrtz : ClangBuiltin<"__builtin_amdgcn_cvt_pkrtz">,
  PureIntrinsic<[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty]
>;

def int_amdgcn_cvt_pknorm_i16 :
  ClangBuiltin<"__builtin_amdgcn_cvt_pknorm_i16">,
  PureIntrinsic<[llvm_v2i16_ty], [llvm_float_ty, llvm_float_ty]
>;

def int_amdgcn_cvt_pknorm_u16 :
  ClangBuiltin<"__builtin_amdgcn_cvt_pknorm_u16">,
  PureIntrinsic<[llvm_v2i16_ty], [llvm_float_ty, llvm_float_ty]
>;

def int_amdgcn_cvt_pk_i16 :
    ClangBuiltin<"__builtin_amdgcn_cvt_pk_i16">,
    PureIntrinsic<
  [llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty]
>;

def int_amdgcn_cvt_pk_u16 : ClangBuiltin<"__builtin_amdgcn_cvt_pk_u16">,
````
- **L513 EN**: Declares TableGen def `int_amdgcn_frexp_exp`.
  **L513 CN**: 声明 TableGen def `int_amdgcn_frexp_exp`。
- **L514 EN**: Continues the surrounding expression or declaration: `[llvm_anyint_ty], [llvm_anyfloat_ty]`.
  **L514 CN**: 继续构造周围的表达式或声明：`[llvm_anyint_ty], [llvm_anyfloat_ty]`。
- **L515 EN**: Executes a standalone statement or declaration: `>;`.
  **L515 CN**: 执行一条独立语句或声明：`>;`。
- **L516 EN**: Blank line separating nearby declarations or logic blocks.
  **L516 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L517 EN**: Comment explains nearby logic, invariants, or intent: `v_fract is buggy on SI/CI. It mishandles infinities, may return 1.0`.
  **L517 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v_fract is buggy on SI/CI. It mishandles infinities, may return 1.0`。
- **L518 EN**: Comment explains nearby logic, invariants, or intent: `and always uses rtz, so is not suitable for implementing the OpenCL`.
  **L518 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and always uses rtz, so is not suitable for implementing the OpenCL`。
- **L519 EN**: Comment explains nearby logic, invariants, or intent: `fract function. It should be ok on VI.`.
  **L519 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`fract function. It should be ok on VI.`。
- **L520 EN**: Declares TableGen def `int_amdgcn_fract`.
  **L520 CN**: 声明 TableGen def `int_amdgcn_fract`。
- **L521 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L521 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L522 EN**: Executes a standalone statement or declaration: `>;`.
  **L522 CN**: 执行一条独立语句或声明：`>;`。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L524 EN**: Declares TableGen def `int_amdgcn_cvt_pkrtz`.
  **L524 CN**: 声明 TableGen def `int_amdgcn_cvt_pkrtz`。
- **L525 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty]`.
  **L525 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty]`。
- **L526 EN**: Executes a standalone statement or declaration: `>;`.
  **L526 CN**: 执行一条独立语句或声明：`>;`。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L528 EN**: Declares TableGen def `int_amdgcn_cvt_pknorm_i16`.
  **L528 CN**: 声明 TableGen def `int_amdgcn_cvt_pknorm_i16`。
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_cvt_pknorm_i16">,`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_cvt_pknorm_i16">,`。
- **L530 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<[llvm_v2i16_ty], [llvm_float_ty, llvm_float_ty]`.
  **L530 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<[llvm_v2i16_ty], [llvm_float_ty, llvm_float_ty]`。
- **L531 EN**: Executes a standalone statement or declaration: `>;`.
  **L531 CN**: 执行一条独立语句或声明：`>;`。
- **L532 EN**: Blank line separating nearby declarations or logic blocks.
  **L532 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L533 EN**: Declares TableGen def `int_amdgcn_cvt_pknorm_u16`.
  **L533 CN**: 声明 TableGen def `int_amdgcn_cvt_pknorm_u16`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_cvt_pknorm_u16">,`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_cvt_pknorm_u16">,`。
- **L535 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<[llvm_v2i16_ty], [llvm_float_ty, llvm_float_ty]`.
  **L535 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<[llvm_v2i16_ty], [llvm_float_ty, llvm_float_ty]`。
- **L536 EN**: Executes a standalone statement or declaration: `>;`.
  **L536 CN**: 执行一条独立语句或声明：`>;`。
- **L537 EN**: Blank line separating nearby declarations or logic blocks.
  **L537 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L538 EN**: Declares TableGen def `int_amdgcn_cvt_pk_i16`.
  **L538 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_i16`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_cvt_pk_i16">,`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_cvt_pk_i16">,`。
- **L540 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L540 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L541 EN**: Continues the surrounding expression or declaration: `[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty]`.
  **L541 CN**: 继续构造周围的表达式或声明：`[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty]`。
- **L542 EN**: Executes a standalone statement or declaration: `>;`.
  **L542 CN**: 执行一条独立语句或声明：`>;`。
- **L543 EN**: Blank line separating nearby declarations or logic blocks.
  **L543 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L544 EN**: Declares TableGen def `int_amdgcn_cvt_pk_u16`.
  **L544 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_u16`。

### Lines 545-576

````tablegen
  PureIntrinsic<[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty]
>;

def int_amdgcn_class : PureIntrinsic<
  [llvm_i1_ty], [llvm_anyfloat_ty, llvm_i32_ty]
>;

def int_amdgcn_fmed3 :
  PureIntrinsic<[llvm_anyfloat_ty],
    [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]
>;

def int_amdgcn_cubeid : ClangBuiltin<"__builtin_amdgcn_cubeid">,
  PureIntrinsic<[llvm_float_ty],
    [llvm_float_ty, llvm_float_ty, llvm_float_ty]
>;

def int_amdgcn_cubema : ClangBuiltin<"__builtin_amdgcn_cubema">,
  PureIntrinsic<[llvm_float_ty],
  [llvm_float_ty, llvm_float_ty, llvm_float_ty]
>;

def int_amdgcn_cubesc : ClangBuiltin<"__builtin_amdgcn_cubesc">,
  PureIntrinsic<[llvm_float_ty],
    [llvm_float_ty, llvm_float_ty, llvm_float_ty]
>;

def int_amdgcn_cubetc : ClangBuiltin<"__builtin_amdgcn_cubetc">,
  PureIntrinsic<[llvm_float_ty],
    [llvm_float_ty, llvm_float_ty, llvm_float_ty]
>;

````
- **L545 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty]`.
  **L545 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty]`。
- **L546 EN**: Executes a standalone statement or declaration: `>;`.
  **L546 CN**: 执行一条独立语句或声明：`>;`。
- **L547 EN**: Blank line separating nearby declarations or logic blocks.
  **L547 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L548 EN**: Declares TableGen def `int_amdgcn_class`.
  **L548 CN**: 声明 TableGen def `int_amdgcn_class`。
- **L549 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty], [llvm_anyfloat_ty, llvm_i32_ty]`.
  **L549 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty], [llvm_anyfloat_ty, llvm_i32_ty]`。
- **L550 EN**: Executes a standalone statement or declaration: `>;`.
  **L550 CN**: 执行一条独立语句或声明：`>;`。
- **L551 EN**: Blank line separating nearby declarations or logic blocks.
  **L551 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L552 EN**: Declares TableGen def `int_amdgcn_fmed3`.
  **L552 CN**: 声明 TableGen def `int_amdgcn_fmed3`。
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_anyfloat_ty],`.
  **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_anyfloat_ty],`。
- **L554 EN**: Continues the surrounding expression or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]`.
  **L554 CN**: 继续构造周围的表达式或声明：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]`。
- **L555 EN**: Executes a standalone statement or declaration: `>;`.
  **L555 CN**: 执行一条独立语句或声明：`>;`。
- **L556 EN**: Blank line separating nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L557 EN**: Declares TableGen def `int_amdgcn_cubeid`.
  **L557 CN**: 声明 TableGen def `int_amdgcn_cubeid`。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_float_ty],`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_float_ty],`。
- **L559 EN**: Continues the surrounding expression or declaration: `[llvm_float_ty, llvm_float_ty, llvm_float_ty]`.
  **L559 CN**: 继续构造周围的表达式或声明：`[llvm_float_ty, llvm_float_ty, llvm_float_ty]`。
- **L560 EN**: Executes a standalone statement or declaration: `>;`.
  **L560 CN**: 执行一条独立语句或声明：`>;`。
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L562 EN**: Declares TableGen def `int_amdgcn_cubema`.
  **L562 CN**: 声明 TableGen def `int_amdgcn_cubema`。
- **L563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_float_ty],`.
  **L563 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_float_ty],`。
- **L564 EN**: Continues the surrounding expression or declaration: `[llvm_float_ty, llvm_float_ty, llvm_float_ty]`.
  **L564 CN**: 继续构造周围的表达式或声明：`[llvm_float_ty, llvm_float_ty, llvm_float_ty]`。
- **L565 EN**: Executes a standalone statement or declaration: `>;`.
  **L565 CN**: 执行一条独立语句或声明：`>;`。
- **L566 EN**: Blank line separating nearby declarations or logic blocks.
  **L566 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L567 EN**: Declares TableGen def `int_amdgcn_cubesc`.
  **L567 CN**: 声明 TableGen def `int_amdgcn_cubesc`。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_float_ty],`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_float_ty],`。
- **L569 EN**: Continues the surrounding expression or declaration: `[llvm_float_ty, llvm_float_ty, llvm_float_ty]`.
  **L569 CN**: 继续构造周围的表达式或声明：`[llvm_float_ty, llvm_float_ty, llvm_float_ty]`。
- **L570 EN**: Executes a standalone statement or declaration: `>;`.
  **L570 CN**: 执行一条独立语句或声明：`>;`。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L572 EN**: Declares TableGen def `int_amdgcn_cubetc`.
  **L572 CN**: 声明 TableGen def `int_amdgcn_cubetc`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_float_ty],`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_float_ty],`。
- **L574 EN**: Continues the surrounding expression or declaration: `[llvm_float_ty, llvm_float_ty, llvm_float_ty]`.
  **L574 CN**: 继续构造周围的表达式或声明：`[llvm_float_ty, llvm_float_ty, llvm_float_ty]`。
- **L575 EN**: Executes a standalone statement or declaration: `>;`.
  **L575 CN**: 执行一条独立语句或声明：`>;`。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 577-608

````tablegen
// v_ffbh_i32, as opposed to v_ffbh_u32. For v_ffbh_u32, llvm.ctlz
// should be used.
def int_amdgcn_sffbh :
  PureIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>]
>;

// v_mad_f32|f16/v_mac_f32|f16, selected regardless of denorm support.
def int_amdgcn_fmad_ftz :
  PureIntrinsic<[llvm_anyfloat_ty],
            [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]
>;

def int_amdgcn_tanh : PureIntrinsic<
  [llvm_anyfloat_ty], [LLVMMatchType<0>]
>;

def int_amdgcn_cvt_sr_pk_f16_f32 : PureIntrinsic<
  [llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]
>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_pk_f16_f32">;

def int_amdgcn_cvt_sr_pk_bf16_f32 : PureIntrinsic<
  [llvm_v2bf16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]
>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_pk_bf16_f32">;

def int_amdgcn_cvt_pk_f16_fp8 : PureIntrinsic<
  [llvm_v2f16_ty], [llvm_i16_ty]
>, ClangBuiltin<"__builtin_amdgcn_cvt_pk_f16_fp8">;

def int_amdgcn_cvt_pk_f16_bf8 : PureIntrinsic<
  [llvm_v2f16_ty], [llvm_i16_ty]
>, ClangBuiltin<"__builtin_amdgcn_cvt_pk_f16_bf8">;

````
- **L577 EN**: Comment explains nearby logic, invariants, or intent: `v_ffbh_i32, as opposed to v_ffbh_u32. For v_ffbh_u32, llvm.ctlz`.
  **L577 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v_ffbh_i32, as opposed to v_ffbh_u32. For v_ffbh_u32, llvm.ctlz`。
- **L578 EN**: Comment explains nearby logic, invariants, or intent: `should be used.`.
  **L578 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`should be used.`。
- **L579 EN**: Declares TableGen def `int_amdgcn_sffbh`.
  **L579 CN**: 声明 TableGen def `int_amdgcn_sffbh`。
- **L580 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>]`.
  **L580 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>]`。
- **L581 EN**: Executes a standalone statement or declaration: `>;`.
  **L581 CN**: 执行一条独立语句或声明：`>;`。
- **L582 EN**: Blank line separating nearby declarations or logic blocks.
  **L582 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L583 EN**: Comment explains nearby logic, invariants, or intent: `v_mad_f32|f16/v_mac_f32|f16, selected regardless of denorm support.`.
  **L583 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v_mad_f32|f16/v_mac_f32|f16, selected regardless of denorm support.`。
- **L584 EN**: Declares TableGen def `int_amdgcn_fmad_ftz`.
  **L584 CN**: 声明 TableGen def `int_amdgcn_fmad_ftz`。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_anyfloat_ty],`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_anyfloat_ty],`。
- **L586 EN**: Continues the surrounding expression or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]`.
  **L586 CN**: 继续构造周围的表达式或声明：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]`。
- **L587 EN**: Executes a standalone statement or declaration: `>;`.
  **L587 CN**: 执行一条独立语句或声明：`>;`。
- **L588 EN**: Blank line separating nearby declarations or logic blocks.
  **L588 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L589 EN**: Declares TableGen def `int_amdgcn_tanh`.
  **L589 CN**: 声明 TableGen def `int_amdgcn_tanh`。
- **L590 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], [LLVMMatchType<0>]`.
  **L590 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], [LLVMMatchType<0>]`。
- **L591 EN**: Executes a standalone statement or declaration: `>;`.
  **L591 CN**: 执行一条独立语句或声明：`>;`。
- **L592 EN**: Blank line separating nearby declarations or logic blocks.
  **L592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L593 EN**: Declares TableGen def `int_amdgcn_cvt_sr_pk_f16_f32`.
  **L593 CN**: 声明 TableGen def `int_amdgcn_cvt_sr_pk_f16_f32`。
- **L594 EN**: Continues the surrounding expression or declaration: `[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]`.
  **L594 CN**: 继续构造周围的表达式或声明：`[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]`。
- **L595 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_pk_f16_f32">;`.
  **L595 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_pk_f16_f32">;`。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L597 EN**: Declares TableGen def `int_amdgcn_cvt_sr_pk_bf16_f32`.
  **L597 CN**: 声明 TableGen def `int_amdgcn_cvt_sr_pk_bf16_f32`。
- **L598 EN**: Continues the surrounding expression or declaration: `[llvm_v2bf16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]`.
  **L598 CN**: 继续构造周围的表达式或声明：`[llvm_v2bf16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]`。
- **L599 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_pk_bf16_f32">;`.
  **L599 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_pk_bf16_f32">;`。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L601 EN**: Declares TableGen def `int_amdgcn_cvt_pk_f16_fp8`.
  **L601 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_f16_fp8`。
- **L602 EN**: Continues the surrounding expression or declaration: `[llvm_v2f16_ty], [llvm_i16_ty]`.
  **L602 CN**: 继续构造周围的表达式或声明：`[llvm_v2f16_ty], [llvm_i16_ty]`。
- **L603 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_cvt_pk_f16_fp8">;`.
  **L603 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_cvt_pk_f16_fp8">;`。
- **L604 EN**: Blank line separating nearby declarations or logic blocks.
  **L604 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L605 EN**: Declares TableGen def `int_amdgcn_cvt_pk_f16_bf8`.
  **L605 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_f16_bf8`。
- **L606 EN**: Continues the surrounding expression or declaration: `[llvm_v2f16_ty], [llvm_i16_ty]`.
  **L606 CN**: 继续构造周围的表达式或声明：`[llvm_v2f16_ty], [llvm_i16_ty]`。
- **L607 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_cvt_pk_f16_bf8">;`.
  **L607 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_cvt_pk_f16_bf8">;`。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 609-640

````tablegen
def int_amdgcn_cvt_pk_fp8_f16
    : PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>,
      ClangBuiltin<"__builtin_amdgcn_cvt_pk_fp8_f16">;

def int_amdgcn_cvt_pk_bf8_f16
    : PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>,
      ClangBuiltin<"__builtin_amdgcn_cvt_pk_bf8_f16">;

// FIXME: The m0 argument should be moved after the normal arguments
class AMDGPUDSOrderedIntrinsic : Intrinsic<
  [llvm_i32_ty],
  // M0 = {hi16:address, lo16:waveID}. Allow passing M0 as a pointer, so that
  // the bit packing can be optimized at the IR level.
  [LLVMQualPointerType<2>, // IntToPtr(M0)
   llvm_i32_ty, // value to add or swap
   llvm_i32_ty, // ordering
   llvm_i32_ty, // scope
   llvm_i1_ty,  // isVolatile
   llvm_i32_ty, // ordered count index (OA index), also added to the address
                // gfx10: bits 24-27 indicate the number of active threads/dwords
   llvm_i1_ty,  // wave release, usually set to 1
   llvm_i1_ty], // wave done, set to 1 for the last ordered instruction
  [IntrWillReturn, NoCapture<ArgIndex<0>>,
   ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>,
   ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>, IntrNoCallback, IntrNoFree
  ]
>;

class AMDGPUDSAppendConsumedIntrinsic : Intrinsic<
  [llvm_i32_ty],
  [llvm_anyptr_ty, // LDS or GDS ptr
   llvm_i1_ty], // isVolatile
````
- **L609 EN**: Declares TableGen def `int_amdgcn_cvt_pk_fp8_f16`.
  **L609 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_fp8_f16`。
- **L610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>,`.
  **L610 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>,`。
- **L611 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_cvt_pk_fp8_f16">;`.
  **L611 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_cvt_pk_fp8_f16">;`。
- **L612 EN**: Blank line separating nearby declarations or logic blocks.
  **L612 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L613 EN**: Declares TableGen def `int_amdgcn_cvt_pk_bf8_f16`.
  **L613 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_bf8_f16`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>,`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>,`。
- **L615 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_cvt_pk_bf8_f16">;`.
  **L615 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_cvt_pk_bf8_f16">;`。
- **L616 EN**: Blank line separating nearby declarations or logic blocks.
  **L616 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L617 EN**: Comment records a pending task or caution: `FIXME: The m0 argument should be moved after the normal arguments`.
  **L617 CN**: 注释记录了待办事项或注意点：`FIXME: The m0 argument should be moved after the normal arguments`。
- **L618 EN**: Declares class `AMDGPUDSOrderedIntrinsic`.
  **L618 CN**: 声明 class `AMDGPUDSOrderedIntrinsic`。
- **L619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L619 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L620 EN**: Comment explains nearby logic, invariants, or intent: `M0 = {hi16:address, lo16:waveID}. Allow passing M0 as a pointer, so that`.
  **L620 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`M0 = {hi16:address, lo16:waveID}. Allow passing M0 as a pointer, so that`。
- **L621 EN**: Comment explains nearby logic, invariants, or intent: `the bit packing can be optimized at the IR level.`.
  **L621 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the bit packing can be optimized at the IR level.`。
- **L622 EN**: Continues logic associated with callable symbol `IntToPtr`.
  **L622 CN**: 继续与可调用符号 `IntToPtr` 相关的逻辑。
- **L623 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // value to add or swap`.
  **L623 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // value to add or swap`。
- **L624 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // ordering`.
  **L624 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // ordering`。
- **L625 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // scope`.
  **L625 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // scope`。
- **L626 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,  // isVolatile`.
  **L626 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,  // isVolatile`。
- **L627 EN**: Continues logic associated with callable symbol `index`.
  **L627 CN**: 继续与可调用符号 `index` 相关的逻辑。
- **L628 EN**: Comment explains nearby logic, invariants, or intent: `gfx10: bits 24-27 indicate the number of active threads/dwords`.
  **L628 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx10: bits 24-27 indicate the number of active threads/dwords`。
- **L629 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,  // wave release, usually set to 1`.
  **L629 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,  // wave release, usually set to 1`。
- **L630 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty], // wave done, set to 1 for the last ordered instruction`.
  **L630 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty], // wave done, set to 1 for the last ordered instruction`。
- **L631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWillReturn, NoCapture<ArgIndex<0>>,`.
  **L631 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWillReturn, NoCapture<ArgIndex<0>>,`。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>,`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>,`。
- **L633 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>, IntrNoCallback, IntrNoFree`.
  **L633 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>, IntrNoCallback, IntrNoFree`。
- **L634 EN**: Continues the surrounding expression or declaration: `]`.
  **L634 CN**: 继续构造周围的表达式或声明：`]`。
- **L635 EN**: Executes a standalone statement or declaration: `>;`.
  **L635 CN**: 执行一条独立语句或声明：`>;`。
- **L636 EN**: Blank line separating nearby declarations or logic blocks.
  **L636 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L637 EN**: Declares class `AMDGPUDSAppendConsumedIntrinsic`.
  **L637 CN**: 声明 class `AMDGPUDSAppendConsumedIntrinsic`。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L639 EN**: Continues the surrounding expression or declaration: `[llvm_anyptr_ty, // LDS or GDS ptr`.
  **L639 CN**: 继续构造周围的表达式或声明：`[llvm_anyptr_ty, // LDS or GDS ptr`。
- **L640 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty], // isVolatile`.
  **L640 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty], // isVolatile`。

### Lines 641-672

````tablegen
   [IntrConvergent, IntrWillReturn, IntrArgMemOnly,
    Align<ArgIndex<0>, 4>, NoCapture<ArgIndex<0>>,
    ImmArg<ArgIndex<1>>, IntrNoCallback, IntrNoFree],
   "",
   [SDNPMemOperand]
>;

def int_amdgcn_ds_ordered_add : AMDGPUDSOrderedIntrinsic;
def int_amdgcn_ds_ordered_swap : AMDGPUDSOrderedIntrinsic;

// The pointer argument is assumed to be dynamically uniform if a VGPR.
def int_amdgcn_ds_append : AMDGPUDSAppendConsumedIntrinsic;
def int_amdgcn_ds_consume : AMDGPUDSAppendConsumedIntrinsic;

// llvm.amdgcn.cvt.sr.fp8.f16 i32 vdst, half src, i32 seed, i32 old, imm byte_sel [0..3]
// byte_sel selects byte to write in vdst.
def int_amdgcn_cvt_sr_fp8_f16 : PureIntrinsic<
  [llvm_i32_ty], [llvm_half_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
  [ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>]
>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_fp8_f16">;

// llvm.amdgcn.cvt.sr.bf8.f16 i32 vdst, half src, i32 seed, i32 old, imm byte_sel [0..3]
// byte_sel selects byte to write in vdst.
def int_amdgcn_cvt_sr_bf8_f16 : PureIntrinsic<
  [llvm_i32_ty], [llvm_half_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
  [ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>]
>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_bf8_f16">;

// Note: these gfx1250 intrinsics are convergent because they read scales from other lanes.
// llvm.amdgcn.cvt.scale.pk32.f16.bf6 v32f16 vdst, v6i32 src0, i32 scale_sel [0..15]
class AMDGPUCvtScaleIntrinsic<LLVMType DstTy, LLVMType Src0Ty, string name> : DefaultAttrsIntrinsic<
  [DstTy], [Src0Ty, llvm_i32_ty, llvm_i32_ty],
````
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrWillReturn, IntrArgMemOnly,`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrWillReturn, IntrArgMemOnly,`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Align<ArgIndex<0>, 4>, NoCapture<ArgIndex<0>>,`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`Align<ArgIndex<0>, 4>, NoCapture<ArgIndex<0>>,`。
- **L643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<1>>, IntrNoCallback, IntrNoFree],`.
  **L643 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<1>>, IntrNoCallback, IntrNoFree],`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"",`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`"",`。
- **L645 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand]`.
  **L645 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand]`。
- **L646 EN**: Executes a standalone statement or declaration: `>;`.
  **L646 CN**: 执行一条独立语句或声明：`>;`。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L648 EN**: Declares TableGen def `int_amdgcn_ds_ordered_add`.
  **L648 CN**: 声明 TableGen def `int_amdgcn_ds_ordered_add`。
- **L649 EN**: Declares TableGen def `int_amdgcn_ds_ordered_swap`.
  **L649 CN**: 声明 TableGen def `int_amdgcn_ds_ordered_swap`。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L651 EN**: Comment explains nearby logic, invariants, or intent: `The pointer argument is assumed to be dynamically uniform if a VGPR.`.
  **L651 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The pointer argument is assumed to be dynamically uniform if a VGPR.`。
- **L652 EN**: Declares TableGen def `int_amdgcn_ds_append`.
  **L652 CN**: 声明 TableGen def `int_amdgcn_ds_append`。
- **L653 EN**: Declares TableGen def `int_amdgcn_ds_consume`.
  **L653 CN**: 声明 TableGen def `int_amdgcn_ds_consume`。
- **L654 EN**: Blank line separating nearby declarations or logic blocks.
  **L654 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L655 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.sr.fp8.f16 i32 vdst, half src, i32 seed, i32 old, imm byte_sel [0..3]`.
  **L655 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.sr.fp8.f16 i32 vdst, half src, i32 seed, i32 old, imm byte_sel [0..3]`。
- **L656 EN**: Comment explains nearby logic, invariants, or intent: `byte_sel selects byte to write in vdst.`.
  **L656 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`byte_sel selects byte to write in vdst.`。
- **L657 EN**: Declares TableGen def `int_amdgcn_cvt_sr_fp8_f16`.
  **L657 CN**: 声明 TableGen def `int_amdgcn_cvt_sr_fp8_f16`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty], [llvm_half_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty], [llvm_half_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L659 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>]`.
  **L659 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>]`。
- **L660 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_fp8_f16">;`.
  **L660 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_fp8_f16">;`。
- **L661 EN**: Blank line separating nearby declarations or logic blocks.
  **L661 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L662 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.sr.bf8.f16 i32 vdst, half src, i32 seed, i32 old, imm byte_sel [0..3]`.
  **L662 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.sr.bf8.f16 i32 vdst, half src, i32 seed, i32 old, imm byte_sel [0..3]`。
- **L663 EN**: Comment explains nearby logic, invariants, or intent: `byte_sel selects byte to write in vdst.`.
  **L663 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`byte_sel selects byte to write in vdst.`。
- **L664 EN**: Declares TableGen def `int_amdgcn_cvt_sr_bf8_f16`.
  **L664 CN**: 声明 TableGen def `int_amdgcn_cvt_sr_bf8_f16`。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty], [llvm_half_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty], [llvm_half_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L666 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>]`.
  **L666 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>]`。
- **L667 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_bf8_f16">;`.
  **L667 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_cvt_sr_bf8_f16">;`。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L669 EN**: Comment explains nearby logic, invariants, or intent: `Note: these gfx1250 intrinsics are convergent because they read scales from other lanes.`.
  **L669 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: these gfx1250 intrinsics are convergent because they read scales from other lanes.`。
- **L670 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scale.pk32.f16.bf6 v32f16 vdst, v6i32 src0, i32 scale_sel [0..15]`.
  **L670 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scale.pk32.f16.bf6 v32f16 vdst, v6i32 src0, i32 scale_sel [0..15]`。
- **L671 EN**: Declares class `AMDGPUCvtScaleIntrinsic<LLVMType`.
  **L671 CN**: 声明 class `AMDGPUCvtScaleIntrinsic<LLVMType`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[DstTy], [Src0Ty, llvm_i32_ty, llvm_i32_ty],`.
  **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`[DstTy], [Src0Ty, llvm_i32_ty, llvm_i32_ty],`。

### Lines 673-704

````tablegen
  [IntrNoMem, IntrConvergent, IntrWillReturn,IntrNoCreateUndefOrPoison,
   ImmArg<ArgIndex<2>>, Range<ArgIndex<2>, 0, 16>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScaleF32Intrinsic<LLVMType DstTy, LLVMType Src0Ty, string name> : PureIntrinsic<
  [DstTy], [Src0Ty, llvm_float_ty]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScaleF32SRIntrinsic<LLVMType DstTy, LLVMType Src0Ty, string name> : PureIntrinsic<
  [DstTy], [Src0Ty, llvm_i32_ty, llvm_float_ty]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

def int_amdgcn_cvt_scale_pk8_f16_fp8   : AMDGPUCvtScaleIntrinsic<llvm_v8f16_ty,   llvm_v2i32_ty, "cvt_scale_pk8_f16_fp8">;
def int_amdgcn_cvt_scale_pk8_bf16_fp8  : AMDGPUCvtScaleIntrinsic<llvm_v8bf16_ty,  llvm_v2i32_ty, "cvt_scale_pk8_bf16_fp8">;
def int_amdgcn_cvt_scale_pk8_f16_bf8   : AMDGPUCvtScaleIntrinsic<llvm_v8f16_ty,   llvm_v2i32_ty, "cvt_scale_pk8_f16_bf8">;
def int_amdgcn_cvt_scale_pk8_bf16_bf8  : AMDGPUCvtScaleIntrinsic<llvm_v8bf16_ty,  llvm_v2i32_ty, "cvt_scale_pk8_bf16_bf8">;
def int_amdgcn_cvt_scale_pk8_f16_fp4   : AMDGPUCvtScaleIntrinsic<llvm_v8f16_ty,   llvm_i32_ty,   "cvt_scale_pk8_f16_fp4">;
def int_amdgcn_cvt_scale_pk8_bf16_fp4  : AMDGPUCvtScaleIntrinsic<llvm_v8bf16_ty,  llvm_i32_ty,   "cvt_scale_pk8_bf16_fp4">;
def int_amdgcn_cvt_scale_pk8_f32_fp8   : AMDGPUCvtScaleIntrinsic<llvm_v8f32_ty,   llvm_v2i32_ty, "cvt_scale_pk8_f32_fp8">;
def int_amdgcn_cvt_scale_pk8_f32_bf8   : AMDGPUCvtScaleIntrinsic<llvm_v8f32_ty,   llvm_v2i32_ty, "cvt_scale_pk8_f32_bf8">;
def int_amdgcn_cvt_scale_pk8_f32_fp4   : AMDGPUCvtScaleIntrinsic<llvm_v8f32_ty,   llvm_i32_ty,   "cvt_scale_pk8_f32_fp4">;
def int_amdgcn_cvt_scale_pk16_f16_bf6  : AMDGPUCvtScaleIntrinsic<llvm_v16f16_ty,  llvm_v3i32_ty, "cvt_scale_pk16_f16_bf6">;
def int_amdgcn_cvt_scale_pk16_bf16_bf6 : AMDGPUCvtScaleIntrinsic<llvm_v16bf16_ty, llvm_v3i32_ty, "cvt_scale_pk16_bf16_bf6">;
def int_amdgcn_cvt_scale_pk16_f16_fp6  : AMDGPUCvtScaleIntrinsic<llvm_v16f16_ty,  llvm_v3i32_ty, "cvt_scale_pk16_f16_fp6">;
def int_amdgcn_cvt_scale_pk16_bf16_fp6 : AMDGPUCvtScaleIntrinsic<llvm_v16bf16_ty, llvm_v3i32_ty, "cvt_scale_pk16_bf16_fp6">;
def int_amdgcn_cvt_scale_pk16_f32_fp6  : AMDGPUCvtScaleIntrinsic<llvm_v16f32_ty,  llvm_v3i32_ty, "cvt_scale_pk16_f32_fp6">;
def int_amdgcn_cvt_scale_pk16_f32_bf6  : AMDGPUCvtScaleIntrinsic<llvm_v16f32_ty,  llvm_v3i32_ty, "cvt_scale_pk16_f32_bf6">;

class AMDGPUCvtScaleF32ToFP6BF6Intrinsic<LLVMType DstTy, LLVMType Src0Ty, LLVMType Src1Ty, string name> : PureIntrinsic<
  [DstTy], [Src0Ty, Src1Ty, llvm_float_ty]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

````
- **L673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,IntrNoCreateUndefOrPoison,`.
  **L673 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,IntrNoCreateUndefOrPoison,`。
- **L674 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<2>>, Range<ArgIndex<2>, 0, 16>]`.
  **L674 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<2>>, Range<ArgIndex<2>, 0, 16>]`。
- **L675 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L675 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L676 EN**: Blank line separating nearby declarations or logic blocks.
  **L676 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L677 EN**: Declares class `AMDGPUCvtScaleF32Intrinsic<LLVMType`.
  **L677 CN**: 声明 class `AMDGPUCvtScaleF32Intrinsic<LLVMType`。
- **L678 EN**: Continues the surrounding expression or declaration: `[DstTy], [Src0Ty, llvm_float_ty]`.
  **L678 CN**: 继续构造周围的表达式或声明：`[DstTy], [Src0Ty, llvm_float_ty]`。
- **L679 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L679 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L681 EN**: Declares class `AMDGPUCvtScaleF32SRIntrinsic<LLVMType`.
  **L681 CN**: 声明 class `AMDGPUCvtScaleF32SRIntrinsic<LLVMType`。
- **L682 EN**: Continues the surrounding expression or declaration: `[DstTy], [Src0Ty, llvm_i32_ty, llvm_float_ty]`.
  **L682 CN**: 继续构造周围的表达式或声明：`[DstTy], [Src0Ty, llvm_i32_ty, llvm_float_ty]`。
- **L683 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L683 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L684 EN**: Blank line separating nearby declarations or logic blocks.
  **L684 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L685 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk8_f16_fp8`.
  **L685 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk8_f16_fp8`。
- **L686 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk8_bf16_fp8`.
  **L686 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk8_bf16_fp8`。
- **L687 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk8_f16_bf8`.
  **L687 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk8_f16_bf8`。
- **L688 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk8_bf16_bf8`.
  **L688 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk8_bf16_bf8`。
- **L689 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk8_f16_fp4`.
  **L689 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk8_f16_fp4`。
- **L690 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk8_bf16_fp4`.
  **L690 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk8_bf16_fp4`。
- **L691 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk8_f32_fp8`.
  **L691 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk8_f32_fp8`。
- **L692 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk8_f32_bf8`.
  **L692 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk8_f32_bf8`。
- **L693 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk8_f32_fp4`.
  **L693 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk8_f32_fp4`。
- **L694 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk16_f16_bf6`.
  **L694 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk16_f16_bf6`。
- **L695 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk16_bf16_bf6`.
  **L695 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk16_bf16_bf6`。
- **L696 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk16_f16_fp6`.
  **L696 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk16_f16_fp6`。
- **L697 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk16_bf16_fp6`.
  **L697 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk16_bf16_fp6`。
- **L698 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk16_f32_fp6`.
  **L698 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk16_f32_fp6`。
- **L699 EN**: Declares TableGen def `int_amdgcn_cvt_scale_pk16_f32_bf6`.
  **L699 CN**: 声明 TableGen def `int_amdgcn_cvt_scale_pk16_f32_bf6`。
- **L700 EN**: Blank line separating nearby declarations or logic blocks.
  **L700 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L701 EN**: Declares class `AMDGPUCvtScaleF32ToFP6BF6Intrinsic<LLVMType`.
  **L701 CN**: 声明 class `AMDGPUCvtScaleF32ToFP6BF6Intrinsic<LLVMType`。
- **L702 EN**: Continues the surrounding expression or declaration: `[DstTy], [Src0Ty, Src1Ty, llvm_float_ty]`.
  **L702 CN**: 继续构造周围的表达式或声明：`[DstTy], [Src0Ty, Src1Ty, llvm_float_ty]`。
- **L703 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L703 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L704 EN**: Blank line separating nearby declarations or logic blocks.
  **L704 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 705-736

````tablegen
def int_amdgcn_cvt_scalef32_pk32_fp6_f16  : AMDGPUCvtScaleF32Intrinsic<llvm_v6i32_ty, llvm_v32f16_ty,  "cvt_scalef32_pk32_fp6_f16">;
def int_amdgcn_cvt_scalef32_pk32_bf6_f16  : AMDGPUCvtScaleF32Intrinsic<llvm_v6i32_ty, llvm_v32f16_ty,  "cvt_scalef32_pk32_bf6_f16">;
def int_amdgcn_cvt_scalef32_pk8_fp8_bf16  : AMDGPUCvtScaleF32Intrinsic<llvm_v2i32_ty, llvm_v8bf16_ty,  "cvt_scalef32_pk8_fp8_bf16">;
def int_amdgcn_cvt_scalef32_pk8_bf8_bf16  : AMDGPUCvtScaleF32Intrinsic<llvm_v2i32_ty, llvm_v8bf16_ty,  "cvt_scalef32_pk8_bf8_bf16">;
def int_amdgcn_cvt_scalef32_pk32_fp6_bf16 : AMDGPUCvtScaleF32Intrinsic<llvm_v6i32_ty, llvm_v32bf16_ty, "cvt_scalef32_pk32_fp6_bf16">;
def int_amdgcn_cvt_scalef32_pk32_bf6_bf16 : AMDGPUCvtScaleF32Intrinsic<llvm_v6i32_ty, llvm_v32bf16_ty, "cvt_scalef32_pk32_bf6_bf16">;
def int_amdgcn_cvt_scalef32_pk8_fp8_f16   : AMDGPUCvtScaleF32Intrinsic<llvm_v2i32_ty, llvm_v8f16_ty,   "cvt_scalef32_pk8_fp8_f16">;
def int_amdgcn_cvt_scalef32_pk8_bf8_f16   : AMDGPUCvtScaleF32Intrinsic<llvm_v2i32_ty, llvm_v8f16_ty,   "cvt_scalef32_pk8_bf8_f16">;
def int_amdgcn_cvt_scalef32_pk8_fp8_f32   : AMDGPUCvtScaleF32Intrinsic<llvm_v2i32_ty, llvm_v8f32_ty,   "cvt_scalef32_pk8_fp8_f32">;
def int_amdgcn_cvt_scalef32_pk8_bf8_f32   : AMDGPUCvtScaleF32Intrinsic<llvm_v2i32_ty, llvm_v8f32_ty,   "cvt_scalef32_pk8_bf8_f32">;
def int_amdgcn_cvt_scalef32_pk8_fp4_f32   : AMDGPUCvtScaleF32Intrinsic<llvm_i32_ty,   llvm_v8f32_ty,   "cvt_scalef32_pk8_fp4_f32">;
def int_amdgcn_cvt_scalef32_pk8_fp4_f16   : AMDGPUCvtScaleF32Intrinsic<llvm_i32_ty,   llvm_v8f16_ty,   "cvt_scalef32_pk8_fp4_f16">;
def int_amdgcn_cvt_scalef32_pk8_fp4_bf16  : AMDGPUCvtScaleF32Intrinsic<llvm_i32_ty,   llvm_v8bf16_ty,  "cvt_scalef32_pk8_fp4_bf16">;
def int_amdgcn_cvt_scalef32_pk16_fp6_f32  : AMDGPUCvtScaleF32Intrinsic<llvm_v3i32_ty, llvm_v16f32_ty,   "cvt_scalef32_pk16_fp6_f32">;
def int_amdgcn_cvt_scalef32_pk16_bf6_f32  : AMDGPUCvtScaleF32Intrinsic<llvm_v3i32_ty, llvm_v16f32_ty,   "cvt_scalef32_pk16_bf6_f32">;
def int_amdgcn_cvt_scalef32_pk16_fp6_f16  : AMDGPUCvtScaleF32Intrinsic<llvm_v3i32_ty, llvm_v16f16_ty,   "cvt_scalef32_pk16_fp6_f16">;
def int_amdgcn_cvt_scalef32_pk16_bf6_f16  : AMDGPUCvtScaleF32Intrinsic<llvm_v3i32_ty, llvm_v16f16_ty,   "cvt_scalef32_pk16_bf6_f16">;
def int_amdgcn_cvt_scalef32_pk16_fp6_bf16 : AMDGPUCvtScaleF32Intrinsic<llvm_v3i32_ty, llvm_v16bf16_ty,  "cvt_scalef32_pk16_fp6_bf16">;
def int_amdgcn_cvt_scalef32_pk16_bf6_bf16 : AMDGPUCvtScaleF32Intrinsic<llvm_v3i32_ty, llvm_v16bf16_ty,  "cvt_scalef32_pk16_bf6_bf16">;

def int_amdgcn_cvt_scalef32_sr_pk32_fp6_f32  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v6i32_ty, llvm_v32f32_ty,  "cvt_scalef32_sr_pk32_fp6_f32">;
def int_amdgcn_cvt_scalef32_sr_pk32_bf6_f32  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v6i32_ty, llvm_v32f32_ty,  "cvt_scalef32_sr_pk32_bf6_f32">;
def int_amdgcn_cvt_scalef32_sr_pk32_fp6_f16  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v6i32_ty, llvm_v32f16_ty,  "cvt_scalef32_sr_pk32_fp6_f16">;
def int_amdgcn_cvt_scalef32_sr_pk32_bf6_f16  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v6i32_ty, llvm_v32f16_ty,  "cvt_scalef32_sr_pk32_bf6_f16">;
def int_amdgcn_cvt_scalef32_sr_pk32_fp6_bf16 : AMDGPUCvtScaleF32SRIntrinsic<llvm_v6i32_ty, llvm_v32bf16_ty, "cvt_scalef32_sr_pk32_fp6_bf16">;
def int_amdgcn_cvt_scalef32_sr_pk32_bf6_bf16 : AMDGPUCvtScaleF32SRIntrinsic<llvm_v6i32_ty, llvm_v32bf16_ty, "cvt_scalef32_sr_pk32_bf6_bf16">;
def int_amdgcn_cvt_scalef32_sr_pk8_fp8_bf16  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v2i32_ty, llvm_v8bf16_ty,  "cvt_scalef32_sr_pk8_fp8_bf16">;
def int_amdgcn_cvt_scalef32_sr_pk8_bf8_bf16  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v2i32_ty, llvm_v8bf16_ty,  "cvt_scalef32_sr_pk8_bf8_bf16">;
def int_amdgcn_cvt_scalef32_sr_pk8_fp8_f16   : AMDGPUCvtScaleF32SRIntrinsic<llvm_v2i32_ty, llvm_v8f16_ty,   "cvt_scalef32_sr_pk8_fp8_f16">;
def int_amdgcn_cvt_scalef32_sr_pk8_bf8_f16   : AMDGPUCvtScaleF32SRIntrinsic<llvm_v2i32_ty, llvm_v8f16_ty,   "cvt_scalef32_sr_pk8_bf8_f16">;
def int_amdgcn_cvt_scalef32_sr_pk8_fp8_f32   : AMDGPUCvtScaleF32SRIntrinsic<llvm_v2i32_ty, llvm_v8f32_ty,   "cvt_scalef32_sr_pk8_fp8_f32">;
def int_amdgcn_cvt_scalef32_sr_pk8_bf8_f32   : AMDGPUCvtScaleF32SRIntrinsic<llvm_v2i32_ty, llvm_v8f32_ty,   "cvt_scalef32_sr_pk8_bf8_f32">;
````
- **L705 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_fp6_f16`.
  **L705 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_fp6_f16`。
- **L706 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_bf6_f16`.
  **L706 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_bf6_f16`。
- **L707 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk8_fp8_bf16`.
  **L707 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk8_fp8_bf16`。
- **L708 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk8_bf8_bf16`.
  **L708 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk8_bf8_bf16`。
- **L709 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_fp6_bf16`.
  **L709 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_fp6_bf16`。
- **L710 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_bf6_bf16`.
  **L710 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_bf6_bf16`。
- **L711 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk8_fp8_f16`.
  **L711 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk8_fp8_f16`。
- **L712 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk8_bf8_f16`.
  **L712 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk8_bf8_f16`。
- **L713 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk8_fp8_f32`.
  **L713 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk8_fp8_f32`。
- **L714 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk8_bf8_f32`.
  **L714 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk8_bf8_f32`。
- **L715 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk8_fp4_f32`.
  **L715 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk8_fp4_f32`。
- **L716 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk8_fp4_f16`.
  **L716 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk8_fp4_f16`。
- **L717 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk8_fp4_bf16`.
  **L717 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk8_fp4_bf16`。
- **L718 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk16_fp6_f32`.
  **L718 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk16_fp6_f32`。
- **L719 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk16_bf6_f32`.
  **L719 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk16_bf6_f32`。
- **L720 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk16_fp6_f16`.
  **L720 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk16_fp6_f16`。
- **L721 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk16_bf6_f16`.
  **L721 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk16_bf6_f16`。
- **L722 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk16_fp6_bf16`.
  **L722 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk16_fp6_bf16`。
- **L723 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk16_bf6_bf16`.
  **L723 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk16_bf6_bf16`。
- **L724 EN**: Blank line separating nearby declarations or logic blocks.
  **L724 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L725 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_fp6_f32`.
  **L725 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_fp6_f32`。
- **L726 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_bf6_f32`.
  **L726 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_bf6_f32`。
- **L727 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_fp6_f16`.
  **L727 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_fp6_f16`。
- **L728 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_bf6_f16`.
  **L728 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_bf6_f16`。
- **L729 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_fp6_bf16`.
  **L729 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_fp6_bf16`。
- **L730 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_bf6_bf16`.
  **L730 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk32_bf6_bf16`。
- **L731 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp8_bf16`.
  **L731 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp8_bf16`。
- **L732 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_bf8_bf16`.
  **L732 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_bf8_bf16`。
- **L733 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp8_f16`.
  **L733 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp8_f16`。
- **L734 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_bf8_f16`.
  **L734 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_bf8_f16`。
- **L735 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp8_f32`.
  **L735 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp8_f32`。
- **L736 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_bf8_f32`.
  **L736 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_bf8_f32`。

### Lines 737-768

````tablegen
def int_amdgcn_cvt_scalef32_sr_pk8_fp4_f32   : AMDGPUCvtScaleF32SRIntrinsic<llvm_i32_ty,   llvm_v8f32_ty,   "cvt_scalef32_sr_pk8_fp4_f32">;
def int_amdgcn_cvt_scalef32_sr_pk8_fp4_f16   : AMDGPUCvtScaleF32SRIntrinsic<llvm_i32_ty,   llvm_v8f16_ty,   "cvt_scalef32_sr_pk8_fp4_f16">;
def int_amdgcn_cvt_scalef32_sr_pk8_fp4_bf16  : AMDGPUCvtScaleF32SRIntrinsic<llvm_i32_ty,   llvm_v8bf16_ty,  "cvt_scalef32_sr_pk8_fp4_bf16">;
def int_amdgcn_cvt_scalef32_sr_pk16_fp6_f32  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v3i32_ty, llvm_v16f32_ty,  "cvt_scalef32_sr_pk16_fp6_f32">;
def int_amdgcn_cvt_scalef32_sr_pk16_bf6_f32  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v3i32_ty, llvm_v16f32_ty,  "cvt_scalef32_sr_pk16_bf6_f32">;
def int_amdgcn_cvt_scalef32_sr_pk16_fp6_f16  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v3i32_ty, llvm_v16f16_ty,  "cvt_scalef32_sr_pk16_fp6_f16">;
def int_amdgcn_cvt_scalef32_sr_pk16_bf6_f16  : AMDGPUCvtScaleF32SRIntrinsic<llvm_v3i32_ty, llvm_v16f16_ty,  "cvt_scalef32_sr_pk16_bf6_f16">;
def int_amdgcn_cvt_scalef32_sr_pk16_fp6_bf16 : AMDGPUCvtScaleF32SRIntrinsic<llvm_v3i32_ty, llvm_v16bf16_ty, "cvt_scalef32_sr_pk16_fp6_bf16">;
def int_amdgcn_cvt_scalef32_sr_pk16_bf6_bf16 : AMDGPUCvtScaleF32SRIntrinsic<llvm_v3i32_ty, llvm_v16bf16_ty, "cvt_scalef32_sr_pk16_bf6_bf16">;

def int_amdgcn_cvt_scalef32_2xpk16_fp6_f32 : AMDGPUCvtScaleF32ToFP6BF6Intrinsic<llvm_v6i32_ty, llvm_v16f32_ty, llvm_v16f32_ty, "cvt_scalef32_2xpk16_fp6_f32">;
def int_amdgcn_cvt_scalef32_2xpk16_bf6_f32 : AMDGPUCvtScaleF32ToFP6BF6Intrinsic<llvm_v6i32_ty, llvm_v16f32_ty, llvm_v16f32_ty, "cvt_scalef32_2xpk16_bf6_f32">;

class AMDGPUCvtScaleFP4FP8BF8ToF1632Intrinsic<LLVMType DstTy, string name> : PureIntrinsic<
  [DstTy],
  [llvm_i32_ty,   // src
   llvm_float_ty, // scale
   llvm_i32_ty],  // src_sel index [0..3]
  [ImmArg<ArgIndex<2>>, Range<ArgIndex<2>, 0, 4>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScale_pk_FromFP8BF8Intrinsic<LLVMType DstTy, string name> : PureIntrinsic<
  [DstTy],
  [llvm_i32_ty,   // src
   llvm_float_ty, // scale
   llvm_i1_ty],   // src_lo_hi_sel[true false]
  [ImmArg<ArgIndex<2>>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScaleF16BF16ToFP8BF8TiedInputIntrinsic<LLVMType SrcTy, string name> : PureIntrinsic<
  [llvm_v2i16_ty],
  [llvm_v2i16_ty, // old_vdst
````
- **L737 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp4_f32`.
  **L737 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp4_f32`。
- **L738 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp4_f16`.
  **L738 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp4_f16`。
- **L739 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp4_bf16`.
  **L739 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk8_fp4_bf16`。
- **L740 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_fp6_f32`.
  **L740 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_fp6_f32`。
- **L741 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_bf6_f32`.
  **L741 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_bf6_f32`。
- **L742 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_fp6_f16`.
  **L742 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_fp6_f16`。
- **L743 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_bf6_f16`.
  **L743 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_bf6_f16`。
- **L744 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_fp6_bf16`.
  **L744 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_fp6_bf16`。
- **L745 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_bf6_bf16`.
  **L745 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk16_bf6_bf16`。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L747 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_2xpk16_fp6_f32`.
  **L747 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_2xpk16_fp6_f32`。
- **L748 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_2xpk16_bf6_f32`.
  **L748 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_2xpk16_bf6_f32`。
- **L749 EN**: Blank line separating nearby declarations or logic blocks.
  **L749 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L750 EN**: Declares class `AMDGPUCvtScaleFP4FP8BF8ToF1632Intrinsic<LLVMType`.
  **L750 CN**: 声明 class `AMDGPUCvtScaleFP4FP8BF8ToF1632Intrinsic<LLVMType`。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[DstTy],`.
  **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`[DstTy],`。
- **L752 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty,   // src`.
  **L752 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty,   // src`。
- **L753 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // scale`.
  **L753 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // scale`。
- **L754 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],  // src_sel index [0..3]`.
  **L754 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],  // src_sel index [0..3]`。
- **L755 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<2>>, Range<ArgIndex<2>, 0, 4>]`.
  **L755 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<2>>, Range<ArgIndex<2>, 0, 4>]`。
- **L756 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L756 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L757 EN**: Blank line separating nearby declarations or logic blocks.
  **L757 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L758 EN**: Declares class `AMDGPUCvtScale_pk_FromFP8BF8Intrinsic<LLVMType`.
  **L758 CN**: 声明 class `AMDGPUCvtScale_pk_FromFP8BF8Intrinsic<LLVMType`。
- **L759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[DstTy],`.
  **L759 CN**: 继续一个多行参数列表、初始化器或聚合项：`[DstTy],`。
- **L760 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty,   // src`.
  **L760 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty,   // src`。
- **L761 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // scale`.
  **L761 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // scale`。
- **L762 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty],   // src_lo_hi_sel[true false]`.
  **L762 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty],   // src_lo_hi_sel[true false]`。
- **L763 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<2>>]`.
  **L763 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<2>>]`。
- **L764 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L764 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L766 EN**: Declares class `AMDGPUCvtScaleF16BF16ToFP8BF8TiedInputIntrinsic<LLVMType`.
  **L766 CN**: 声明 class `AMDGPUCvtScaleF16BF16ToFP8BF8TiedInputIntrinsic<LLVMType`。
- **L767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i16_ty],`.
  **L767 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i16_ty],`。
- **L768 EN**: Continues the surrounding expression or declaration: `[llvm_v2i16_ty, // old_vdst`.
  **L768 CN**: 继续构造周围的表达式或声明：`[llvm_v2i16_ty, // old_vdst`。

### Lines 769-800

````tablegen
   SrcTy,         // src
   llvm_float_ty, // scale
   llvm_i1_ty],   // dst_lo_hi_sel[true false]
  [ImmArg<ArgIndex<3>>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScaleF32ToFP8BF8TiedInputIntrinsic<string name> : PureIntrinsic<
  [llvm_v2i16_ty],
  [llvm_v2i16_ty, // old_vdst
   llvm_float_ty, // src0
   llvm_float_ty, // src1
   llvm_float_ty, // scale
   llvm_i1_ty],   // dst_lo_hi_sel[true false]
  [ImmArg<ArgIndex<4>>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScaleFP8BF8ToF16TiedInputIntrinsic<LLVMType DstTy, string name> :PureIntrinsic<
  [DstTy],
  [llvm_v2f16_ty, // old_vdst
   llvm_i32_ty,   // src
   llvm_float_ty, // scale
   llvm_i32_ty,   // src_sel_index[0..3]
   llvm_i1_ty],   // dst_lo_hi_sel[true false]
  [ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>, ImmArg<ArgIndex<4>>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScaleF32ToFP4Intrinsic<string name> : PureIntrinsic<
  [llvm_i32_ty],
  [llvm_i32_ty,   // old_vdst
   llvm_float_ty, // src0
   llvm_float_ty, // src1
   llvm_float_ty, // scale
````
- **L769 EN**: Continues the surrounding expression or declaration: `SrcTy,         // src`.
  **L769 CN**: 继续构造周围的表达式或声明：`SrcTy,         // src`。
- **L770 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // scale`.
  **L770 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // scale`。
- **L771 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty],   // dst_lo_hi_sel[true false]`.
  **L771 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty],   // dst_lo_hi_sel[true false]`。
- **L772 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L772 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L773 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L773 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L775 EN**: Declares class `AMDGPUCvtScaleF32ToFP8BF8TiedInputIntrinsic<string`.
  **L775 CN**: 声明 class `AMDGPUCvtScaleF32ToFP8BF8TiedInputIntrinsic<string`。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i16_ty],`.
  **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i16_ty],`。
- **L777 EN**: Continues the surrounding expression or declaration: `[llvm_v2i16_ty, // old_vdst`.
  **L777 CN**: 继续构造周围的表达式或声明：`[llvm_v2i16_ty, // old_vdst`。
- **L778 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // src0`.
  **L778 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // src0`。
- **L779 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // src1`.
  **L779 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // src1`。
- **L780 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // scale`.
  **L780 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // scale`。
- **L781 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty],   // dst_lo_hi_sel[true false]`.
  **L781 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty],   // dst_lo_hi_sel[true false]`。
- **L782 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>]`.
  **L782 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>]`。
- **L783 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L783 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L784 EN**: Blank line separating nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L785 EN**: Declares class `AMDGPUCvtScaleFP8BF8ToF16TiedInputIntrinsic<LLVMType`.
  **L785 CN**: 声明 class `AMDGPUCvtScaleFP8BF8ToF16TiedInputIntrinsic<LLVMType`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[DstTy],`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`[DstTy],`。
- **L787 EN**: Continues the surrounding expression or declaration: `[llvm_v2f16_ty, // old_vdst`.
  **L787 CN**: 继续构造周围的表达式或声明：`[llvm_v2f16_ty, // old_vdst`。
- **L788 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // src`.
  **L788 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // src`。
- **L789 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // scale`.
  **L789 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // scale`。
- **L790 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // src_sel_index[0..3]`.
  **L790 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // src_sel_index[0..3]`。
- **L791 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty],   // dst_lo_hi_sel[true false]`.
  **L791 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty],   // dst_lo_hi_sel[true false]`。
- **L792 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>, ImmArg<ArgIndex<4>>]`.
  **L792 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>, ImmArg<ArgIndex<4>>]`。
- **L793 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L793 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L795 EN**: Declares class `AMDGPUCvtScaleF32ToFP4Intrinsic<string`.
  **L795 CN**: 声明 class `AMDGPUCvtScaleF32ToFP4Intrinsic<string`。
- **L796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L796 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L797 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty,   // old_vdst`.
  **L797 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty,   // old_vdst`。
- **L798 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // src0`.
  **L798 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // src0`。
- **L799 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // src1`.
  **L799 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // src1`。
- **L800 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // scale`.
  **L800 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // scale`。

### Lines 801-832

````tablegen
   llvm_i32_ty],  // dst_sel_index[0..3]
  [ImmArg<ArgIndex<4>>, Range<ArgIndex<4>, 0, 4>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScaleF16ToFP4TiedInputIntrinsic<LLVMType SrcTy, string name> : PureIntrinsic<
  [llvm_i32_ty],
  [llvm_i32_ty,   // old_vdst
   SrcTy,         // src
   llvm_float_ty, // scale
   llvm_i32_ty],  // dest_sel_index [0..3]
  [ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<LLVMType Src0Ty, string name> : PureIntrinsic<
  [llvm_i32_ty],
  [llvm_i32_ty,   // old_vdst
   Src0Ty,        // src0
   llvm_i32_ty,   // seed
   llvm_float_ty, // scale
   llvm_i32_ty],  // dst_sel_index[0..3]
  [ImmArg<ArgIndex<4>>, Range<ArgIndex<4>, 0, 4>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

class AMDGPUCvtScaleSRF32ToBF16F16TiedInputIntrinsic<LLVMType DstTy, string name> : PureIntrinsic<
  [DstTy],
  [DstTy,         // old_vdst
   llvm_float_ty, // src0
   llvm_i32_ty,   // seed
   llvm_i1_ty],   // dst_lo_hi_sel[true false]
  [ImmArg<ArgIndex<3>>]
>, ClangBuiltin<"__builtin_amdgcn_"#name>;

````
- **L801 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],  // dst_sel_index[0..3]`.
  **L801 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],  // dst_sel_index[0..3]`。
- **L802 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, Range<ArgIndex<4>, 0, 4>]`.
  **L802 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, Range<ArgIndex<4>, 0, 4>]`。
- **L803 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L803 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L804 EN**: Blank line separating nearby declarations or logic blocks.
  **L804 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L805 EN**: Declares class `AMDGPUCvtScaleF16ToFP4TiedInputIntrinsic<LLVMType`.
  **L805 CN**: 声明 class `AMDGPUCvtScaleF16ToFP4TiedInputIntrinsic<LLVMType`。
- **L806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L806 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L807 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty,   // old_vdst`.
  **L807 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty,   // old_vdst`。
- **L808 EN**: Continues the surrounding expression or declaration: `SrcTy,         // src`.
  **L808 CN**: 继续构造周围的表达式或声明：`SrcTy,         // src`。
- **L809 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // scale`.
  **L809 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // scale`。
- **L810 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],  // dest_sel_index [0..3]`.
  **L810 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],  // dest_sel_index [0..3]`。
- **L811 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>]`.
  **L811 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, Range<ArgIndex<3>, 0, 4>]`。
- **L812 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L812 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L813 EN**: Blank line separating nearby declarations or logic blocks.
  **L813 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L814 EN**: Declares class `AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<LLVMType`.
  **L814 CN**: 声明 class `AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<LLVMType`。
- **L815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L815 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L816 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty,   // old_vdst`.
  **L816 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty,   // old_vdst`。
- **L817 EN**: Continues the surrounding expression or declaration: `Src0Ty,        // src0`.
  **L817 CN**: 继续构造周围的表达式或声明：`Src0Ty,        // src0`。
- **L818 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // seed`.
  **L818 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // seed`。
- **L819 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // scale`.
  **L819 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // scale`。
- **L820 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],  // dst_sel_index[0..3]`.
  **L820 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],  // dst_sel_index[0..3]`。
- **L821 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<4>>, Range<ArgIndex<4>, 0, 4>]`.
  **L821 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<4>>, Range<ArgIndex<4>, 0, 4>]`。
- **L822 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L822 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L823 EN**: Blank line separating nearby declarations or logic blocks.
  **L823 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L824 EN**: Declares class `AMDGPUCvtScaleSRF32ToBF16F16TiedInputIntrinsic<LLVMType`.
  **L824 CN**: 声明 class `AMDGPUCvtScaleSRF32ToBF16F16TiedInputIntrinsic<LLVMType`。
- **L825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[DstTy],`.
  **L825 CN**: 继续一个多行参数列表、初始化器或聚合项：`[DstTy],`。
- **L826 EN**: Continues the surrounding expression or declaration: `[DstTy,         // old_vdst`.
  **L826 CN**: 继续构造周围的表达式或声明：`[DstTy,         // old_vdst`。
- **L827 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // src0`.
  **L827 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // src0`。
- **L828 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // seed`.
  **L828 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // seed`。
- **L829 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty],   // dst_lo_hi_sel[true false]`.
  **L829 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty],   // dst_lo_hi_sel[true false]`。
- **L830 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L830 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L831 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_"#name>;`.
  **L831 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_"#name>;`。
- **L832 EN**: Blank line separating nearby declarations or logic blocks.
  **L832 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 833-864

````tablegen
def int_amdgcn_cvt_sr_bf16_f32: AMDGPUCvtScaleSRF32ToBF16F16TiedInputIntrinsic<llvm_v2bf16_ty, "cvt_sr_bf16_f32">;
def int_amdgcn_cvt_sr_f16_f32 : AMDGPUCvtScaleSRF32ToBF16F16TiedInputIntrinsic<llvm_v2f16_ty, "cvt_sr_f16_f32">;

// llvm.amdgcn.cvt.scalef32.fp16.fp8 v2f16 old_vdst, int src, float scale, int src_sel_index [0..3], bool dst_lo_hi_sel
def int_amdgcn_cvt_scalef32_f16_fp8  : AMDGPUCvtScaleFP8BF8ToF16TiedInputIntrinsic<llvm_v2f16_ty, "cvt_scalef32_f16_fp8">;
def int_amdgcn_cvt_scalef32_f16_bf8  : AMDGPUCvtScaleFP8BF8ToF16TiedInputIntrinsic<llvm_v2f16_ty, "cvt_scalef32_f16_bf8">;

// llvm.amdgcn.cvt.scalef32.f32.fp8 int src, float scale, int src_sel_index [0..3]
def int_amdgcn_cvt_scalef32_f32_fp8  : AMDGPUCvtScaleFP4FP8BF8ToF1632Intrinsic<llvm_float_ty, "cvt_scalef32_f32_fp8">;
def int_amdgcn_cvt_scalef32_f32_bf8  : AMDGPUCvtScaleFP4FP8BF8ToF1632Intrinsic<llvm_float_ty, "cvt_scalef32_f32_bf8">;

// llvm.amdgcn.cvt.scalef32.pk.fp8.f32 v2i16 old_vdst, float srcA, float srcB, float scale, bool dst_lo_hi_sel
def int_amdgcn_cvt_scalef32_pk_fp8_f32 : AMDGPUCvtScaleF32ToFP8BF8TiedInputIntrinsic<"cvt_scalef32_pk_fp8_f32">;
def int_amdgcn_cvt_scalef32_pk_bf8_f32 : AMDGPUCvtScaleF32ToFP8BF8TiedInputIntrinsic<"cvt_scalef32_pk_bf8_f32">;

// llvm.amdgcn.cvt.scalef32.pk.fp32.fp8 int src, float scale, bool src_lo_hi_sel
def int_amdgcn_cvt_scalef32_pk_f32_fp8 : AMDGPUCvtScale_pk_FromFP8BF8Intrinsic<llvm_v2f32_ty, "cvt_scalef32_pk_f32_fp8">;
def int_amdgcn_cvt_scalef32_pk_f32_bf8 : AMDGPUCvtScale_pk_FromFP8BF8Intrinsic<llvm_v2f32_ty, "cvt_scalef32_pk_f32_bf8">;

// llvm.amdgcn.cvt.scalef32.fp8.fp16 v2i16 old_vdst, v2f16 src, float scale, bool dst_lo_hi_sel
def int_amdgcn_cvt_scalef32_pk_fp8_f16 : AMDGPUCvtScaleF16BF16ToFP8BF8TiedInputIntrinsic<llvm_v2f16_ty, "cvt_scalef32_pk_fp8_f16">;
def int_amdgcn_cvt_scalef32_pk_fp8_bf16: AMDGPUCvtScaleF16BF16ToFP8BF8TiedInputIntrinsic<llvm_v2bf16_ty, "cvt_scalef32_pk_fp8_bf16">;
def int_amdgcn_cvt_scalef32_pk_bf8_f16 : AMDGPUCvtScaleF16BF16ToFP8BF8TiedInputIntrinsic<llvm_v2f16_ty, "cvt_scalef32_pk_bf8_f16">;
def int_amdgcn_cvt_scalef32_pk_bf8_bf16: AMDGPUCvtScaleF16BF16ToFP8BF8TiedInputIntrinsic<llvm_v2bf16_ty, "cvt_scalef32_pk_bf8_bf16">;

// llvm.amdgcn.cvt.scalef32.pk.f32.fp4 int src, float scale, int src_sel_index [0..3]
def int_amdgcn_cvt_scalef32_pk_f32_fp4 : AMDGPUCvtScaleFP4FP8BF8ToF1632Intrinsic<llvm_v2f32_ty, "cvt_scalef32_pk_f32_fp4">;

// llvm.amdgcn.cvt.scalef32.pk.fp4.f32 i32 old_vdst, float srcA, float srcB, float scale, int dst_sel_index[0..3]
def int_amdgcn_cvt_scalef32_pk_fp4_f32 : AMDGPUCvtScaleF32ToFP4Intrinsic<"cvt_scalef32_pk_fp4_f32">;

// llvm.amdgcn.cvt.scalef32.pk.f32.fp4 int src, float scale, int src_sel_index [0..3]
````
- **L833 EN**: Declares TableGen def `int_amdgcn_cvt_sr_bf16_f32`.
  **L833 CN**: 声明 TableGen def `int_amdgcn_cvt_sr_bf16_f32`。
- **L834 EN**: Declares TableGen def `int_amdgcn_cvt_sr_f16_f32`.
  **L834 CN**: 声明 TableGen def `int_amdgcn_cvt_sr_f16_f32`。
- **L835 EN**: Blank line separating nearby declarations or logic blocks.
  **L835 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L836 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.fp16.fp8 v2f16 old_vdst, int src, float scale, int src_sel_index [0..3], bool dst_lo_hi_sel`.
  **L836 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.fp16.fp8 v2f16 old_vdst, int src, float scale, int src_sel_index [0..3], bool dst_lo_hi_sel`。
- **L837 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_f16_fp8`.
  **L837 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_f16_fp8`。
- **L838 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_f16_bf8`.
  **L838 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_f16_bf8`。
- **L839 EN**: Blank line separating nearby declarations or logic blocks.
  **L839 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L840 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.f32.fp8 int src, float scale, int src_sel_index [0..3]`.
  **L840 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.f32.fp8 int src, float scale, int src_sel_index [0..3]`。
- **L841 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_f32_fp8`.
  **L841 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_f32_fp8`。
- **L842 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_f32_bf8`.
  **L842 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_f32_bf8`。
- **L843 EN**: Blank line separating nearby declarations or logic blocks.
  **L843 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L844 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.pk.fp8.f32 v2i16 old_vdst, float srcA, float srcB, float scale, bool dst_lo_hi_sel`.
  **L844 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.pk.fp8.f32 v2i16 old_vdst, float srcA, float srcB, float scale, bool dst_lo_hi_sel`。
- **L845 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_fp8_f32`.
  **L845 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_fp8_f32`。
- **L846 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_bf8_f32`.
  **L846 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_bf8_f32`。
- **L847 EN**: Blank line separating nearby declarations or logic blocks.
  **L847 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L848 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.pk.fp32.fp8 int src, float scale, bool src_lo_hi_sel`.
  **L848 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.pk.fp32.fp8 int src, float scale, bool src_lo_hi_sel`。
- **L849 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_f32_fp8`.
  **L849 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_f32_fp8`。
- **L850 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_f32_bf8`.
  **L850 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_f32_bf8`。
- **L851 EN**: Blank line separating nearby declarations or logic blocks.
  **L851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L852 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.fp8.fp16 v2i16 old_vdst, v2f16 src, float scale, bool dst_lo_hi_sel`.
  **L852 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.fp8.fp16 v2i16 old_vdst, v2f16 src, float scale, bool dst_lo_hi_sel`。
- **L853 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_fp8_f16`.
  **L853 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_fp8_f16`。
- **L854 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_fp8_bf16`.
  **L854 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_fp8_bf16`。
- **L855 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_bf8_f16`.
  **L855 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_bf8_f16`。
- **L856 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_bf8_bf16`.
  **L856 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_bf8_bf16`。
- **L857 EN**: Blank line separating nearby declarations or logic blocks.
  **L857 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L858 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.pk.f32.fp4 int src, float scale, int src_sel_index [0..3]`.
  **L858 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.pk.f32.fp4 int src, float scale, int src_sel_index [0..3]`。
- **L859 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_f32_fp4`.
  **L859 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_f32_fp4`。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L861 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.pk.fp4.f32 i32 old_vdst, float srcA, float srcB, float scale, int dst_sel_index[0..3]`.
  **L861 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.pk.fp4.f32 i32 old_vdst, float srcA, float srcB, float scale, int dst_sel_index[0..3]`。
- **L862 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_fp4_f32`.
  **L862 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_fp4_f32`。
- **L863 EN**: Blank line separating nearby declarations or logic blocks.
  **L863 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L864 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.pk.f32.fp4 int src, float scale, int src_sel_index [0..3]`.
  **L864 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.pk.f32.fp4 int src, float scale, int src_sel_index [0..3]`。

### Lines 865-896

````tablegen
def int_amdgcn_cvt_scalef32_pk_f16_fp4 : AMDGPUCvtScaleFP4FP8BF8ToF1632Intrinsic<llvm_v2f16_ty, "cvt_scalef32_pk_f16_fp4">;
def int_amdgcn_cvt_scalef32_pk_bf16_fp4: AMDGPUCvtScaleFP4FP8BF8ToF1632Intrinsic<llvm_v2bf16_ty, "cvt_scalef32_pk_bf16_fp4">;

// llvm.amdgcn.cvt.scalef32.pk32.f32.fp6 v6i32 src, float scale
def int_amdgcn_cvt_scalef32_pk32_f32_fp6  : AMDGPUCvtScaleF32Intrinsic<llvm_v32f32_ty, llvm_v6i32_ty, "cvt_scalef32_pk32_f32_fp6">;
def int_amdgcn_cvt_scalef32_pk32_f32_bf6  : AMDGPUCvtScaleF32Intrinsic<llvm_v32f32_ty, llvm_v6i32_ty, "cvt_scalef32_pk32_f32_bf6">;

// llvm.amdgcn.cvt.scalef32.pk32.f16.fp6 v6i32 src, float scale
def int_amdgcn_cvt_scalef32_pk32_f16_bf6  : AMDGPUCvtScaleF32Intrinsic<llvm_v32f16_ty,  llvm_v6i32_ty, "cvt_scalef32_pk32_f16_bf6">;
def int_amdgcn_cvt_scalef32_pk32_bf16_bf6 : AMDGPUCvtScaleF32Intrinsic<llvm_v32bf16_ty, llvm_v6i32_ty, "cvt_scalef32_pk32_bf16_bf6">;
def int_amdgcn_cvt_scalef32_pk32_f16_fp6  : AMDGPUCvtScaleF32Intrinsic<llvm_v32f16_ty,  llvm_v6i32_ty, "cvt_scalef32_pk32_f16_fp6">;
def int_amdgcn_cvt_scalef32_pk32_bf16_fp6 : AMDGPUCvtScaleF32Intrinsic<llvm_v32bf16_ty, llvm_v6i32_ty, "cvt_scalef32_pk32_bf16_fp6">;

// llvm.amdgcn.cvt.scalef32.pk.fp16.fp8 int src, float scale, bool src_lo_hi_sel
def int_amdgcn_cvt_scalef32_pk_f16_bf8    : AMDGPUCvtScale_pk_FromFP8BF8Intrinsic<llvm_v2f16_ty, "cvt_scalef32_pk_f16_bf8">;
def int_amdgcn_cvt_scalef32_pk_bf16_bf8   : AMDGPUCvtScale_pk_FromFP8BF8Intrinsic<llvm_v2bf16_ty, "cvt_scalef32_pk_bf16_bf8">;
def int_amdgcn_cvt_scalef32_pk_f16_fp8    : AMDGPUCvtScale_pk_FromFP8BF8Intrinsic<llvm_v2f16_ty, "cvt_scalef32_pk_f16_fp8">;
def int_amdgcn_cvt_scalef32_pk_bf16_fp8   : AMDGPUCvtScale_pk_FromFP8BF8Intrinsic<llvm_v2bf16_ty, "cvt_scalef32_pk_bf16_fp8">;

// llvm.amdgcn.cvt.scalef32.pk.fp4.f16 int src, float scale, int dst_sel_index [0..3]
def int_amdgcn_cvt_scalef32_pk_fp4_f16 : AMDGPUCvtScaleF16ToFP4TiedInputIntrinsic<llvm_v2f16_ty, "cvt_scalef32_pk_fp4_f16">;
def int_amdgcn_cvt_scalef32_pk_fp4_bf16: AMDGPUCvtScaleF16ToFP4TiedInputIntrinsic<llvm_v2bf16_ty, "cvt_scalef32_pk_fp4_bf16">;

def int_amdgcn_cvt_scalef32_sr_pk_fp4_f16: AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<llvm_v2f16_ty, "cvt_scalef32_sr_pk_fp4_f16">;
def int_amdgcn_cvt_scalef32_sr_pk_fp4_bf16: AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<llvm_v2bf16_ty, "cvt_scalef32_sr_pk_fp4_bf16">;
def int_amdgcn_cvt_scalef32_sr_pk_fp4_f32: AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<llvm_v2f32_ty, "cvt_scalef32_sr_pk_fp4_f32">;
def int_amdgcn_cvt_scalef32_sr_bf8_bf16: AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<llvm_bfloat_ty, "cvt_scalef32_sr_bf8_bf16">;
def int_amdgcn_cvt_scalef32_sr_bf8_f16: AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<llvm_half_ty, "cvt_scalef32_sr_bf8_f16">;
def int_amdgcn_cvt_scalef32_sr_bf8_f32: AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<llvm_float_ty, "cvt_scalef32_sr_bf8_f32">;
def int_amdgcn_cvt_scalef32_sr_fp8_bf16: AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<llvm_bfloat_ty, "cvt_scalef32_sr_fp8_bf16">;
def int_amdgcn_cvt_scalef32_sr_fp8_f16: AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<llvm_half_ty, "cvt_scalef32_sr_fp8_f16">;
def int_amdgcn_cvt_scalef32_sr_fp8_f32: AMDGPUCvtScaleBF16F16F32SRToFP4BF8F8TiedInputIntrinsic<llvm_float_ty, "cvt_scalef32_sr_fp8_f32">;
````
- **L865 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_f16_fp4`.
  **L865 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_f16_fp4`。
- **L866 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_bf16_fp4`.
  **L866 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_bf16_fp4`。
- **L867 EN**: Blank line separating nearby declarations or logic blocks.
  **L867 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L868 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.pk32.f32.fp6 v6i32 src, float scale`.
  **L868 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.pk32.f32.fp6 v6i32 src, float scale`。
- **L869 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_f32_fp6`.
  **L869 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_f32_fp6`。
- **L870 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_f32_bf6`.
  **L870 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_f32_bf6`。
- **L871 EN**: Blank line separating nearby declarations or logic blocks.
  **L871 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L872 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.pk32.f16.fp6 v6i32 src, float scale`.
  **L872 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.pk32.f16.fp6 v6i32 src, float scale`。
- **L873 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_f16_bf6`.
  **L873 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_f16_bf6`。
- **L874 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_bf16_bf6`.
  **L874 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_bf16_bf6`。
- **L875 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_f16_fp6`.
  **L875 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_f16_fp6`。
- **L876 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk32_bf16_fp6`.
  **L876 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk32_bf16_fp6`。
- **L877 EN**: Blank line separating nearby declarations or logic blocks.
  **L877 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L878 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.pk.fp16.fp8 int src, float scale, bool src_lo_hi_sel`.
  **L878 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.pk.fp16.fp8 int src, float scale, bool src_lo_hi_sel`。
- **L879 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_f16_bf8`.
  **L879 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_f16_bf8`。
- **L880 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_bf16_bf8`.
  **L880 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_bf16_bf8`。
- **L881 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_f16_fp8`.
  **L881 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_f16_fp8`。
- **L882 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_bf16_fp8`.
  **L882 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_bf16_fp8`。
- **L883 EN**: Blank line separating nearby declarations or logic blocks.
  **L883 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L884 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.scalef32.pk.fp4.f16 int src, float scale, int dst_sel_index [0..3]`.
  **L884 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.scalef32.pk.fp4.f16 int src, float scale, int dst_sel_index [0..3]`。
- **L885 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_fp4_f16`.
  **L885 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_fp4_f16`。
- **L886 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_pk_fp4_bf16`.
  **L886 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_pk_fp4_bf16`。
- **L887 EN**: Blank line separating nearby declarations or logic blocks.
  **L887 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L888 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk_fp4_f16`.
  **L888 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk_fp4_f16`。
- **L889 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk_fp4_bf16`.
  **L889 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk_fp4_bf16`。
- **L890 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_pk_fp4_f32`.
  **L890 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_pk_fp4_f32`。
- **L891 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_bf8_bf16`.
  **L891 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_bf8_bf16`。
- **L892 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_bf8_f16`.
  **L892 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_bf8_f16`。
- **L893 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_bf8_f32`.
  **L893 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_bf8_f32`。
- **L894 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_fp8_bf16`.
  **L894 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_fp8_bf16`。
- **L895 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_fp8_f16`.
  **L895 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_fp8_f16`。
- **L896 EN**: Declares TableGen def `int_amdgcn_cvt_scalef32_sr_fp8_f32`.
  **L896 CN**: 声明 TableGen def `int_amdgcn_cvt_scalef32_sr_fp8_f32`。

### Lines 897-928

````tablegen

def int_amdgcn_prng_b32 : DefaultAttrsIntrinsic<
  [llvm_i32_ty], [llvm_i32_ty], [IntrNoMem, IntrNoCreateUndefOrPoison]
>, ClangBuiltin<"__builtin_amdgcn_prng_b32">;

def int_amdgcn_bitop3 :
  PureIntrinsic<[llvm_anyint_ty],
                        [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty],
                        [ImmArg<ArgIndex<3>>]>;

} // TargetPrefix = "amdgcn"

// New-style image intrinsics

//////////////////////////////////////////////////////////////////////////
// Dimension-aware image intrinsics framework
//////////////////////////////////////////////////////////////////////////

// Helper class to represent (type, name) combinations of arguments. The
// argument names are explanatory and used as DAG operand names for codegen
// pattern matching.
class AMDGPUArg<LLVMType ty, string name> {
  LLVMType Type = ty;
  string Name = name;
}

// Return [AMDGPUArg<basety, names[0]>, AMDGPUArg<LLVMMatchType<0>, names[1]>, ...]
class makeArgList<list<string> names, LLVMType basety> {
  list<AMDGPUArg> ret =
    !listconcat([AMDGPUArg<basety, names[0]>],
                !foreach(name, !tail(names), AMDGPUArg<LLVMMatchType<0>, name>));
}
````
- **L897 EN**: Blank line separating nearby declarations or logic blocks.
  **L897 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L898 EN**: Declares TableGen def `int_amdgcn_prng_b32`.
  **L898 CN**: 声明 TableGen def `int_amdgcn_prng_b32`。
- **L899 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem, IntrNoCreateUndefOrPoison]`.
  **L899 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem, IntrNoCreateUndefOrPoison]`。
- **L900 EN**: Executes a standalone statement or declaration: `>, ClangBuiltin<"__builtin_amdgcn_prng_b32">;`.
  **L900 CN**: 执行一条独立语句或声明：`>, ClangBuiltin<"__builtin_amdgcn_prng_b32">;`。
- **L901 EN**: Blank line separating nearby declarations or logic blocks.
  **L901 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L902 EN**: Declares TableGen def `int_amdgcn_bitop3`.
  **L902 CN**: 声明 TableGen def `int_amdgcn_bitop3`。
- **L903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_anyint_ty],`.
  **L903 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_anyint_ty],`。
- **L904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty],`.
  **L904 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty],`。
- **L905 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<3>>]>;`.
  **L905 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<3>>]>;`。
- **L906 EN**: Blank line separating nearby declarations or logic blocks.
  **L906 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L907 EN**: Continues the surrounding expression or declaration: `} // TargetPrefix = "amdgcn"`.
  **L907 CN**: 继续构造周围的表达式或声明：`} // TargetPrefix = "amdgcn"`。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L909 EN**: Comment explains nearby logic, invariants, or intent: `New-style image intrinsics`.
  **L909 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`New-style image intrinsics`。
- **L910 EN**: Blank line separating nearby declarations or logic blocks.
  **L910 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L911 EN**: Separator comment used for visual grouping.
  **L911 CN**: 用于视觉分组的分隔注释。
- **L912 EN**: Comment explains nearby logic, invariants, or intent: `Dimension-aware image intrinsics framework`.
  **L912 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Dimension-aware image intrinsics framework`。
- **L913 EN**: Separator comment used for visual grouping.
  **L913 CN**: 用于视觉分组的分隔注释。
- **L914 EN**: Blank line separating nearby declarations or logic blocks.
  **L914 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L915 EN**: Comment explains nearby logic, invariants, or intent: `Helper class to represent (type, name) combinations of arguments. The`.
  **L915 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper class to represent (type, name) combinations of arguments. The`。
- **L916 EN**: Comment explains nearby logic, invariants, or intent: `argument names are explanatory and used as DAG operand names for codegen`.
  **L916 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`argument names are explanatory and used as DAG operand names for codegen`。
- **L917 EN**: Comment explains nearby logic, invariants, or intent: `pattern matching.`.
  **L917 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`pattern matching.`。
- **L918 EN**: Declares class `AMDGPUArg<LLVMType`.
  **L918 CN**: 声明 class `AMDGPUArg<LLVMType`。
- **L919 EN**: Initializes variable `Type` from the right-hand expression.
  **L919 CN**: 使用右侧表达式初始化变量 `Type`。
- **L920 EN**: Initializes variable `Name` from the right-hand expression.
  **L920 CN**: 使用右侧表达式初始化变量 `Name`。
- **L921 EN**: Closes the current lexical scope or compound statement.
  **L921 CN**: 结束当前词法作用域或复合语句块。
- **L922 EN**: Blank line separating nearby declarations or logic blocks.
  **L922 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L923 EN**: Comment explains nearby logic, invariants, or intent: `Return [AMDGPUArg<basety, names[0]>, AMDGPUArg<LLVMMatchType<0>, names[1]>, ...]`.
  **L923 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return [AMDGPUArg<basety, names[0]>, AMDGPUArg<LLVMMatchType<0>, names[1]>, ...]`。
- **L924 EN**: Declares class `makeArgList<list<string>`.
  **L924 CN**: 声明 class `makeArgList<list<string>`。
- **L925 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> ret =`.
  **L925 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> ret =`。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([AMDGPUArg<basety, names[0]>],`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([AMDGPUArg<basety, names[0]>],`。
- **L927 EN**: Executes a call or declaration centered on `!foreach`.
  **L927 CN**: 执行以 `!foreach` 为核心的调用或声明。
- **L928 EN**: Closes the current lexical scope or compound statement.
  **L928 CN**: 结束当前词法作用域或复合语句块。

### Lines 929-960

````tablegen

// Return arglist, with LLVMMatchType's references shifted by 'shift'.
class arglistmatchshift<list<AMDGPUArg> arglist, int shift> {
  list<AMDGPUArg> ret =
    !foreach(arg, arglist,
             !if(!isa<LLVMMatchType>(arg.Type),
                 AMDGPUArg<LLVMMatchType<!add(!cast<LLVMMatchType>(arg.Type).OverloadIndex, shift)>,
                           arg.Name>,
                 arg));
}

// Return the concatenation of the given arglists. LLVMMatchType's are adjusted
// accordingly, and shifted by an additional 'shift'.
class arglistconcat<list<list<AMDGPUArg>> arglists, int shift = 0> {
  list<AMDGPUArg> ret =
    !foldl([]<AMDGPUArg>, arglists, lhs, rhs,
           !listconcat(
             lhs,
             arglistmatchshift<rhs,
                               !add(shift, !foldl(0, lhs, a, b,
                                                  !add(a, !isa<LLVMAnyType>(b.Type))))>.ret));
}

// Represent texture/image types / dimensionality.
class AMDGPUDimProps<bits<3> enc, string name, string asmsuffix,
                     list<string> coord_names, list<string> slice_names,
                     bit msaa = 0> {
  AMDGPUDimProps Dim = !cast<AMDGPUDimProps>(NAME);
  string Name = name; // e.g. "2darraymsaa"
  string AsmSuffix = asmsuffix; // e.g. 2D_MSAA_ARRAY (used in assembly strings)
  bits<3> Encoding = enc;
  bit DA = 0; // DA bit in MIMG encoding
````
- **L929 EN**: Blank line separating nearby declarations or logic blocks.
  **L929 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L930 EN**: Comment explains nearby logic, invariants, or intent: `Return arglist, with LLVMMatchType's references shifted by 'shift'.`.
  **L930 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return arglist, with LLVMMatchType's references shifted by 'shift'.`。
- **L931 EN**: Declares class `arglistmatchshift<list<AMDGPUArg>`.
  **L931 CN**: 声明 class `arglistmatchshift<list<AMDGPUArg>`。
- **L932 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> ret =`.
  **L932 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> ret =`。
- **L933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foreach(arg, arglist,`.
  **L933 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foreach(arg, arglist,`。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!isa<LLVMMatchType>(arg.Type),`.
  **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!isa<LLVMMatchType>(arg.Type),`。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUArg<LLVMMatchType<!add(!cast<LLVMMatchType>(arg.Type).OverloadIndex, shift)>,`.
  **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUArg<LLVMMatchType<!add(!cast<LLVMMatchType>(arg.Type).OverloadIndex, shift)>,`。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arg.Name>,`.
  **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`arg.Name>,`。
- **L937 EN**: Executes a standalone statement or declaration: `arg));`.
  **L937 CN**: 执行一条独立语句或声明：`arg));`。
- **L938 EN**: Closes the current lexical scope or compound statement.
  **L938 CN**: 结束当前词法作用域或复合语句块。
- **L939 EN**: Blank line separating nearby declarations or logic blocks.
  **L939 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L940 EN**: Comment explains nearby logic, invariants, or intent: `Return the concatenation of the given arglists. LLVMMatchType's are adjusted`.
  **L940 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return the concatenation of the given arglists. LLVMMatchType's are adjusted`。
- **L941 EN**: Comment explains nearby logic, invariants, or intent: `accordingly, and shifted by an additional 'shift'.`.
  **L941 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`accordingly, and shifted by an additional 'shift'.`。
- **L942 EN**: Declares class `arglistconcat<list<list<AMDGPUArg>>`.
  **L942 CN**: 声明 class `arglistconcat<list<list<AMDGPUArg>>`。
- **L943 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> ret =`.
  **L943 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> ret =`。
- **L944 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<AMDGPUArg>, arglists, lhs, rhs,`.
  **L944 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<AMDGPUArg>, arglists, lhs, rhs,`。
- **L945 EN**: Continues logic associated with callable symbol `listconcat`.
  **L945 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lhs,`.
  **L946 CN**: 继续一个多行参数列表、初始化器或聚合项：`lhs,`。
- **L947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arglistmatchshift<rhs,`.
  **L947 CN**: 继续一个多行参数列表、初始化器或聚合项：`arglistmatchshift<rhs,`。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!add(shift, !foldl(0, lhs, a, b,`.
  **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`!add(shift, !foldl(0, lhs, a, b,`。
- **L949 EN**: Executes a call or declaration centered on `!add`.
  **L949 CN**: 执行以 `!add` 为核心的调用或声明。
- **L950 EN**: Closes the current lexical scope or compound statement.
  **L950 CN**: 结束当前词法作用域或复合语句块。
- **L951 EN**: Blank line separating nearby declarations or logic blocks.
  **L951 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L952 EN**: Comment explains nearby logic, invariants, or intent: `Represent texture/image types / dimensionality.`.
  **L952 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Represent texture/image types / dimensionality.`。
- **L953 EN**: Declares class `AMDGPUDimProps<bits<3>`.
  **L953 CN**: 声明 class `AMDGPUDimProps<bits<3>`。
- **L954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<string> coord_names, list<string> slice_names,`.
  **L954 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<string> coord_names, list<string> slice_names,`。
- **L955 EN**: Continues the surrounding expression or declaration: `bit msaa = 0> {`.
  **L955 CN**: 继续构造周围的表达式或声明：`bit msaa = 0> {`。
- **L956 EN**: Initializes variable `Dim` from the right-hand expression.
  **L956 CN**: 使用右侧表达式初始化变量 `Dim`。
- **L957 EN**: Continues the surrounding expression or declaration: `string Name = name; // e.g. "2darraymsaa"`.
  **L957 CN**: 继续构造周围的表达式或声明：`string Name = name; // e.g. "2darraymsaa"`。
- **L958 EN**: Continues logic associated with callable symbol `D_MSAA_ARRAY`.
  **L958 CN**: 继续与可调用符号 `D_MSAA_ARRAY` 相关的逻辑。
- **L959 EN**: Initializes variable `Encoding` from the right-hand expression.
  **L959 CN**: 使用右侧表达式初始化变量 `Encoding`。
- **L960 EN**: Continues the surrounding expression or declaration: `bit DA = 0; // DA bit in MIMG encoding`.
  **L960 CN**: 继续构造周围的表达式或声明：`bit DA = 0; // DA bit in MIMG encoding`。

### Lines 961-992

````tablegen
  bit MSAA = msaa;

  list<AMDGPUArg> CoordSliceArgs =
    makeArgList<!listconcat(coord_names, slice_names), llvm_anyfloat_ty>.ret;
  list<AMDGPUArg> CoordSliceIntArgs =
    makeArgList<!listconcat(coord_names, slice_names), llvm_anyint_ty>.ret;
  list<AMDGPUArg> GradientArgs =
    makeArgList<!listconcat(!foreach(name, coord_names, "d" # name # "dh"),
                            !foreach(name, coord_names, "d" # name # "dv")),
                llvm_anyfloat_ty>.ret;

  bits<8> NumCoords = !size(CoordSliceArgs);
  bits<8> NumGradients = !size(GradientArgs);
}

def AMDGPUDim1D : AMDGPUDimProps<0x0, "1d", "1D", ["s"], []>;
def AMDGPUDim2D : AMDGPUDimProps<0x1, "2d", "2D", ["s", "t"], []>;
def AMDGPUDim3D : AMDGPUDimProps<0x2, "3d", "3D", ["s", "t", "r"], []>;
let DA = 1 in {
  def AMDGPUDimCube : AMDGPUDimProps<0x3, "cube", "CUBE", ["s", "t"], ["face"]>;
  def AMDGPUDim1DArray : AMDGPUDimProps<0x4, "1darray", "1D_ARRAY", ["s"], ["slice"]>;
  def AMDGPUDim2DArray : AMDGPUDimProps<0x5, "2darray", "2D_ARRAY", ["s", "t"], ["slice"]>;
}
def AMDGPUDim2DMsaa : AMDGPUDimProps<0x6, "2dmsaa", "2D_MSAA", ["s", "t"], ["fragid"], 1>;
let DA = 1 in {
  def AMDGPUDim2DArrayMsaa : AMDGPUDimProps<0x7, "2darraymsaa", "2D_MSAA_ARRAY", ["s", "t"], ["slice", "fragid"], 1>;
}

def AMDGPUDims {
  list<AMDGPUDimProps> NoMsaa = [AMDGPUDim1D, AMDGPUDim2D, AMDGPUDim3D,
                                 AMDGPUDimCube, AMDGPUDim1DArray,
                                 AMDGPUDim2DArray];
````
- **L961 EN**: Initializes variable `MSAA` from the right-hand expression.
  **L961 CN**: 使用右侧表达式初始化变量 `MSAA`。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L963 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> CoordSliceArgs =`.
  **L963 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> CoordSliceArgs =`。
- **L964 EN**: Executes a call or declaration centered on `makeArgList<!listconcat`.
  **L964 CN**: 执行以 `makeArgList<!listconcat` 为核心的调用或声明。
- **L965 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> CoordSliceIntArgs =`.
  **L965 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> CoordSliceIntArgs =`。
- **L966 EN**: Executes a call or declaration centered on `makeArgList<!listconcat`.
  **L966 CN**: 执行以 `makeArgList<!listconcat` 为核心的调用或声明。
- **L967 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> GradientArgs =`.
  **L967 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> GradientArgs =`。
- **L968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `makeArgList<!listconcat(!foreach(name, coord_names, "d" # name # "dh"),`.
  **L968 CN**: 继续一个多行参数列表、初始化器或聚合项：`makeArgList<!listconcat(!foreach(name, coord_names, "d" # name # "dh"),`。
- **L969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foreach(name, coord_names, "d" # name # "dv")),`.
  **L969 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foreach(name, coord_names, "d" # name # "dv")),`。
- **L970 EN**: Executes a standalone statement or declaration: `llvm_anyfloat_ty>.ret;`.
  **L970 CN**: 执行一条独立语句或声明：`llvm_anyfloat_ty>.ret;`。
- **L971 EN**: Blank line separating nearby declarations or logic blocks.
  **L971 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L972 EN**: Initializes variable `NumCoords` from the right-hand expression.
  **L972 CN**: 使用右侧表达式初始化变量 `NumCoords`。
- **L973 EN**: Initializes variable `NumGradients` from the right-hand expression.
  **L973 CN**: 使用右侧表达式初始化变量 `NumGradients`。
- **L974 EN**: Closes the current lexical scope or compound statement.
  **L974 CN**: 结束当前词法作用域或复合语句块。
- **L975 EN**: Blank line separating nearby declarations or logic blocks.
  **L975 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L976 EN**: Declares TableGen def `AMDGPUDim1D`.
  **L976 CN**: 声明 TableGen def `AMDGPUDim1D`。
- **L977 EN**: Declares TableGen def `AMDGPUDim2D`.
  **L977 CN**: 声明 TableGen def `AMDGPUDim2D`。
- **L978 EN**: Declares TableGen def `AMDGPUDim3D`.
  **L978 CN**: 声明 TableGen def `AMDGPUDim3D`。
- **L979 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L979 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L980 EN**: Declares TableGen def `AMDGPUDimCube`.
  **L980 CN**: 声明 TableGen def `AMDGPUDimCube`。
- **L981 EN**: Declares TableGen def `AMDGPUDim1DArray`.
  **L981 CN**: 声明 TableGen def `AMDGPUDim1DArray`。
- **L982 EN**: Declares TableGen def `AMDGPUDim2DArray`.
  **L982 CN**: 声明 TableGen def `AMDGPUDim2DArray`。
- **L983 EN**: Closes the current lexical scope or compound statement.
  **L983 CN**: 结束当前词法作用域或复合语句块。
- **L984 EN**: Declares TableGen def `AMDGPUDim2DMsaa`.
  **L984 CN**: 声明 TableGen def `AMDGPUDim2DMsaa`。
- **L985 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L985 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L986 EN**: Declares TableGen def `AMDGPUDim2DArrayMsaa`.
  **L986 CN**: 声明 TableGen def `AMDGPUDim2DArrayMsaa`。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Blank line separating nearby declarations or logic blocks.
  **L988 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L989 EN**: Declares TableGen def `AMDGPUDims`.
  **L989 CN**: 声明 TableGen def `AMDGPUDims`。
- **L990 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<AMDGPUDimProps> NoMsaa = [AMDGPUDim1D, AMDGPUDim2D, AMDGPUDim3D,`.
  **L990 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<AMDGPUDimProps> NoMsaa = [AMDGPUDim1D, AMDGPUDim2D, AMDGPUDim3D,`。
- **L991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimCube, AMDGPUDim1DArray,`.
  **L991 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimCube, AMDGPUDim1DArray,`。
- **L992 EN**: Executes a standalone statement or declaration: `AMDGPUDim2DArray];`.
  **L992 CN**: 执行一条独立语句或声明：`AMDGPUDim2DArray];`。

### Lines 993-1024

````tablegen
  list<AMDGPUDimProps> Msaa = [AMDGPUDim2DMsaa, AMDGPUDim2DArrayMsaa];
  list<AMDGPUDimProps> All = !listconcat(NoMsaa, Msaa);
}

// Represent sample variants, i.e. _C, _O, _B, ... and combinations thereof.
class AMDGPUSampleVariant<string ucmod, string lcmod, list<AMDGPUArg> extra_addr> {
  string UpperCaseMod = ucmod;
  string LowerCaseMod = lcmod;

  // {offset} {bias} {z-compare}
  list<AMDGPUArg> ExtraAddrArgs = extra_addr;
  bit Offset = false;
  bit Bias = false;
  bit ZCompare = false;
  bit Gradients = false;

  // Name of the {lod} or {clamp} argument that is appended to the coordinates,
  // if any.
  string LodOrClamp = "";

  bit UsesWQM = false;
}

// AMDGPUSampleVariants: all variants supported by IMAGE_SAMPLE
// AMDGPUSampleVariantsNoGradients: variants supported by IMAGE_GATHER4
defset list<AMDGPUSampleVariant> AMDGPUSampleVariants = {
  multiclass AMDGPUSampleHelper_Offset<string ucmod, string lcmod,
                                       list<AMDGPUArg> extra_addr> {
    def NAME#lcmod : AMDGPUSampleVariant<ucmod, lcmod, extra_addr>;
    let Offset = true in
    def NAME#lcmod#_o : AMDGPUSampleVariant<
        ucmod#"_O", lcmod#"_o", !listconcat([AMDGPUArg<llvm_i32_ty, "offset">], extra_addr)>;
````
- **L993 EN**: Initializes variable `Msaa` from the right-hand expression.
  **L993 CN**: 使用右侧表达式初始化变量 `Msaa`。
- **L994 EN**: Initializes variable `All` from the right-hand expression.
  **L994 CN**: 使用右侧表达式初始化变量 `All`。
- **L995 EN**: Closes the current lexical scope or compound statement.
  **L995 CN**: 结束当前词法作用域或复合语句块。
- **L996 EN**: Blank line separating nearby declarations or logic blocks.
  **L996 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L997 EN**: Comment explains nearby logic, invariants, or intent: `Represent sample variants, i.e. _C, _O, _B, ... and combinations thereof.`.
  **L997 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Represent sample variants, i.e. _C, _O, _B, ... and combinations thereof.`。
- **L998 EN**: Declares class `AMDGPUSampleVariant<string`.
  **L998 CN**: 声明 class `AMDGPUSampleVariant<string`。
- **L999 EN**: Initializes variable `UpperCaseMod` from the right-hand expression.
  **L999 CN**: 使用右侧表达式初始化变量 `UpperCaseMod`。
- **L1000 EN**: Initializes variable `LowerCaseMod` from the right-hand expression.
  **L1000 CN**: 使用右侧表达式初始化变量 `LowerCaseMod`。
- **L1001 EN**: Blank line separating nearby declarations or logic blocks.
  **L1001 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1002 EN**: Comment explains nearby logic, invariants, or intent: `{offset} {bias} {z-compare}`.
  **L1002 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`{offset} {bias} {z-compare}`。
- **L1003 EN**: Initializes variable `ExtraAddrArgs` from the right-hand expression.
  **L1003 CN**: 使用右侧表达式初始化变量 `ExtraAddrArgs`。
- **L1004 EN**: Initializes variable `Offset` from the right-hand expression.
  **L1004 CN**: 使用右侧表达式初始化变量 `Offset`。
- **L1005 EN**: Initializes variable `Bias` from the right-hand expression.
  **L1005 CN**: 使用右侧表达式初始化变量 `Bias`。
- **L1006 EN**: Initializes variable `ZCompare` from the right-hand expression.
  **L1006 CN**: 使用右侧表达式初始化变量 `ZCompare`。
- **L1007 EN**: Initializes variable `Gradients` from the right-hand expression.
  **L1007 CN**: 使用右侧表达式初始化变量 `Gradients`。
- **L1008 EN**: Blank line separating nearby declarations or logic blocks.
  **L1008 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1009 EN**: Comment explains nearby logic, invariants, or intent: `Name of the {lod} or {clamp} argument that is appended to the coordinates,`.
  **L1009 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Name of the {lod} or {clamp} argument that is appended to the coordinates,`。
- **L1010 EN**: Comment explains nearby logic, invariants, or intent: `if any.`.
  **L1010 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if any.`。
- **L1011 EN**: Initializes variable `LodOrClamp` from the right-hand expression.
  **L1011 CN**: 使用右侧表达式初始化变量 `LodOrClamp`。
- **L1012 EN**: Blank line separating nearby declarations or logic blocks.
  **L1012 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1013 EN**: Initializes variable `UsesWQM` from the right-hand expression.
  **L1013 CN**: 使用右侧表达式初始化变量 `UsesWQM`。
- **L1014 EN**: Closes the current lexical scope or compound statement.
  **L1014 CN**: 结束当前词法作用域或复合语句块。
- **L1015 EN**: Blank line separating nearby declarations or logic blocks.
  **L1015 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1016 EN**: Comment explains nearby logic, invariants, or intent: `AMDGPUSampleVariants: all variants supported by IMAGE_SAMPLE`.
  **L1016 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AMDGPUSampleVariants: all variants supported by IMAGE_SAMPLE`。
- **L1017 EN**: Comment explains nearby logic, invariants, or intent: `AMDGPUSampleVariantsNoGradients: variants supported by IMAGE_GATHER4`.
  **L1017 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AMDGPUSampleVariantsNoGradients: variants supported by IMAGE_GATHER4`。
- **L1018 EN**: Continues the surrounding expression or declaration: `defset list<AMDGPUSampleVariant> AMDGPUSampleVariants = {`.
  **L1018 CN**: 继续构造周围的表达式或声明：`defset list<AMDGPUSampleVariant> AMDGPUSampleVariants = {`。
- **L1019 EN**: Declares TableGen multiclass `AMDGPUSampleHelper_Offset`.
  **L1019 CN**: 声明 TableGen multiclass `AMDGPUSampleHelper_Offset`。
- **L1020 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> extra_addr> {`.
  **L1020 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> extra_addr> {`。
- **L1021 EN**: Declares TableGen def `NAME#lcmod`.
  **L1021 CN**: 声明 TableGen def `NAME#lcmod`。
- **L1022 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1022 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1023 EN**: Declares TableGen def `NAME#lcmod#_o`.
  **L1023 CN**: 声明 TableGen def `NAME#lcmod#_o`。
- **L1024 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1024 CN**: 执行以 `!listconcat` 为核心的调用或声明。

### Lines 1025-1056

````tablegen
  }

  multiclass AMDGPUSampleHelper_Compare<string ucmod, string lcmod,
                                        list<AMDGPUArg> extra_addr> {
    defm NAME : AMDGPUSampleHelper_Offset<ucmod, lcmod, extra_addr>;
    let ZCompare = true in
    defm NAME : AMDGPUSampleHelper_Offset<
        "_C"#ucmod, "_c"#lcmod, !listconcat(extra_addr, [AMDGPUArg<llvm_float_ty, "zcompare">])>;
  }

  multiclass AMDGPUSampleHelper_Clamp<string ucmod, string lcmod,
                                      list<AMDGPUArg> extra_addr> {
    defm NAME : AMDGPUSampleHelper_Compare<ucmod, lcmod, extra_addr>;
    let LodOrClamp = "clamp" in
    defm NAME : AMDGPUSampleHelper_Compare<ucmod#"_CL", lcmod#"_cl", extra_addr>;
  }

  defset list<AMDGPUSampleVariant> AMDGPUSampleVariantsNoGradients = {
    let UsesWQM = true in
    defm AMDGPUSample : AMDGPUSampleHelper_Clamp<"", "", []>;
    let Bias = true, UsesWQM = true in
    defm AMDGPUSample : AMDGPUSampleHelper_Clamp<
        "_B", "_b", [AMDGPUArg<llvm_anyfloat_ty, "bias">]>;
    let LodOrClamp = "lod" in
    defm AMDGPUSample : AMDGPUSampleHelper_Compare<"_L", "_l", []>;
    defm AMDGPUSample : AMDGPUSampleHelper_Compare<"_LZ", "_lz", []>;
  }

  let Gradients = true in {
    defm AMDGPUSample : AMDGPUSampleHelper_Clamp<"_D", "_d", []>;
    defm AMDGPUSample : AMDGPUSampleHelper_Clamp<"_CD", "_cd", []>;
  }
````
- **L1025 EN**: Closes the current lexical scope or compound statement.
  **L1025 CN**: 结束当前词法作用域或复合语句块。
- **L1026 EN**: Blank line separating nearby declarations or logic blocks.
  **L1026 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1027 EN**: Declares TableGen multiclass `AMDGPUSampleHelper_Compare`.
  **L1027 CN**: 声明 TableGen multiclass `AMDGPUSampleHelper_Compare`。
- **L1028 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> extra_addr> {`.
  **L1028 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> extra_addr> {`。
- **L1029 EN**: Declares TableGen defm `NAME`.
  **L1029 CN**: 声明 TableGen defm `NAME`。
- **L1030 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1030 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1031 EN**: Declares TableGen defm `NAME`.
  **L1031 CN**: 声明 TableGen defm `NAME`。
- **L1032 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1032 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1033 EN**: Closes the current lexical scope or compound statement.
  **L1033 CN**: 结束当前词法作用域或复合语句块。
- **L1034 EN**: Blank line separating nearby declarations or logic blocks.
  **L1034 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1035 EN**: Declares TableGen multiclass `AMDGPUSampleHelper_Clamp`.
  **L1035 CN**: 声明 TableGen multiclass `AMDGPUSampleHelper_Clamp`。
- **L1036 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> extra_addr> {`.
  **L1036 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> extra_addr> {`。
- **L1037 EN**: Declares TableGen defm `NAME`.
  **L1037 CN**: 声明 TableGen defm `NAME`。
- **L1038 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1038 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1039 EN**: Declares TableGen defm `NAME`.
  **L1039 CN**: 声明 TableGen defm `NAME`。
- **L1040 EN**: Closes the current lexical scope or compound statement.
  **L1040 CN**: 结束当前词法作用域或复合语句块。
- **L1041 EN**: Blank line separating nearby declarations or logic blocks.
  **L1041 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1042 EN**: Continues the surrounding expression or declaration: `defset list<AMDGPUSampleVariant> AMDGPUSampleVariantsNoGradients = {`.
  **L1042 CN**: 继续构造周围的表达式或声明：`defset list<AMDGPUSampleVariant> AMDGPUSampleVariantsNoGradients = {`。
- **L1043 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1043 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1044 EN**: Declares TableGen defm `AMDGPUSample`.
  **L1044 CN**: 声明 TableGen defm `AMDGPUSample`。
- **L1045 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1045 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1046 EN**: Declares TableGen defm `AMDGPUSample`.
  **L1046 CN**: 声明 TableGen defm `AMDGPUSample`。
- **L1047 EN**: Executes a standalone statement or declaration: `"_B", "_b", [AMDGPUArg<llvm_anyfloat_ty, "bias">]>;`.
  **L1047 CN**: 执行一条独立语句或声明：`"_B", "_b", [AMDGPUArg<llvm_anyfloat_ty, "bias">]>;`。
- **L1048 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1048 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1049 EN**: Declares TableGen defm `AMDGPUSample`.
  **L1049 CN**: 声明 TableGen defm `AMDGPUSample`。
- **L1050 EN**: Declares TableGen defm `AMDGPUSample`.
  **L1050 CN**: 声明 TableGen defm `AMDGPUSample`。
- **L1051 EN**: Closes the current lexical scope or compound statement.
  **L1051 CN**: 结束当前词法作用域或复合语句块。
- **L1052 EN**: Blank line separating nearby declarations or logic blocks.
  **L1052 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1053 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1053 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1054 EN**: Declares TableGen defm `AMDGPUSample`.
  **L1054 CN**: 声明 TableGen defm `AMDGPUSample`。
- **L1055 EN**: Declares TableGen defm `AMDGPUSample`.
  **L1055 CN**: 声明 TableGen defm `AMDGPUSample`。
- **L1056 EN**: Closes the current lexical scope or compound statement.
  **L1056 CN**: 结束当前词法作用域或复合语句块。

### Lines 1057-1088

````tablegen
}

// Helper class to capture the profile of a dimension-aware image intrinsic.
// This information is used to generate the intrinsic's type and to inform
// codegen pattern matching.
class AMDGPUDimProfile<string opmod,
                       AMDGPUDimProps dim> {
  AMDGPUDimProps Dim = dim;
  string OpMod = opmod; // the corresponding instruction is named IMAGE_OpMod

  // These are intended to be overwritten by subclasses
  bit IsSample = false;
  bit IsAtomic = false;
  list<LLVMType> RetTypes = [];
  list<AMDGPUArg> DataArgs = [];
  list<AMDGPUArg> ExtraAddrArgs = [];
  bit Offset = false;
  bit Bias = false;
  bit ZCompare = false;
  bit Gradients = false;
  string LodClampMip = "";

  int NumRetAndDataAnyTypes =
    !foldl(0, !listconcat(RetTypes, !foreach(arg, DataArgs, arg.Type)), a, b,
           !add(a, !isa<LLVMAnyType>(b)));

  list<AMDGPUArg> AddrArgs =
    arglistconcat<[ExtraAddrArgs,
                   !if(Gradients, dim.GradientArgs, []),
                   !listconcat(!if(IsSample, dim.CoordSliceArgs, dim.CoordSliceIntArgs),
                               !if(!empty(LodClampMip),
                                   []<AMDGPUArg>,
````
- **L1057 EN**: Closes the current lexical scope or compound statement.
  **L1057 CN**: 结束当前词法作用域或复合语句块。
- **L1058 EN**: Blank line separating nearby declarations or logic blocks.
  **L1058 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1059 EN**: Comment explains nearby logic, invariants, or intent: `Helper class to capture the profile of a dimension-aware image intrinsic.`.
  **L1059 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper class to capture the profile of a dimension-aware image intrinsic.`。
- **L1060 EN**: Comment explains nearby logic, invariants, or intent: `This information is used to generate the intrinsic's type and to inform`.
  **L1060 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This information is used to generate the intrinsic's type and to inform`。
- **L1061 EN**: Comment explains nearby logic, invariants, or intent: `codegen pattern matching.`.
  **L1061 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`codegen pattern matching.`。
- **L1062 EN**: Declares class `AMDGPUDimProfile<string`.
  **L1062 CN**: 声明 class `AMDGPUDimProfile<string`。
- **L1063 EN**: Continues the surrounding expression or declaration: `AMDGPUDimProps dim> {`.
  **L1063 CN**: 继续构造周围的表达式或声明：`AMDGPUDimProps dim> {`。
- **L1064 EN**: Initializes variable `Dim` from the right-hand expression.
  **L1064 CN**: 使用右侧表达式初始化变量 `Dim`。
- **L1065 EN**: Continues the surrounding expression or declaration: `string OpMod = opmod; // the corresponding instruction is named IMAGE_OpMod`.
  **L1065 CN**: 继续构造周围的表达式或声明：`string OpMod = opmod; // the corresponding instruction is named IMAGE_OpMod`。
- **L1066 EN**: Blank line separating nearby declarations or logic blocks.
  **L1066 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1067 EN**: Comment explains nearby logic, invariants, or intent: `These are intended to be overwritten by subclasses`.
  **L1067 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These are intended to be overwritten by subclasses`。
- **L1068 EN**: Initializes variable `IsSample` from the right-hand expression.
  **L1068 CN**: 使用右侧表达式初始化变量 `IsSample`。
- **L1069 EN**: Initializes variable `IsAtomic` from the right-hand expression.
  **L1069 CN**: 使用右侧表达式初始化变量 `IsAtomic`。
- **L1070 EN**: Initializes variable `RetTypes` from the right-hand expression.
  **L1070 CN**: 使用右侧表达式初始化变量 `RetTypes`。
- **L1071 EN**: Initializes variable `DataArgs` from the right-hand expression.
  **L1071 CN**: 使用右侧表达式初始化变量 `DataArgs`。
- **L1072 EN**: Initializes variable `ExtraAddrArgs` from the right-hand expression.
  **L1072 CN**: 使用右侧表达式初始化变量 `ExtraAddrArgs`。
- **L1073 EN**: Initializes variable `Offset` from the right-hand expression.
  **L1073 CN**: 使用右侧表达式初始化变量 `Offset`。
- **L1074 EN**: Initializes variable `Bias` from the right-hand expression.
  **L1074 CN**: 使用右侧表达式初始化变量 `Bias`。
- **L1075 EN**: Initializes variable `ZCompare` from the right-hand expression.
  **L1075 CN**: 使用右侧表达式初始化变量 `ZCompare`。
- **L1076 EN**: Initializes variable `Gradients` from the right-hand expression.
  **L1076 CN**: 使用右侧表达式初始化变量 `Gradients`。
- **L1077 EN**: Initializes variable `LodClampMip` from the right-hand expression.
  **L1077 CN**: 使用右侧表达式初始化变量 `LodClampMip`。
- **L1078 EN**: Blank line separating nearby declarations or logic blocks.
  **L1078 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1079 EN**: Continues the surrounding expression or declaration: `int NumRetAndDataAnyTypes =`.
  **L1079 CN**: 继续构造周围的表达式或声明：`int NumRetAndDataAnyTypes =`。
- **L1080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl(0, !listconcat(RetTypes, !foreach(arg, DataArgs, arg.Type)), a, b,`.
  **L1080 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl(0, !listconcat(RetTypes, !foreach(arg, DataArgs, arg.Type)), a, b,`。
- **L1081 EN**: Executes a call or declaration centered on `!add`.
  **L1081 CN**: 执行以 `!add` 为核心的调用或声明。
- **L1082 EN**: Blank line separating nearby declarations or logic blocks.
  **L1082 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1083 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> AddrArgs =`.
  **L1083 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> AddrArgs =`。
- **L1084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arglistconcat<[ExtraAddrArgs,`.
  **L1084 CN**: 继续一个多行参数列表、初始化器或聚合项：`arglistconcat<[ExtraAddrArgs,`。
- **L1085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(Gradients, dim.GradientArgs, []),`.
  **L1085 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(Gradients, dim.GradientArgs, []),`。
- **L1086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(!if(IsSample, dim.CoordSliceArgs, dim.CoordSliceIntArgs),`.
  **L1086 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(!if(IsSample, dim.CoordSliceArgs, dim.CoordSliceIntArgs),`。
- **L1087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!empty(LodClampMip),`.
  **L1087 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!empty(LodClampMip),`。
- **L1088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[]<AMDGPUArg>,`.
  **L1088 CN**: 继续一个多行参数列表、初始化器或聚合项：`[]<AMDGPUArg>,`。

### Lines 1089-1120

````tablegen
                                   [AMDGPUArg<LLVMMatchType<0>, LodClampMip>]))],
                  NumRetAndDataAnyTypes>.ret;
  list<LLVMType> AddrTypes = !foreach(arg, AddrArgs, arg.Type);
  list<AMDGPUArg> AddrDefaultArgs =
    !foreach(arg, AddrArgs,
             AMDGPUArg<!if(!or(!isa<LLVMAnyType>(arg.Type), !isa<LLVMMatchType>(arg.Type)),
                           !if(IsSample, llvm_float_ty, llvm_i32_ty), arg.Type),
                       arg.Name>);
  list<AMDGPUArg> AddrA16Args =
    !foreach(arg, AddrArgs,
             AMDGPUArg<!if(!or(!isa<LLVMAnyType>(arg.Type), !isa<LLVMMatchType>(arg.Type)),
                           !if(IsSample, llvm_half_ty, llvm_i16_ty), arg.Type),
                       arg.Name>);
}

class AMDGPUDimProfileCopy<AMDGPUDimProfile base> : AMDGPUDimProfile<base.OpMod, base.Dim> {
  let IsSample = base.IsSample;
  let IsAtomic = base.IsAtomic;
  let RetTypes = base.RetTypes;
  let DataArgs = base.DataArgs;
  let ExtraAddrArgs = base.ExtraAddrArgs;
  let Offset = base.Offset;
  let Bias = base.Bias;
  let ZCompare = base.ZCompare;
  let Gradients = base.Gradients;
  let LodClampMip = base.LodClampMip;
}

class AMDGPUDimSampleProfile<string opmod,
                             AMDGPUDimProps dim,
                             AMDGPUSampleVariant sample,
                             bit has_return = true> : AMDGPUDimProfile<opmod, dim> {
````
- **L1089 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[AMDGPUArg<LLVMMatchType<0>, LodClampMip>]))],`.
  **L1089 CN**: 继续一个多行参数列表、初始化器或聚合项：`[AMDGPUArg<LLVMMatchType<0>, LodClampMip>]))],`。
- **L1090 EN**: Executes a standalone statement or declaration: `NumRetAndDataAnyTypes>.ret;`.
  **L1090 CN**: 执行一条独立语句或声明：`NumRetAndDataAnyTypes>.ret;`。
- **L1091 EN**: Initializes variable `AddrTypes` from the right-hand expression.
  **L1091 CN**: 使用右侧表达式初始化变量 `AddrTypes`。
- **L1092 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> AddrDefaultArgs =`.
  **L1092 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> AddrDefaultArgs =`。
- **L1093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foreach(arg, AddrArgs,`.
  **L1093 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foreach(arg, AddrArgs,`。
- **L1094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUArg<!if(!or(!isa<LLVMAnyType>(arg.Type), !isa<LLVMMatchType>(arg.Type)),`.
  **L1094 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUArg<!if(!or(!isa<LLVMAnyType>(arg.Type), !isa<LLVMMatchType>(arg.Type)),`。
- **L1095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(IsSample, llvm_float_ty, llvm_i32_ty), arg.Type),`.
  **L1095 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(IsSample, llvm_float_ty, llvm_i32_ty), arg.Type),`。
- **L1096 EN**: Executes a standalone statement or declaration: `arg.Name>);`.
  **L1096 CN**: 执行一条独立语句或声明：`arg.Name>);`。
- **L1097 EN**: Continues the surrounding expression or declaration: `list<AMDGPUArg> AddrA16Args =`.
  **L1097 CN**: 继续构造周围的表达式或声明：`list<AMDGPUArg> AddrA16Args =`。
- **L1098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foreach(arg, AddrArgs,`.
  **L1098 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foreach(arg, AddrArgs,`。
- **L1099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUArg<!if(!or(!isa<LLVMAnyType>(arg.Type), !isa<LLVMMatchType>(arg.Type)),`.
  **L1099 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUArg<!if(!or(!isa<LLVMAnyType>(arg.Type), !isa<LLVMMatchType>(arg.Type)),`。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(IsSample, llvm_half_ty, llvm_i16_ty), arg.Type),`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(IsSample, llvm_half_ty, llvm_i16_ty), arg.Type),`。
- **L1101 EN**: Executes a standalone statement or declaration: `arg.Name>);`.
  **L1101 CN**: 执行一条独立语句或声明：`arg.Name>);`。
- **L1102 EN**: Closes the current lexical scope or compound statement.
  **L1102 CN**: 结束当前词法作用域或复合语句块。
- **L1103 EN**: Blank line separating nearby declarations or logic blocks.
  **L1103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1104 EN**: Declares class `AMDGPUDimProfileCopy<AMDGPUDimProfile`.
  **L1104 CN**: 声明 class `AMDGPUDimProfileCopy<AMDGPUDimProfile`。
- **L1105 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1105 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1106 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1106 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1107 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1107 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1108 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1108 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1109 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1109 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1110 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1110 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1111 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1111 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1112 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1112 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1113 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1113 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1114 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1114 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1115 EN**: Closes the current lexical scope or compound statement.
  **L1115 CN**: 结束当前词法作用域或复合语句块。
- **L1116 EN**: Blank line separating nearby declarations or logic blocks.
  **L1116 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1117 EN**: Declares class `AMDGPUDimSampleProfile<string`.
  **L1117 CN**: 声明 class `AMDGPUDimSampleProfile<string`。
- **L1118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimProps dim,`.
  **L1118 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimProps dim,`。
- **L1119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUSampleVariant sample,`.
  **L1119 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUSampleVariant sample,`。
- **L1120 EN**: Continues the surrounding expression or declaration: `bit has_return = true> : AMDGPUDimProfile<opmod, dim> {`.
  **L1120 CN**: 继续构造周围的表达式或声明：`bit has_return = true> : AMDGPUDimProfile<opmod, dim> {`。

### Lines 1121-1152

````tablegen
  let IsSample = true;
  let RetTypes = !if(has_return, [llvm_any_ty], []);
  let ExtraAddrArgs = sample.ExtraAddrArgs;
  let Offset = sample.Offset;
  let Bias = sample.Bias;
  let ZCompare = sample.ZCompare;
  let Gradients = sample.Gradients;
  let LodClampMip = sample.LodOrClamp;
}

class AMDGPUDimSampleNoReturnProfile<string opmod,
                             AMDGPUDimProps dim,
                             AMDGPUSampleVariant sample>
    : AMDGPUDimSampleProfile<opmod, dim, sample, false> {
}

class AMDGPUDimNoSampleProfile<string opmod,
                               AMDGPUDimProps dim,
                               list<LLVMType> retty,
                               list<AMDGPUArg> dataargs,
                               bit Mip = false> : AMDGPUDimProfile<opmod, dim> {
  let RetTypes = retty;
  let DataArgs = dataargs;
  let LodClampMip = !if(Mip, "mip", "");
}

class AMDGPUDimAtomicProfile<string opmod,
                             AMDGPUDimProps dim,
                             list<AMDGPUArg> dataargs,
                             LLVMType rettype> : AMDGPUDimProfile<opmod, dim> {
  let RetTypes = [rettype];
  let DataArgs = dataargs;
````
- **L1121 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1121 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1122 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1122 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1123 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1123 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1124 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1124 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1125 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1125 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1126 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1126 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1127 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1127 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1128 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1128 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1129 EN**: Closes the current lexical scope or compound statement.
  **L1129 CN**: 结束当前词法作用域或复合语句块。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1131 EN**: Declares class `AMDGPUDimSampleNoReturnProfile<string`.
  **L1131 CN**: 声明 class `AMDGPUDimSampleNoReturnProfile<string`。
- **L1132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimProps dim,`.
  **L1132 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimProps dim,`。
- **L1133 EN**: Continues the surrounding expression or declaration: `AMDGPUSampleVariant sample>`.
  **L1133 CN**: 继续构造周围的表达式或声明：`AMDGPUSampleVariant sample>`。
- **L1134 EN**: Continues the surrounding expression or declaration: `: AMDGPUDimSampleProfile<opmod, dim, sample, false> {`.
  **L1134 CN**: 继续构造周围的表达式或声明：`: AMDGPUDimSampleProfile<opmod, dim, sample, false> {`。
- **L1135 EN**: Closes the current lexical scope or compound statement.
  **L1135 CN**: 结束当前词法作用域或复合语句块。
- **L1136 EN**: Blank line separating nearby declarations or logic blocks.
  **L1136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1137 EN**: Declares class `AMDGPUDimNoSampleProfile<string`.
  **L1137 CN**: 声明 class `AMDGPUDimNoSampleProfile<string`。
- **L1138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimProps dim,`.
  **L1138 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimProps dim,`。
- **L1139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> retty,`.
  **L1139 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> retty,`。
- **L1140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<AMDGPUArg> dataargs,`.
  **L1140 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<AMDGPUArg> dataargs,`。
- **L1141 EN**: Continues the surrounding expression or declaration: `bit Mip = false> : AMDGPUDimProfile<opmod, dim> {`.
  **L1141 CN**: 继续构造周围的表达式或声明：`bit Mip = false> : AMDGPUDimProfile<opmod, dim> {`。
- **L1142 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1142 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1143 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1143 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1144 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1144 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1145 EN**: Closes the current lexical scope or compound statement.
  **L1145 CN**: 结束当前词法作用域或复合语句块。
- **L1146 EN**: Blank line separating nearby declarations or logic blocks.
  **L1146 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1147 EN**: Declares class `AMDGPUDimAtomicProfile<string`.
  **L1147 CN**: 声明 class `AMDGPUDimAtomicProfile<string`。
- **L1148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimProps dim,`.
  **L1148 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimProps dim,`。
- **L1149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<AMDGPUArg> dataargs,`.
  **L1149 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<AMDGPUArg> dataargs,`。
- **L1150 EN**: Continues the surrounding expression or declaration: `LLVMType rettype> : AMDGPUDimProfile<opmod, dim> {`.
  **L1150 CN**: 继续构造周围的表达式或声明：`LLVMType rettype> : AMDGPUDimProfile<opmod, dim> {`。
- **L1151 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1151 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1152 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1152 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1153-1184

````tablegen
  let IsAtomic = true;
}

class AMDGPUDimGetResInfoProfile<AMDGPUDimProps dim>
    : AMDGPUDimProfile<"GET_RESINFO", dim> {
  let RetTypes = [llvm_anyfloat_ty];
  let DataArgs = [];
  let AddrArgs = [AMDGPUArg<llvm_anyint_ty, "mip">];
  let LodClampMip = "mip";
}

// Helper class for figuring out image intrinsic argument indexes.
class AMDGPUImageDimIntrinsicEval<AMDGPUDimProfile P_> {
  int NumDataArgs = !size(P_.DataArgs);
  int NumDmaskArgs = !not(P_.IsAtomic);
  int NumOffsetArgs = !if(P_.Offset, 1, 0);
  int NumBiasArgs = !if(P_.Bias, 1, 0);
  int NumZCompareArgs = !if(P_.ZCompare, 1, 0);
  int NumExtraAddrArgs = !add(NumOffsetArgs, NumBiasArgs, NumZCompareArgs);
  int NumVAddrArgs = !size(P_.AddrArgs);
  int NumGradientArgs = !if(P_.Gradients, !size(P_.Dim.GradientArgs), 0);
  int NumCoordArgs = !if(P_.IsSample, !size(P_.Dim.CoordSliceArgs), !size(P_.Dim.CoordSliceIntArgs));
  int NumRSrcArgs = 1;
  int NumSampArgs = !if(P_.IsSample, 2, 0);
  int DmaskArgIndex = NumDataArgs;
  int VAddrArgIndex = !add(DmaskArgIndex, NumDmaskArgs);
  int OffsetArgIndex = VAddrArgIndex;
  int BiasArgIndex = !add(VAddrArgIndex, NumOffsetArgs);
  int ZCompareArgIndex = !add(BiasArgIndex, NumBiasArgs);
  int GradientArgIndex = !add(VAddrArgIndex, NumExtraAddrArgs);
  int CoordArgIndex = !add(GradientArgIndex, NumGradientArgs);
  int LodArgIndex = !add(VAddrArgIndex, NumVAddrArgs, -1);
````
- **L1153 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1153 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1154 EN**: Closes the current lexical scope or compound statement.
  **L1154 CN**: 结束当前词法作用域或复合语句块。
- **L1155 EN**: Blank line separating nearby declarations or logic blocks.
  **L1155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1156 EN**: Declares class `AMDGPUDimGetResInfoProfile<AMDGPUDimProps`.
  **L1156 CN**: 声明 class `AMDGPUDimGetResInfoProfile<AMDGPUDimProps`。
- **L1157 EN**: Continues the surrounding expression or declaration: `: AMDGPUDimProfile<"GET_RESINFO", dim> {`.
  **L1157 CN**: 继续构造周围的表达式或声明：`: AMDGPUDimProfile<"GET_RESINFO", dim> {`。
- **L1158 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1158 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1159 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1159 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1160 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1160 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1161 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1161 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1162 EN**: Closes the current lexical scope or compound statement.
  **L1162 CN**: 结束当前词法作用域或复合语句块。
- **L1163 EN**: Blank line separating nearby declarations or logic blocks.
  **L1163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1164 EN**: Comment explains nearby logic, invariants, or intent: `Helper class for figuring out image intrinsic argument indexes.`.
  **L1164 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper class for figuring out image intrinsic argument indexes.`。
- **L1165 EN**: Declares class `AMDGPUImageDimIntrinsicEval<AMDGPUDimProfile`.
  **L1165 CN**: 声明 class `AMDGPUImageDimIntrinsicEval<AMDGPUDimProfile`。
- **L1166 EN**: Initializes variable `NumDataArgs` from the right-hand expression.
  **L1166 CN**: 使用右侧表达式初始化变量 `NumDataArgs`。
- **L1167 EN**: Initializes variable `NumDmaskArgs` from the right-hand expression.
  **L1167 CN**: 使用右侧表达式初始化变量 `NumDmaskArgs`。
- **L1168 EN**: Initializes variable `NumOffsetArgs` from the right-hand expression.
  **L1168 CN**: 使用右侧表达式初始化变量 `NumOffsetArgs`。
- **L1169 EN**: Initializes variable `NumBiasArgs` from the right-hand expression.
  **L1169 CN**: 使用右侧表达式初始化变量 `NumBiasArgs`。
- **L1170 EN**: Initializes variable `NumZCompareArgs` from the right-hand expression.
  **L1170 CN**: 使用右侧表达式初始化变量 `NumZCompareArgs`。
- **L1171 EN**: Initializes variable `NumExtraAddrArgs` from the right-hand expression.
  **L1171 CN**: 使用右侧表达式初始化变量 `NumExtraAddrArgs`。
- **L1172 EN**: Initializes variable `NumVAddrArgs` from the right-hand expression.
  **L1172 CN**: 使用右侧表达式初始化变量 `NumVAddrArgs`。
- **L1173 EN**: Initializes variable `NumGradientArgs` from the right-hand expression.
  **L1173 CN**: 使用右侧表达式初始化变量 `NumGradientArgs`。
- **L1174 EN**: Initializes variable `NumCoordArgs` from the right-hand expression.
  **L1174 CN**: 使用右侧表达式初始化变量 `NumCoordArgs`。
- **L1175 EN**: Initializes variable `NumRSrcArgs` from the right-hand expression.
  **L1175 CN**: 使用右侧表达式初始化变量 `NumRSrcArgs`。
- **L1176 EN**: Initializes variable `NumSampArgs` from the right-hand expression.
  **L1176 CN**: 使用右侧表达式初始化变量 `NumSampArgs`。
- **L1177 EN**: Initializes variable `DmaskArgIndex` from the right-hand expression.
  **L1177 CN**: 使用右侧表达式初始化变量 `DmaskArgIndex`。
- **L1178 EN**: Initializes variable `VAddrArgIndex` from the right-hand expression.
  **L1178 CN**: 使用右侧表达式初始化变量 `VAddrArgIndex`。
- **L1179 EN**: Initializes variable `OffsetArgIndex` from the right-hand expression.
  **L1179 CN**: 使用右侧表达式初始化变量 `OffsetArgIndex`。
- **L1180 EN**: Initializes variable `BiasArgIndex` from the right-hand expression.
  **L1180 CN**: 使用右侧表达式初始化变量 `BiasArgIndex`。
- **L1181 EN**: Initializes variable `ZCompareArgIndex` from the right-hand expression.
  **L1181 CN**: 使用右侧表达式初始化变量 `ZCompareArgIndex`。
- **L1182 EN**: Initializes variable `GradientArgIndex` from the right-hand expression.
  **L1182 CN**: 使用右侧表达式初始化变量 `GradientArgIndex`。
- **L1183 EN**: Initializes variable `CoordArgIndex` from the right-hand expression.
  **L1183 CN**: 使用右侧表达式初始化变量 `CoordArgIndex`。
- **L1184 EN**: Initializes variable `LodArgIndex` from the right-hand expression.
  **L1184 CN**: 使用右侧表达式初始化变量 `LodArgIndex`。

### Lines 1185-1216

````tablegen
  int MipArgIndex = LodArgIndex;
  int RsrcArgIndex = !add(VAddrArgIndex, NumVAddrArgs);
  int SampArgIndex = !add(RsrcArgIndex, NumRSrcArgs);
  int UnormArgIndex = !add(SampArgIndex, 1);
  int TexFailCtrlArgIndex = !add(SampArgIndex, NumSampArgs);
  int CachePolicyArgIndex = !add(TexFailCtrlArgIndex, 1);
}

// All dimension-aware intrinsics are derived from this class.
class AMDGPUImageDimIntrinsic<AMDGPUDimProfile P_,
                              list<IntrinsicProperty> props,
                              list<SDNodeProperty> sdnodeprops> : Intrinsic<
    P_.RetTypes,        // vdata(VGPR) -- for load/atomic-with-return
    !listconcat(
      !foreach(arg, P_.DataArgs, arg.Type),    // vdata(VGPR) -- for store/atomic
      !if(P_.IsAtomic, [], [llvm_i32_ty]),     // dmask(imm)
      P_.AddrTypes,                            // vaddr(VGPR)
      [llvm_any_ty],                           // rsrc(SGPR); Valid types: v4i32 and v8i32
      !if(P_.IsSample, [llvm_any_ty,           // samp(SGPR);
                        llvm_i1_ty], []),      // unorm(imm)
      [llvm_i32_ty,                            // texfailctrl(imm; bit 0 = tfe, bit 1 = lwe)
       llvm_i32_ty]),                          // auxiliary/cachepolicy(imm):
                                               //                bit 0 = glc, bit 1 = slc,
                                               //                bit 2 = dlc (gfx10/gfx11),
                                               //                bit 4 = scc (gfx90a)
                                               //        gfx942: bit 0 = sc0, bit 1 = nt, bit 4 = sc1
                                               //        gfx12+: bits [0-2] = th, bits [3-4] = scope
     !listconcat(props, [IntrNoCallback, IntrNoFree, IntrWillReturn],
          !if(P_.IsAtomic, [], [ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.DmaskArgIndex>>]),
          !if(P_.IsSample, [ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.UnormArgIndex>>], []),
          [ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.TexFailCtrlArgIndex>>,
           ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.CachePolicyArgIndex>>],
````
- **L1185 EN**: Initializes variable `MipArgIndex` from the right-hand expression.
  **L1185 CN**: 使用右侧表达式初始化变量 `MipArgIndex`。
- **L1186 EN**: Initializes variable `RsrcArgIndex` from the right-hand expression.
  **L1186 CN**: 使用右侧表达式初始化变量 `RsrcArgIndex`。
- **L1187 EN**: Initializes variable `SampArgIndex` from the right-hand expression.
  **L1187 CN**: 使用右侧表达式初始化变量 `SampArgIndex`。
- **L1188 EN**: Initializes variable `UnormArgIndex` from the right-hand expression.
  **L1188 CN**: 使用右侧表达式初始化变量 `UnormArgIndex`。
- **L1189 EN**: Initializes variable `TexFailCtrlArgIndex` from the right-hand expression.
  **L1189 CN**: 使用右侧表达式初始化变量 `TexFailCtrlArgIndex`。
- **L1190 EN**: Initializes variable `CachePolicyArgIndex` from the right-hand expression.
  **L1190 CN**: 使用右侧表达式初始化变量 `CachePolicyArgIndex`。
- **L1191 EN**: Closes the current lexical scope or compound statement.
  **L1191 CN**: 结束当前词法作用域或复合语句块。
- **L1192 EN**: Blank line separating nearby declarations or logic blocks.
  **L1192 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1193 EN**: Comment explains nearby logic, invariants, or intent: `All dimension-aware intrinsics are derived from this class.`.
  **L1193 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All dimension-aware intrinsics are derived from this class.`。
- **L1194 EN**: Declares class `AMDGPUImageDimIntrinsic<AMDGPUDimProfile`.
  **L1194 CN**: 声明 class `AMDGPUImageDimIntrinsic<AMDGPUDimProfile`。
- **L1195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<IntrinsicProperty> props,`.
  **L1195 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<IntrinsicProperty> props,`。
- **L1196 EN**: Continues the surrounding expression or declaration: `list<SDNodeProperty> sdnodeprops> : Intrinsic<`.
  **L1196 CN**: 继续构造周围的表达式或声明：`list<SDNodeProperty> sdnodeprops> : Intrinsic<`。
- **L1197 EN**: Continues logic associated with callable symbol `vdata`.
  **L1197 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1198 EN**: Continues logic associated with callable symbol `listconcat`.
  **L1198 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L1199 EN**: Continues logic associated with callable symbol `foreach`.
  **L1199 CN**: 继续与可调用符号 `foreach` 相关的逻辑。
- **L1200 EN**: Continues the surrounding expression or declaration: `!if(P_.IsAtomic, [], [llvm_i32_ty]),     // dmask(imm)`.
  **L1200 CN**: 继续构造周围的表达式或声明：`!if(P_.IsAtomic, [], [llvm_i32_ty]),     // dmask(imm)`。
- **L1201 EN**: Continues logic associated with callable symbol `vaddr`.
  **L1201 CN**: 继续与可调用符号 `vaddr` 相关的逻辑。
- **L1202 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1202 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1203 EN**: Executes a call or declaration centered on `!if`.
  **L1203 CN**: 执行以 `!if` 为核心的调用或声明。
- **L1204 EN**: Continues logic associated with callable symbol `unorm`.
  **L1204 CN**: 继续与可调用符号 `unorm` 相关的逻辑。
- **L1205 EN**: Continues logic associated with callable symbol `texfailctrl`.
  **L1205 CN**: 继续与可调用符号 `texfailctrl` 相关的逻辑。
- **L1206 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1206 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1207 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc,`.
  **L1207 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc,`。
- **L1208 EN**: Comment explains nearby logic, invariants, or intent: `bit 2 = dlc (gfx10/gfx11),`.
  **L1208 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 2 = dlc (gfx10/gfx11),`。
- **L1209 EN**: Comment explains nearby logic, invariants, or intent: `bit 4 = scc (gfx90a)`.
  **L1209 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 4 = scc (gfx90a)`。
- **L1210 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 4 = sc1`.
  **L1210 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 4 = sc1`。
- **L1211 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope`.
  **L1211 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope`。
- **L1212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(props, [IntrNoCallback, IntrNoFree, IntrWillReturn],`.
  **L1212 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(props, [IntrNoCallback, IntrNoFree, IntrWillReturn],`。
- **L1213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(P_.IsAtomic, [], [ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.DmaskArgIndex>>]),`.
  **L1213 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(P_.IsAtomic, [], [ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.DmaskArgIndex>>]),`。
- **L1214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(P_.IsSample, [ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.UnormArgIndex>>], []),`.
  **L1214 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(P_.IsSample, [ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.UnormArgIndex>>], []),`。
- **L1215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.TexFailCtrlArgIndex>>,`.
  **L1215 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.TexFailCtrlArgIndex>>,`。
- **L1216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.CachePolicyArgIndex>>],`.
  **L1216 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<AMDGPUImageDimIntrinsicEval<P_>.CachePolicyArgIndex>>],`。

### Lines 1217-1248

````tablegen
          !if(P_.IsAtomic, [], [IntrNoSync])),


      "", sdnodeprops>,
  AMDGPURsrcIntrinsic<!add(!size(P_.DataArgs), !size(P_.AddrTypes),
                           !if(P_.IsAtomic, 0, 1)), 1> {
  AMDGPUDimProfile P = P_;

  AMDGPUImageDimIntrinsic Intr = !cast<AMDGPUImageDimIntrinsic>(NAME);

  let TargetPrefix = "amdgcn";
}

// Marker class for intrinsics with a DMask that determines the returned
// channels.
class AMDGPUImageDMaskIntrinsic;

defset list<AMDGPUImageDimIntrinsic> AMDGPUImageDimIntrinsics = {

  //////////////////////////////////////////////////////////////////////////
  // Load and store intrinsics
  //////////////////////////////////////////////////////////////////////////
  multiclass AMDGPUImageDimIntrinsicsNoMsaa<string opmod,
                                            list<LLVMType> retty,
                                            list<AMDGPUArg> dataargs,
                                            list<IntrinsicProperty> props,
                                            list<SDNodeProperty> sdnodeprops,
                                            bit Mip = false> {
    foreach dim = AMDGPUDims.NoMsaa in {
      def !strconcat(NAME, "_", dim.Name)
        : AMDGPUImageDimIntrinsic<
            AMDGPUDimNoSampleProfile<opmod, dim, retty, dataargs, Mip>,
````
- **L1217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(P_.IsAtomic, [], [IntrNoSync])),`.
  **L1217 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(P_.IsAtomic, [], [IntrNoSync])),`。
- **L1218 EN**: Blank line separating nearby declarations or logic blocks.
  **L1218 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1219 EN**: Blank line separating nearby declarations or logic blocks.
  **L1219 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"", sdnodeprops>,`.
  **L1220 CN**: 继续一个多行参数列表、初始化器或聚合项：`"", sdnodeprops>,`。
- **L1221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPURsrcIntrinsic<!add(!size(P_.DataArgs), !size(P_.AddrTypes),`.
  **L1221 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPURsrcIntrinsic<!add(!size(P_.DataArgs), !size(P_.AddrTypes),`。
- **L1222 EN**: Starts a function, method, lambda, or structured scope: `!if(P_.IsAtomic, 0, 1)), 1> {`.
  **L1222 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!if(P_.IsAtomic, 0, 1)), 1> {`。
- **L1223 EN**: Initializes variable `P` from the right-hand expression.
  **L1223 CN**: 使用右侧表达式初始化变量 `P`。
- **L1224 EN**: Blank line separating nearby declarations or logic blocks.
  **L1224 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1225 EN**: Initializes variable `Intr` from the right-hand expression.
  **L1225 CN**: 使用右侧表达式初始化变量 `Intr`。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1227 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1227 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1228 EN**: Closes the current lexical scope or compound statement.
  **L1228 CN**: 结束当前词法作用域或复合语句块。
- **L1229 EN**: Blank line separating nearby declarations or logic blocks.
  **L1229 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1230 EN**: Comment explains nearby logic, invariants, or intent: `Marker class for intrinsics with a DMask that determines the returned`.
  **L1230 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Marker class for intrinsics with a DMask that determines the returned`。
- **L1231 EN**: Comment explains nearby logic, invariants, or intent: `channels.`.
  **L1231 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`channels.`。
- **L1232 EN**: Declares class `AMDGPUImageDMaskIntrinsic`.
  **L1232 CN**: 声明 class `AMDGPUImageDMaskIntrinsic`。
- **L1233 EN**: Blank line separating nearby declarations or logic blocks.
  **L1233 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1234 EN**: Continues the surrounding expression or declaration: `defset list<AMDGPUImageDimIntrinsic> AMDGPUImageDimIntrinsics = {`.
  **L1234 CN**: 继续构造周围的表达式或声明：`defset list<AMDGPUImageDimIntrinsic> AMDGPUImageDimIntrinsics = {`。
- **L1235 EN**: Blank line separating nearby declarations or logic blocks.
  **L1235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1236 EN**: Separator comment used for visual grouping.
  **L1236 CN**: 用于视觉分组的分隔注释。
- **L1237 EN**: Comment explains nearby logic, invariants, or intent: `Load and store intrinsics`.
  **L1237 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Load and store intrinsics`。
- **L1238 EN**: Separator comment used for visual grouping.
  **L1238 CN**: 用于视觉分组的分隔注释。
- **L1239 EN**: Declares TableGen multiclass `AMDGPUImageDimIntrinsicsNoMsaa`.
  **L1239 CN**: 声明 TableGen multiclass `AMDGPUImageDimIntrinsicsNoMsaa`。
- **L1240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> retty,`.
  **L1240 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> retty,`。
- **L1241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<AMDGPUArg> dataargs,`.
  **L1241 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<AMDGPUArg> dataargs,`。
- **L1242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<IntrinsicProperty> props,`.
  **L1242 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<IntrinsicProperty> props,`。
- **L1243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<SDNodeProperty> sdnodeprops,`.
  **L1243 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<SDNodeProperty> sdnodeprops,`。
- **L1244 EN**: Continues the surrounding expression or declaration: `bit Mip = false> {`.
  **L1244 CN**: 继续构造周围的表达式或声明：`bit Mip = false> {`。
- **L1245 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1245 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1246 EN**: Declares TableGen def `!strconcat(NAME,`.
  **L1246 CN**: 声明 TableGen def `!strconcat(NAME,`。
- **L1247 EN**: Continues the surrounding expression or declaration: `: AMDGPUImageDimIntrinsic<`.
  **L1247 CN**: 继续构造周围的表达式或声明：`: AMDGPUImageDimIntrinsic<`。
- **L1248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimNoSampleProfile<opmod, dim, retty, dataargs, Mip>,`.
  **L1248 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimNoSampleProfile<opmod, dim, retty, dataargs, Mip>,`。

### Lines 1249-1280

````tablegen
            props, sdnodeprops>;
    }
  }

  multiclass AMDGPUImageDimIntrinsicsAll<string opmod,
                                         list<LLVMType> retty,
                                         list<AMDGPUArg> dataargs,
                                         list<IntrinsicProperty> props,
                                         list<SDNodeProperty> sdnodeprops,
                                         bit Mip = false> {
    foreach dim = AMDGPUDims.All in {
      def !strconcat(NAME, "_", dim.Name)
        : AMDGPUImageDimIntrinsic<
            AMDGPUDimNoSampleProfile<opmod, dim, retty, dataargs, Mip>,
            props, sdnodeprops>;
    }
  }

  defm int_amdgcn_image_load
    : AMDGPUImageDimIntrinsicsAll<"LOAD", [llvm_any_ty], [], [IntrReadMem],
                                  [SDNPMemOperand]>,
      AMDGPUImageDMaskIntrinsic;
  defm int_amdgcn_image_load_mip
    : AMDGPUImageDimIntrinsicsNoMsaa<"LOAD_MIP", [llvm_any_ty], [],
                                     [IntrReadMem, IntrWillReturn], [SDNPMemOperand], 1>,
      AMDGPUImageDMaskIntrinsic;

  defm int_amdgcn_image_store : AMDGPUImageDimIntrinsicsAll<
              "STORE", [], [AMDGPUArg<llvm_anyfloat_ty, "vdata">],
              [IntrWriteMem, IntrWillReturn], [SDNPMemOperand]>,
              AMDGPUImageDMaskIntrinsic;
  defm int_amdgcn_image_store_mip : AMDGPUImageDimIntrinsicsNoMsaa<
````
- **L1249 EN**: Executes a standalone statement or declaration: `props, sdnodeprops>;`.
  **L1249 CN**: 执行一条独立语句或声明：`props, sdnodeprops>;`。
- **L1250 EN**: Closes the current lexical scope or compound statement.
  **L1250 CN**: 结束当前词法作用域或复合语句块。
- **L1251 EN**: Closes the current lexical scope or compound statement.
  **L1251 CN**: 结束当前词法作用域或复合语句块。
- **L1252 EN**: Blank line separating nearby declarations or logic blocks.
  **L1252 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1253 EN**: Declares TableGen multiclass `AMDGPUImageDimIntrinsicsAll`.
  **L1253 CN**: 声明 TableGen multiclass `AMDGPUImageDimIntrinsicsAll`。
- **L1254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> retty,`.
  **L1254 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> retty,`。
- **L1255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<AMDGPUArg> dataargs,`.
  **L1255 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<AMDGPUArg> dataargs,`。
- **L1256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<IntrinsicProperty> props,`.
  **L1256 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<IntrinsicProperty> props,`。
- **L1257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<SDNodeProperty> sdnodeprops,`.
  **L1257 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<SDNodeProperty> sdnodeprops,`。
- **L1258 EN**: Continues the surrounding expression or declaration: `bit Mip = false> {`.
  **L1258 CN**: 继续构造周围的表达式或声明：`bit Mip = false> {`。
- **L1259 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1259 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1260 EN**: Declares TableGen def `!strconcat(NAME,`.
  **L1260 CN**: 声明 TableGen def `!strconcat(NAME,`。
- **L1261 EN**: Continues the surrounding expression or declaration: `: AMDGPUImageDimIntrinsic<`.
  **L1261 CN**: 继续构造周围的表达式或声明：`: AMDGPUImageDimIntrinsic<`。
- **L1262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimNoSampleProfile<opmod, dim, retty, dataargs, Mip>,`.
  **L1262 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimNoSampleProfile<opmod, dim, retty, dataargs, Mip>,`。
- **L1263 EN**: Executes a standalone statement or declaration: `props, sdnodeprops>;`.
  **L1263 CN**: 执行一条独立语句或声明：`props, sdnodeprops>;`。
- **L1264 EN**: Closes the current lexical scope or compound statement.
  **L1264 CN**: 结束当前词法作用域或复合语句块。
- **L1265 EN**: Closes the current lexical scope or compound statement.
  **L1265 CN**: 结束当前词法作用域或复合语句块。
- **L1266 EN**: Blank line separating nearby declarations or logic blocks.
  **L1266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1267 EN**: Declares TableGen defm `int_amdgcn_image_load`.
  **L1267 CN**: 声明 TableGen defm `int_amdgcn_image_load`。
- **L1268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: AMDGPUImageDimIntrinsicsAll<"LOAD", [llvm_any_ty], [], [IntrReadMem],`.
  **L1268 CN**: 继续一个多行参数列表、初始化器或聚合项：`: AMDGPUImageDimIntrinsicsAll<"LOAD", [llvm_any_ty], [], [IntrReadMem],`。
- **L1269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[SDNPMemOperand]>,`.
  **L1269 CN**: 继续一个多行参数列表、初始化器或聚合项：`[SDNPMemOperand]>,`。
- **L1270 EN**: Executes a standalone statement or declaration: `AMDGPUImageDMaskIntrinsic;`.
  **L1270 CN**: 执行一条独立语句或声明：`AMDGPUImageDMaskIntrinsic;`。
- **L1271 EN**: Declares TableGen defm `int_amdgcn_image_load_mip`.
  **L1271 CN**: 声明 TableGen defm `int_amdgcn_image_load_mip`。
- **L1272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: AMDGPUImageDimIntrinsicsNoMsaa<"LOAD_MIP", [llvm_any_ty], [],`.
  **L1272 CN**: 继续一个多行参数列表、初始化器或聚合项：`: AMDGPUImageDimIntrinsicsNoMsaa<"LOAD_MIP", [llvm_any_ty], [],`。
- **L1273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem, IntrWillReturn], [SDNPMemOperand], 1>,`.
  **L1273 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem, IntrWillReturn], [SDNPMemOperand], 1>,`。
- **L1274 EN**: Executes a standalone statement or declaration: `AMDGPUImageDMaskIntrinsic;`.
  **L1274 CN**: 执行一条独立语句或声明：`AMDGPUImageDMaskIntrinsic;`。
- **L1275 EN**: Blank line separating nearby declarations or logic blocks.
  **L1275 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1276 EN**: Declares TableGen defm `int_amdgcn_image_store`.
  **L1276 CN**: 声明 TableGen defm `int_amdgcn_image_store`。
- **L1277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"STORE", [], [AMDGPUArg<llvm_anyfloat_ty, "vdata">],`.
  **L1277 CN**: 继续一个多行参数列表、初始化器或聚合项：`"STORE", [], [AMDGPUArg<llvm_anyfloat_ty, "vdata">],`。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem, IntrWillReturn], [SDNPMemOperand]>,`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem, IntrWillReturn], [SDNPMemOperand]>,`。
- **L1279 EN**: Executes a standalone statement or declaration: `AMDGPUImageDMaskIntrinsic;`.
  **L1279 CN**: 执行一条独立语句或声明：`AMDGPUImageDMaskIntrinsic;`。
- **L1280 EN**: Declares TableGen defm `int_amdgcn_image_store_mip`.
  **L1280 CN**: 声明 TableGen defm `int_amdgcn_image_store_mip`。

### Lines 1281-1312

````tablegen
              "STORE_MIP", [], [AMDGPUArg<llvm_anyfloat_ty, "vdata">],
              [IntrWriteMem, IntrWillReturn], [SDNPMemOperand], 1>,
              AMDGPUImageDMaskIntrinsic;

  //////////////////////////////////////////////////////////////////////////
  // MSAA intrinsics
  //////////////////////////////////////////////////////////////////////////
  foreach dim = AMDGPUDims.Msaa in {
    def int_amdgcn_image_msaa_load_x # _ # dim.Name:
        AMDGPUImageDimIntrinsic<
            AMDGPUDimNoSampleProfile<"MSAA_LOAD_X", dim, [llvm_any_ty], []>,
            [IntrReadMem], [SDNPMemOperand]>;
  }

  foreach dim = AMDGPUDims.Msaa in {
    def int_amdgcn_image_msaa_load # _ # dim.Name:
        AMDGPUImageDimIntrinsic<
            AMDGPUDimNoSampleProfile<"MSAA_LOAD", dim, [llvm_any_ty], []>,
            [IntrReadMem], [SDNPMemOperand]>;
  }

  //////////////////////////////////////////////////////////////////////////
  // sample and getlod intrinsics
  //////////////////////////////////////////////////////////////////////////
  multiclass AMDGPUImageDimSampleDims<string opmod,
                                      AMDGPUSampleVariant sample,
                                      bit NoMem = false> {
    foreach dim = AMDGPUDims.NoMsaa in {
      def !strconcat(NAME, "_", dim.Name) : AMDGPUImageDimIntrinsic<
          AMDGPUDimSampleProfile<opmod, dim, sample>,
          !listconcat(!if(NoMem, [IntrNoMem], [IntrReadMem]),
                      !if(sample.UsesWQM, [IntrConvergent], [])),
````
- **L1281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"STORE_MIP", [], [AMDGPUArg<llvm_anyfloat_ty, "vdata">],`.
  **L1281 CN**: 继续一个多行参数列表、初始化器或聚合项：`"STORE_MIP", [], [AMDGPUArg<llvm_anyfloat_ty, "vdata">],`。
- **L1282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem, IntrWillReturn], [SDNPMemOperand], 1>,`.
  **L1282 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem, IntrWillReturn], [SDNPMemOperand], 1>,`。
- **L1283 EN**: Executes a standalone statement or declaration: `AMDGPUImageDMaskIntrinsic;`.
  **L1283 CN**: 执行一条独立语句或声明：`AMDGPUImageDMaskIntrinsic;`。
- **L1284 EN**: Blank line separating nearby declarations or logic blocks.
  **L1284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1285 EN**: Separator comment used for visual grouping.
  **L1285 CN**: 用于视觉分组的分隔注释。
- **L1286 EN**: Comment explains nearby logic, invariants, or intent: `MSAA intrinsics`.
  **L1286 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MSAA intrinsics`。
- **L1287 EN**: Separator comment used for visual grouping.
  **L1287 CN**: 用于视觉分组的分隔注释。
- **L1288 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1288 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1289 EN**: Declares TableGen def `int_amdgcn_image_msaa_load_x`.
  **L1289 CN**: 声明 TableGen def `int_amdgcn_image_msaa_load_x`。
- **L1290 EN**: Continues the surrounding expression or declaration: `AMDGPUImageDimIntrinsic<`.
  **L1290 CN**: 继续构造周围的表达式或声明：`AMDGPUImageDimIntrinsic<`。
- **L1291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimNoSampleProfile<"MSAA_LOAD_X", dim, [llvm_any_ty], []>,`.
  **L1291 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimNoSampleProfile<"MSAA_LOAD_X", dim, [llvm_any_ty], []>,`。
- **L1292 EN**: Executes a standalone statement or declaration: `[IntrReadMem], [SDNPMemOperand]>;`.
  **L1292 CN**: 执行一条独立语句或声明：`[IntrReadMem], [SDNPMemOperand]>;`。
- **L1293 EN**: Closes the current lexical scope or compound statement.
  **L1293 CN**: 结束当前词法作用域或复合语句块。
- **L1294 EN**: Blank line separating nearby declarations or logic blocks.
  **L1294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1295 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1295 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1296 EN**: Declares TableGen def `int_amdgcn_image_msaa_load`.
  **L1296 CN**: 声明 TableGen def `int_amdgcn_image_msaa_load`。
- **L1297 EN**: Continues the surrounding expression or declaration: `AMDGPUImageDimIntrinsic<`.
  **L1297 CN**: 继续构造周围的表达式或声明：`AMDGPUImageDimIntrinsic<`。
- **L1298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimNoSampleProfile<"MSAA_LOAD", dim, [llvm_any_ty], []>,`.
  **L1298 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimNoSampleProfile<"MSAA_LOAD", dim, [llvm_any_ty], []>,`。
- **L1299 EN**: Executes a standalone statement or declaration: `[IntrReadMem], [SDNPMemOperand]>;`.
  **L1299 CN**: 执行一条独立语句或声明：`[IntrReadMem], [SDNPMemOperand]>;`。
- **L1300 EN**: Closes the current lexical scope or compound statement.
  **L1300 CN**: 结束当前词法作用域或复合语句块。
- **L1301 EN**: Blank line separating nearby declarations or logic blocks.
  **L1301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1302 EN**: Separator comment used for visual grouping.
  **L1302 CN**: 用于视觉分组的分隔注释。
- **L1303 EN**: Comment explains nearby logic, invariants, or intent: `sample and getlod intrinsics`.
  **L1303 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sample and getlod intrinsics`。
- **L1304 EN**: Separator comment used for visual grouping.
  **L1304 CN**: 用于视觉分组的分隔注释。
- **L1305 EN**: Declares TableGen multiclass `AMDGPUImageDimSampleDims`.
  **L1305 CN**: 声明 TableGen multiclass `AMDGPUImageDimSampleDims`。
- **L1306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUSampleVariant sample,`.
  **L1306 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUSampleVariant sample,`。
- **L1307 EN**: Continues the surrounding expression or declaration: `bit NoMem = false> {`.
  **L1307 CN**: 继续构造周围的表达式或声明：`bit NoMem = false> {`。
- **L1308 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1308 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1309 EN**: Declares TableGen def `!strconcat(NAME,`.
  **L1309 CN**: 声明 TableGen def `!strconcat(NAME,`。
- **L1310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimSampleProfile<opmod, dim, sample>,`.
  **L1310 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimSampleProfile<opmod, dim, sample>,`。
- **L1311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(!if(NoMem, [IntrNoMem], [IntrReadMem]),`.
  **L1311 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(!if(NoMem, [IntrNoMem], [IntrReadMem]),`。
- **L1312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(sample.UsesWQM, [IntrConvergent], [])),`.
  **L1312 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(sample.UsesWQM, [IntrConvergent], [])),`。

### Lines 1313-1344

````tablegen
          !if(NoMem, [], [SDNPMemOperand])>;
    }
  }

  foreach sample = AMDGPUSampleVariants in {
    defm int_amdgcn_image_sample # sample.LowerCaseMod
      : AMDGPUImageDimSampleDims<"SAMPLE" # sample.UpperCaseMod, sample>,
        AMDGPUImageDMaskIntrinsic;
  }

  multiclass AMDGPUImageDimSampleNoReturnDims<string opmod,
                                      AMDGPUSampleVariant sample> {
    foreach dim = AMDGPUDims.NoMsaa in {
      def !strconcat(NAME, "_", dim.Name, "_nortn") : AMDGPUImageDimIntrinsic<
          AMDGPUDimSampleNoReturnProfile<opmod, dim, sample>,
          !listconcat([IntrWillReturn], !if(sample.UsesWQM, [IntrConvergent], [])),
          [SDNPMemOperand]>;
    }
  }
  foreach sample = AMDGPUSampleVariants in {
    defm int_amdgcn_image_sample # sample.LowerCaseMod
      : AMDGPUImageDimSampleNoReturnDims<
        "SAMPLE" # sample.UpperCaseMod # "_nortn", sample>,
        AMDGPUImageDMaskIntrinsic;
  }

  defm int_amdgcn_image_getlod
    : AMDGPUImageDimSampleDims<"GET_LOD", AMDGPUSample, 1>,
      AMDGPUImageDMaskIntrinsic;

  //////////////////////////////////////////////////////////////////////////
  // getresinfo intrinsics
````
- **L1313 EN**: Executes a call or declaration centered on `!if`.
  **L1313 CN**: 执行以 `!if` 为核心的调用或声明。
- **L1314 EN**: Closes the current lexical scope or compound statement.
  **L1314 CN**: 结束当前词法作用域或复合语句块。
- **L1315 EN**: Closes the current lexical scope or compound statement.
  **L1315 CN**: 结束当前词法作用域或复合语句块。
- **L1316 EN**: Blank line separating nearby declarations or logic blocks.
  **L1316 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1317 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1317 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1318 EN**: Declares TableGen defm `int_amdgcn_image_sample`.
  **L1318 CN**: 声明 TableGen defm `int_amdgcn_image_sample`。
- **L1319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: AMDGPUImageDimSampleDims<"SAMPLE" # sample.UpperCaseMod, sample>,`.
  **L1319 CN**: 继续一个多行参数列表、初始化器或聚合项：`: AMDGPUImageDimSampleDims<"SAMPLE" # sample.UpperCaseMod, sample>,`。
- **L1320 EN**: Executes a standalone statement or declaration: `AMDGPUImageDMaskIntrinsic;`.
  **L1320 CN**: 执行一条独立语句或声明：`AMDGPUImageDMaskIntrinsic;`。
- **L1321 EN**: Closes the current lexical scope or compound statement.
  **L1321 CN**: 结束当前词法作用域或复合语句块。
- **L1322 EN**: Blank line separating nearby declarations or logic blocks.
  **L1322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1323 EN**: Declares TableGen multiclass `AMDGPUImageDimSampleNoReturnDims`.
  **L1323 CN**: 声明 TableGen multiclass `AMDGPUImageDimSampleNoReturnDims`。
- **L1324 EN**: Continues the surrounding expression or declaration: `AMDGPUSampleVariant sample> {`.
  **L1324 CN**: 继续构造周围的表达式或声明：`AMDGPUSampleVariant sample> {`。
- **L1325 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1325 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1326 EN**: Declares TableGen def `!strconcat(NAME,`.
  **L1326 CN**: 声明 TableGen def `!strconcat(NAME,`。
- **L1327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimSampleNoReturnProfile<opmod, dim, sample>,`.
  **L1327 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimSampleNoReturnProfile<opmod, dim, sample>,`。
- **L1328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([IntrWillReturn], !if(sample.UsesWQM, [IntrConvergent], [])),`.
  **L1328 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([IntrWillReturn], !if(sample.UsesWQM, [IntrConvergent], [])),`。
- **L1329 EN**: Executes a standalone statement or declaration: `[SDNPMemOperand]>;`.
  **L1329 CN**: 执行一条独立语句或声明：`[SDNPMemOperand]>;`。
- **L1330 EN**: Closes the current lexical scope or compound statement.
  **L1330 CN**: 结束当前词法作用域或复合语句块。
- **L1331 EN**: Closes the current lexical scope or compound statement.
  **L1331 CN**: 结束当前词法作用域或复合语句块。
- **L1332 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1332 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1333 EN**: Declares TableGen defm `int_amdgcn_image_sample`.
  **L1333 CN**: 声明 TableGen defm `int_amdgcn_image_sample`。
- **L1334 EN**: Continues the surrounding expression or declaration: `: AMDGPUImageDimSampleNoReturnDims<`.
  **L1334 CN**: 继续构造周围的表达式或声明：`: AMDGPUImageDimSampleNoReturnDims<`。
- **L1335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"SAMPLE" # sample.UpperCaseMod # "_nortn", sample>,`.
  **L1335 CN**: 继续一个多行参数列表、初始化器或聚合项：`"SAMPLE" # sample.UpperCaseMod # "_nortn", sample>,`。
- **L1336 EN**: Executes a standalone statement or declaration: `AMDGPUImageDMaskIntrinsic;`.
  **L1336 CN**: 执行一条独立语句或声明：`AMDGPUImageDMaskIntrinsic;`。
- **L1337 EN**: Closes the current lexical scope or compound statement.
  **L1337 CN**: 结束当前词法作用域或复合语句块。
- **L1338 EN**: Blank line separating nearby declarations or logic blocks.
  **L1338 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1339 EN**: Declares TableGen defm `int_amdgcn_image_getlod`.
  **L1339 CN**: 声明 TableGen defm `int_amdgcn_image_getlod`。
- **L1340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: AMDGPUImageDimSampleDims<"GET_LOD", AMDGPUSample, 1>,`.
  **L1340 CN**: 继续一个多行参数列表、初始化器或聚合项：`: AMDGPUImageDimSampleDims<"GET_LOD", AMDGPUSample, 1>,`。
- **L1341 EN**: Executes a standalone statement or declaration: `AMDGPUImageDMaskIntrinsic;`.
  **L1341 CN**: 执行一条独立语句或声明：`AMDGPUImageDMaskIntrinsic;`。
- **L1342 EN**: Blank line separating nearby declarations or logic blocks.
  **L1342 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1343 EN**: Separator comment used for visual grouping.
  **L1343 CN**: 用于视觉分组的分隔注释。
- **L1344 EN**: Comment explains nearby logic, invariants, or intent: `getresinfo intrinsics`.
  **L1344 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`getresinfo intrinsics`。

### Lines 1345-1376

````tablegen
  //////////////////////////////////////////////////////////////////////////
  foreach dim = AMDGPUDims.All in {
    def !strconcat("int_amdgcn_image_getresinfo_", dim.Name)
      : AMDGPUImageDimIntrinsic<AMDGPUDimGetResInfoProfile<dim>, [IntrNoMem], []>,
        AMDGPUImageDMaskIntrinsic;
  }

  //////////////////////////////////////////////////////////////////////////
  // gather4 intrinsics
  //////////////////////////////////////////////////////////////////////////
  foreach sample = AMDGPUSampleVariantsNoGradients in {
    foreach dim = [AMDGPUDim2D, AMDGPUDimCube, AMDGPUDim2DArray] in {
      def int_amdgcn_image_gather4 # sample.LowerCaseMod # _ # dim.Name:
          AMDGPUImageDimIntrinsic<
              AMDGPUDimSampleProfile<"GATHER4" # sample.UpperCaseMod, dim, sample>,
              [IntrReadMem], [SDNPMemOperand]>;
    }
  }
}

//////////////////////////////////////////////////////////////////////////
// atomic intrinsics
//////////////////////////////////////////////////////////////////////////
defset list<AMDGPUImageDimIntrinsic> AMDGPUImageDimAtomicIntrinsics = {
  multiclass AMDGPUImageDimAtomicX<string opmod, list<AMDGPUArg> dataargs,
                                   LLVMType rettype = llvm_anyint_ty> {
        foreach dim = AMDGPUDims.All in {
          def !strconcat(NAME, "_", dim.Name):
            AMDGPUImageDimIntrinsic<AMDGPUDimAtomicProfile<opmod, dim, dataargs, rettype>,
            [], [SDNPMemOperand]>;
        }
  }
````
- **L1345 EN**: Separator comment used for visual grouping.
  **L1345 CN**: 用于视觉分组的分隔注释。
- **L1346 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1346 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1347 EN**: Declares TableGen def `!strconcat("int_amdgcn_image_getresinfo_",`.
  **L1347 CN**: 声明 TableGen def `!strconcat("int_amdgcn_image_getresinfo_",`。
- **L1348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: AMDGPUImageDimIntrinsic<AMDGPUDimGetResInfoProfile<dim>, [IntrNoMem], []>,`.
  **L1348 CN**: 继续一个多行参数列表、初始化器或聚合项：`: AMDGPUImageDimIntrinsic<AMDGPUDimGetResInfoProfile<dim>, [IntrNoMem], []>,`。
- **L1349 EN**: Executes a standalone statement or declaration: `AMDGPUImageDMaskIntrinsic;`.
  **L1349 CN**: 执行一条独立语句或声明：`AMDGPUImageDMaskIntrinsic;`。
- **L1350 EN**: Closes the current lexical scope or compound statement.
  **L1350 CN**: 结束当前词法作用域或复合语句块。
- **L1351 EN**: Blank line separating nearby declarations or logic blocks.
  **L1351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1352 EN**: Separator comment used for visual grouping.
  **L1352 CN**: 用于视觉分组的分隔注释。
- **L1353 EN**: Comment explains nearby logic, invariants, or intent: `gather4 intrinsics`.
  **L1353 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gather4 intrinsics`。
- **L1354 EN**: Separator comment used for visual grouping.
  **L1354 CN**: 用于视觉分组的分隔注释。
- **L1355 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1355 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1356 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1356 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1357 EN**: Declares TableGen def `int_amdgcn_image_gather4`.
  **L1357 CN**: 声明 TableGen def `int_amdgcn_image_gather4`。
- **L1358 EN**: Continues the surrounding expression or declaration: `AMDGPUImageDimIntrinsic<`.
  **L1358 CN**: 继续构造周围的表达式或声明：`AMDGPUImageDimIntrinsic<`。
- **L1359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUDimSampleProfile<"GATHER4" # sample.UpperCaseMod, dim, sample>,`.
  **L1359 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUDimSampleProfile<"GATHER4" # sample.UpperCaseMod, dim, sample>,`。
- **L1360 EN**: Executes a standalone statement or declaration: `[IntrReadMem], [SDNPMemOperand]>;`.
  **L1360 CN**: 执行一条独立语句或声明：`[IntrReadMem], [SDNPMemOperand]>;`。
- **L1361 EN**: Closes the current lexical scope or compound statement.
  **L1361 CN**: 结束当前词法作用域或复合语句块。
- **L1362 EN**: Closes the current lexical scope or compound statement.
  **L1362 CN**: 结束当前词法作用域或复合语句块。
- **L1363 EN**: Closes the current lexical scope or compound statement.
  **L1363 CN**: 结束当前词法作用域或复合语句块。
- **L1364 EN**: Blank line separating nearby declarations or logic blocks.
  **L1364 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1365 EN**: Separator comment used for visual grouping.
  **L1365 CN**: 用于视觉分组的分隔注释。
- **L1366 EN**: Comment explains nearby logic, invariants, or intent: `atomic intrinsics`.
  **L1366 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`atomic intrinsics`。
- **L1367 EN**: Separator comment used for visual grouping.
  **L1367 CN**: 用于视觉分组的分隔注释。
- **L1368 EN**: Continues the surrounding expression or declaration: `defset list<AMDGPUImageDimIntrinsic> AMDGPUImageDimAtomicIntrinsics = {`.
  **L1368 CN**: 继续构造周围的表达式或声明：`defset list<AMDGPUImageDimIntrinsic> AMDGPUImageDimAtomicIntrinsics = {`。
- **L1369 EN**: Declares TableGen multiclass `AMDGPUImageDimAtomicX`.
  **L1369 CN**: 声明 TableGen multiclass `AMDGPUImageDimAtomicX`。
- **L1370 EN**: Continues the surrounding expression or declaration: `LLVMType rettype = llvm_anyint_ty> {`.
  **L1370 CN**: 继续构造周围的表达式或声明：`LLVMType rettype = llvm_anyint_ty> {`。
- **L1371 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1371 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1372 EN**: Declares TableGen def `!strconcat(NAME,`.
  **L1372 CN**: 声明 TableGen def `!strconcat(NAME,`。
- **L1373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUImageDimIntrinsic<AMDGPUDimAtomicProfile<opmod, dim, dataargs, rettype>,`.
  **L1373 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUImageDimIntrinsic<AMDGPUDimAtomicProfile<opmod, dim, dataargs, rettype>,`。
- **L1374 EN**: Executes a standalone statement or declaration: `[], [SDNPMemOperand]>;`.
  **L1374 CN**: 执行一条独立语句或声明：`[], [SDNPMemOperand]>;`。
- **L1375 EN**: Closes the current lexical scope or compound statement.
  **L1375 CN**: 结束当前词法作用域或复合语句块。
- **L1376 EN**: Closes the current lexical scope or compound statement.
  **L1376 CN**: 结束当前词法作用域或复合语句块。

### Lines 1377-1408

````tablegen

  multiclass AMDGPUImageDimAtomic<string opmod, LLVMType rettype = llvm_anyint_ty> :
    AMDGPUImageDimAtomicX<opmod, [AMDGPUArg<LLVMMatchType<0>, "vdata">], rettype>;

  multiclass AMDGPUImageDimFloatAtomic<string opmod> :
    AMDGPUImageDimAtomic<opmod, llvm_anyfloat_ty>;

  multiclass AMDGPUImageDimAnyAtomic<string opmod> :
    AMDGPUImageDimAtomic<opmod, llvm_any_ty>;

  defm int_amdgcn_image_atomic_swap : AMDGPUImageDimAnyAtomic<"ATOMIC_SWAP">;
  defm int_amdgcn_image_atomic_add : AMDGPUImageDimAtomic<"ATOMIC_ADD">;
  defm int_amdgcn_image_atomic_sub : AMDGPUImageDimAtomic<"ATOMIC_SUB">;
  defm int_amdgcn_image_atomic_smin : AMDGPUImageDimAtomic<"ATOMIC_SMIN">;
  defm int_amdgcn_image_atomic_umin : AMDGPUImageDimAtomic<"ATOMIC_UMIN">;
  defm int_amdgcn_image_atomic_fmin : AMDGPUImageDimFloatAtomic<"ATOMIC_FMIN">;
  defm int_amdgcn_image_atomic_smax : AMDGPUImageDimAtomic<"ATOMIC_SMAX">;
  defm int_amdgcn_image_atomic_umax : AMDGPUImageDimAtomic<"ATOMIC_UMAX">;
  defm int_amdgcn_image_atomic_fmax : AMDGPUImageDimFloatAtomic<"ATOMIC_FMAX">;
  defm int_amdgcn_image_atomic_and : AMDGPUImageDimAtomic<"ATOMIC_AND">;
  defm int_amdgcn_image_atomic_or : AMDGPUImageDimAtomic<"ATOMIC_OR">;
  defm int_amdgcn_image_atomic_xor : AMDGPUImageDimAtomic<"ATOMIC_XOR">;
  defm int_amdgcn_image_atomic_inc : AMDGPUImageDimAtomic<"ATOMIC_INC">;
  defm int_amdgcn_image_atomic_dec : AMDGPUImageDimAtomic<"ATOMIC_DEC">;
  defm int_amdgcn_image_atomic_add_flt : AMDGPUImageDimFloatAtomic<"ATOMIC_ADD_FLT">;
  defm int_amdgcn_image_atomic_min_flt : AMDGPUImageDimFloatAtomic<"ATOMIC_MIN_FLT">;
  defm int_amdgcn_image_atomic_max_flt : AMDGPUImageDimFloatAtomic<"ATOMIC_MAX_FLT">;

  defm int_amdgcn_image_atomic_cmpswap :
      AMDGPUImageDimAtomicX<"ATOMIC_CMPSWAP", [AMDGPUArg<LLVMMatchType<0>, "src">,
                                               AMDGPUArg<LLVMMatchType<0>, "cmp">]>;

````
- **L1377 EN**: Blank line separating nearby declarations or logic blocks.
  **L1377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1378 EN**: Declares TableGen multiclass `AMDGPUImageDimAtomic`.
  **L1378 CN**: 声明 TableGen multiclass `AMDGPUImageDimAtomic`。
- **L1379 EN**: Executes a standalone statement or declaration: `AMDGPUImageDimAtomicX<opmod, [AMDGPUArg<LLVMMatchType<0>, "vdata">], rettype>;`.
  **L1379 CN**: 执行一条独立语句或声明：`AMDGPUImageDimAtomicX<opmod, [AMDGPUArg<LLVMMatchType<0>, "vdata">], rettype>;`。
- **L1380 EN**: Blank line separating nearby declarations or logic blocks.
  **L1380 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1381 EN**: Declares TableGen multiclass `AMDGPUImageDimFloatAtomic`.
  **L1381 CN**: 声明 TableGen multiclass `AMDGPUImageDimFloatAtomic`。
- **L1382 EN**: Executes a standalone statement or declaration: `AMDGPUImageDimAtomic<opmod, llvm_anyfloat_ty>;`.
  **L1382 CN**: 执行一条独立语句或声明：`AMDGPUImageDimAtomic<opmod, llvm_anyfloat_ty>;`。
- **L1383 EN**: Blank line separating nearby declarations or logic blocks.
  **L1383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1384 EN**: Declares TableGen multiclass `AMDGPUImageDimAnyAtomic`.
  **L1384 CN**: 声明 TableGen multiclass `AMDGPUImageDimAnyAtomic`。
- **L1385 EN**: Executes a standalone statement or declaration: `AMDGPUImageDimAtomic<opmod, llvm_any_ty>;`.
  **L1385 CN**: 执行一条独立语句或声明：`AMDGPUImageDimAtomic<opmod, llvm_any_ty>;`。
- **L1386 EN**: Blank line separating nearby declarations or logic blocks.
  **L1386 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1387 EN**: Declares TableGen defm `int_amdgcn_image_atomic_swap`.
  **L1387 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_swap`。
- **L1388 EN**: Declares TableGen defm `int_amdgcn_image_atomic_add`.
  **L1388 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_add`。
- **L1389 EN**: Declares TableGen defm `int_amdgcn_image_atomic_sub`.
  **L1389 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_sub`。
- **L1390 EN**: Declares TableGen defm `int_amdgcn_image_atomic_smin`.
  **L1390 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_smin`。
- **L1391 EN**: Declares TableGen defm `int_amdgcn_image_atomic_umin`.
  **L1391 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_umin`。
- **L1392 EN**: Declares TableGen defm `int_amdgcn_image_atomic_fmin`.
  **L1392 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_fmin`。
- **L1393 EN**: Declares TableGen defm `int_amdgcn_image_atomic_smax`.
  **L1393 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_smax`。
- **L1394 EN**: Declares TableGen defm `int_amdgcn_image_atomic_umax`.
  **L1394 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_umax`。
- **L1395 EN**: Declares TableGen defm `int_amdgcn_image_atomic_fmax`.
  **L1395 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_fmax`。
- **L1396 EN**: Declares TableGen defm `int_amdgcn_image_atomic_and`.
  **L1396 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_and`。
- **L1397 EN**: Declares TableGen defm `int_amdgcn_image_atomic_or`.
  **L1397 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_or`。
- **L1398 EN**: Declares TableGen defm `int_amdgcn_image_atomic_xor`.
  **L1398 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_xor`。
- **L1399 EN**: Declares TableGen defm `int_amdgcn_image_atomic_inc`.
  **L1399 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_inc`。
- **L1400 EN**: Declares TableGen defm `int_amdgcn_image_atomic_dec`.
  **L1400 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_dec`。
- **L1401 EN**: Declares TableGen defm `int_amdgcn_image_atomic_add_flt`.
  **L1401 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_add_flt`。
- **L1402 EN**: Declares TableGen defm `int_amdgcn_image_atomic_min_flt`.
  **L1402 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_min_flt`。
- **L1403 EN**: Declares TableGen defm `int_amdgcn_image_atomic_max_flt`.
  **L1403 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_max_flt`。
- **L1404 EN**: Blank line separating nearby declarations or logic blocks.
  **L1404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1405 EN**: Declares TableGen defm `int_amdgcn_image_atomic_cmpswap`.
  **L1405 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_cmpswap`。
- **L1406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPUImageDimAtomicX<"ATOMIC_CMPSWAP", [AMDGPUArg<LLVMMatchType<0>, "src">,`.
  **L1406 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPUImageDimAtomicX<"ATOMIC_CMPSWAP", [AMDGPUArg<LLVMMatchType<0>, "src">,`。
- **L1407 EN**: Executes a standalone statement or declaration: `AMDGPUArg<LLVMMatchType<0>, "cmp">]>;`.
  **L1407 CN**: 执行一条独立语句或声明：`AMDGPUArg<LLVMMatchType<0>, "cmp">]>;`。
- **L1408 EN**: Blank line separating nearby declarations or logic blocks.
  **L1408 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1409-1440

````tablegen
  defm int_amdgcn_image_atomic_pk_add_f16 : AMDGPUImageDimFloatAtomic<"ATOMIC_PK_ADD_F16">;
  defm int_amdgcn_image_atomic_pk_add_bf16 : AMDGPUImageDimFloatAtomic<"ATOMIC_PK_ADD_BF16">;
}

//////////////////////////////////////////////////////////////////////////
// Buffer intrinsics
//////////////////////////////////////////////////////////////////////////

// Data type for buffer resources (V#). Maybe, in the future, we can create a
// similar one for textures (T#).
def AMDGPUBufferRsrcTy : LLVMQualPointerType<8>;
// Data type for buffer fat pointers, which are a buffer resource (V#) followed by
// a 32-bit offset. These don't exist in hardware and are a compiler-internal
// convenience.
def AMDGPUBufferFatPointerTy : LLVMQualPointerType<7>;

let TargetPrefix = "amdgcn" in {

// Create a buffer resource wrapping `base` with the specified `stride`
// `numrecords`, and `flags`. All of these values will need to be
// wave-uniform when the buffer instructions are invoked, so non-uniform
// inputs to this intrinsic will trigger waterfall loops.
//
// In addition to creating ptr addrspace(8), whe representation of buffer
// resources, it can create the fat pointers ptr addrspace(7) and ptr addrspace(9),
// which carry additional offset bits. When this intrinsic is used to create
// these fat pointers, their offset and index fields (if applicable) are zero.
def int_amdgcn_make_buffer_rsrc : PureIntrinsic <
  [llvm_anyptr_ty],
  [llvm_anyptr_ty, // base
   llvm_i16_ty,    // stride (and swizzle control)
   llvm_i64_ty,    // NumRecords / extent
````
- **L1409 EN**: Declares TableGen defm `int_amdgcn_image_atomic_pk_add_f16`.
  **L1409 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_pk_add_f16`。
- **L1410 EN**: Declares TableGen defm `int_amdgcn_image_atomic_pk_add_bf16`.
  **L1410 CN**: 声明 TableGen defm `int_amdgcn_image_atomic_pk_add_bf16`。
- **L1411 EN**: Closes the current lexical scope or compound statement.
  **L1411 CN**: 结束当前词法作用域或复合语句块。
- **L1412 EN**: Blank line separating nearby declarations or logic blocks.
  **L1412 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1413 EN**: Separator comment used for visual grouping.
  **L1413 CN**: 用于视觉分组的分隔注释。
- **L1414 EN**: Comment explains nearby logic, invariants, or intent: `Buffer intrinsics`.
  **L1414 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Buffer intrinsics`。
- **L1415 EN**: Separator comment used for visual grouping.
  **L1415 CN**: 用于视觉分组的分隔注释。
- **L1416 EN**: Blank line separating nearby declarations or logic blocks.
  **L1416 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1417 EN**: Comment explains nearby logic, invariants, or intent: `Data type for buffer resources (V#). Maybe, in the future, we can create a`.
  **L1417 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Data type for buffer resources (V#). Maybe, in the future, we can create a`。
- **L1418 EN**: Comment explains nearby logic, invariants, or intent: `similar one for textures (T#).`.
  **L1418 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`similar one for textures (T#).`。
- **L1419 EN**: Declares TableGen def `AMDGPUBufferRsrcTy`.
  **L1419 CN**: 声明 TableGen def `AMDGPUBufferRsrcTy`。
- **L1420 EN**: Comment explains nearby logic, invariants, or intent: `Data type for buffer fat pointers, which are a buffer resource (V#) followed by`.
  **L1420 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Data type for buffer fat pointers, which are a buffer resource (V#) followed by`。
- **L1421 EN**: Comment explains nearby logic, invariants, or intent: `a 32-bit offset. These don't exist in hardware and are a compiler-internal`.
  **L1421 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a 32-bit offset. These don't exist in hardware and are a compiler-internal`。
- **L1422 EN**: Comment explains nearby logic, invariants, or intent: `convenience.`.
  **L1422 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`convenience.`。
- **L1423 EN**: Declares TableGen def `AMDGPUBufferFatPointerTy`.
  **L1423 CN**: 声明 TableGen def `AMDGPUBufferFatPointerTy`。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1425 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1425 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1426 EN**: Blank line separating nearby declarations or logic blocks.
  **L1426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1427 EN**: Comment explains nearby logic, invariants, or intent: `Create a buffer resource wrapping `base` with the specified `stride``.
  **L1427 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create a buffer resource wrapping `base` with the specified `stride``。
- **L1428 EN**: Comment explains nearby logic, invariants, or intent: ``numrecords`, and `flags`. All of these values will need to be`.
  **L1428 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``numrecords`, and `flags`. All of these values will need to be`。
- **L1429 EN**: Comment explains nearby logic, invariants, or intent: `wave-uniform when the buffer instructions are invoked, so non-uniform`.
  **L1429 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wave-uniform when the buffer instructions are invoked, so non-uniform`。
- **L1430 EN**: Comment explains nearby logic, invariants, or intent: `inputs to this intrinsic will trigger waterfall loops.`.
  **L1430 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`inputs to this intrinsic will trigger waterfall loops.`。
- **L1431 EN**: Separator comment used for visual grouping.
  **L1431 CN**: 用于视觉分组的分隔注释。
- **L1432 EN**: Comment explains nearby logic, invariants, or intent: `In addition to creating ptr addrspace(8), whe representation of buffer`.
  **L1432 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In addition to creating ptr addrspace(8), whe representation of buffer`。
- **L1433 EN**: Comment explains nearby logic, invariants, or intent: `resources, it can create the fat pointers ptr addrspace(7) and ptr addrspace(9),`.
  **L1433 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`resources, it can create the fat pointers ptr addrspace(7) and ptr addrspace(9),`。
- **L1434 EN**: Comment explains nearby logic, invariants, or intent: `which carry additional offset bits. When this intrinsic is used to create`.
  **L1434 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`which carry additional offset bits. When this intrinsic is used to create`。
- **L1435 EN**: Comment explains nearby logic, invariants, or intent: `these fat pointers, their offset and index fields (if applicable) are zero.`.
  **L1435 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`these fat pointers, their offset and index fields (if applicable) are zero.`。
- **L1436 EN**: Declares TableGen def `int_amdgcn_make_buffer_rsrc`.
  **L1436 CN**: 声明 TableGen def `int_amdgcn_make_buffer_rsrc`。
- **L1437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty],`.
  **L1437 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty],`。
- **L1438 EN**: Continues the surrounding expression or declaration: `[llvm_anyptr_ty, // base`.
  **L1438 CN**: 继续构造周围的表达式或声明：`[llvm_anyptr_ty, // base`。
- **L1439 EN**: Continues logic associated with callable symbol `stride`.
  **L1439 CN**: 继续与可调用符号 `stride` 相关的逻辑。
- **L1440 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty,    // NumRecords / extent`.
  **L1440 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty,    // NumRecords / extent`。

### Lines 1441-1472

````tablegen
   llvm_i32_ty],   // flags
  // Attributes lifted from ptrmask + some extra argument attributes.
  [ReadNone<ArgIndex<0>>]>;

defset list<AMDGPURsrcIntrinsic> AMDGPUBufferIntrinsics = {

// Generate a buffer_load instruction that may be optimized to s_buffer_load if
// the offset argument is uniform.
def int_amdgcn_s_buffer_load : DefaultAttrsIntrinsic <
  [llvm_any_ty],
  [llvm_v4i32_ty,    // rsrc(SGPR)
   llvm_i32_ty,      // byte offset
   llvm_i32_ty],     // auxiliary/cachepolicy(imm):
                     //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                     //                bit 3 = swz, bit 4 = scc (gfx90a)
                     //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                     //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                     //                bit 6 = swz
                     // Note: volatile bit is **not** permitted here.
  [IntrNoMem, ImmArg<ArgIndex<2>>]>,
  AMDGPURsrcIntrinsic<0>;

// Buffer intrinsics with separate raw and struct variants.  The raw
// variant never has an index. The struct variant always has an index, even if
// it is const 0. A struct intrinsic with constant 0 index is different to the
// corresponding raw intrinsic on gfx9+ because the behavior of bound checking
// and swizzling changes depending on whether idxen is set in the instruction.
// These intrinsics also keep the offset and soffset arguments separate as
// they behave differently in bounds checking and swizzling.

// The versions of these intrinsics that take <4 x i32> arguments are deprecated
// in favor of their .ptr.buffer variants that take ptr addrspace(8) arguments,
````
- **L1441 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],   // flags`.
  **L1441 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],   // flags`。
- **L1442 EN**: Comment explains nearby logic, invariants, or intent: `Attributes lifted from ptrmask + some extra argument attributes.`.
  **L1442 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Attributes lifted from ptrmask + some extra argument attributes.`。
- **L1443 EN**: Executes a standalone statement or declaration: `[ReadNone<ArgIndex<0>>]>;`.
  **L1443 CN**: 执行一条独立语句或声明：`[ReadNone<ArgIndex<0>>]>;`。
- **L1444 EN**: Blank line separating nearby declarations or logic blocks.
  **L1444 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1445 EN**: Continues the surrounding expression or declaration: `defset list<AMDGPURsrcIntrinsic> AMDGPUBufferIntrinsics = {`.
  **L1445 CN**: 继续构造周围的表达式或声明：`defset list<AMDGPURsrcIntrinsic> AMDGPUBufferIntrinsics = {`。
- **L1446 EN**: Blank line separating nearby declarations or logic blocks.
  **L1446 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1447 EN**: Comment explains nearby logic, invariants, or intent: `Generate a buffer_load instruction that may be optimized to s_buffer_load if`.
  **L1447 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generate a buffer_load instruction that may be optimized to s_buffer_load if`。
- **L1448 EN**: Comment explains nearby logic, invariants, or intent: `the offset argument is uniform.`.
  **L1448 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the offset argument is uniform.`。
- **L1449 EN**: Declares TableGen def `int_amdgcn_s_buffer_load`.
  **L1449 CN**: 声明 TableGen def `int_amdgcn_s_buffer_load`。
- **L1450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty],`.
  **L1450 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty],`。
- **L1451 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1451 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1452 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // byte offset`.
  **L1452 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // byte offset`。
- **L1453 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1453 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1454 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1454 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1455 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1455 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1456 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1456 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1457 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1457 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1458 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1458 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1459 EN**: Comment explains nearby logic, invariants, or intent: `Note: volatile bit is **not** permitted here.`.
  **L1459 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: volatile bit is **not** permitted here.`。
- **L1460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, ImmArg<ArgIndex<2>>]>,`.
  **L1460 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, ImmArg<ArgIndex<2>>]>,`。
- **L1461 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1461 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1462 EN**: Blank line separating nearby declarations or logic blocks.
  **L1462 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1463 EN**: Comment explains nearby logic, invariants, or intent: `Buffer intrinsics with separate raw and struct variants.  The raw`.
  **L1463 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Buffer intrinsics with separate raw and struct variants.  The raw`。
- **L1464 EN**: Comment explains nearby logic, invariants, or intent: `variant never has an index. The struct variant always has an index, even if`.
  **L1464 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`variant never has an index. The struct variant always has an index, even if`。
- **L1465 EN**: Comment explains nearby logic, invariants, or intent: `it is const 0. A struct intrinsic with constant 0 index is different to the`.
  **L1465 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`it is const 0. A struct intrinsic with constant 0 index is different to the`。
- **L1466 EN**: Comment explains nearby logic, invariants, or intent: `corresponding raw intrinsic on gfx9+ because the behavior of bound checking`.
  **L1466 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`corresponding raw intrinsic on gfx9+ because the behavior of bound checking`。
- **L1467 EN**: Comment explains nearby logic, invariants, or intent: `and swizzling changes depending on whether idxen is set in the instruction.`.
  **L1467 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and swizzling changes depending on whether idxen is set in the instruction.`。
- **L1468 EN**: Comment explains nearby logic, invariants, or intent: `These intrinsics also keep the offset and soffset arguments separate as`.
  **L1468 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These intrinsics also keep the offset and soffset arguments separate as`。
- **L1469 EN**: Comment explains nearby logic, invariants, or intent: `they behave differently in bounds checking and swizzling.`.
  **L1469 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`they behave differently in bounds checking and swizzling.`。
- **L1470 EN**: Blank line separating nearby declarations or logic blocks.
  **L1470 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1471 EN**: Comment explains nearby logic, invariants, or intent: `The versions of these intrinsics that take <4 x i32> arguments are deprecated`.
  **L1471 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The versions of these intrinsics that take <4 x i32> arguments are deprecated`。
- **L1472 EN**: Comment explains nearby logic, invariants, or intent: `in favor of their .ptr.buffer variants that take ptr addrspace(8) arguments,`.
  **L1472 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`in favor of their .ptr.buffer variants that take ptr addrspace(8) arguments,`。

### Lines 1473-1504

````tablegen
// which allow for improved reasoning about memory accesses.
//
// Note that in the cachepolicy for all these intrinsics, bit 31 is not preserved
// through to final assembly selection and is used to signal that the buffer
// operation is volatile.
class AMDGPURawBufferLoad : DefaultAttrsIntrinsic <
  [llvm_any_ty],
  [llvm_v4i32_ty,    // rsrc(SGPR)
   llvm_i32_ty,      // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,      // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],     // auxiliary/cachepolicy(imm):
                     //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                     //                bit 3 = swz, bit 4 = scc (gfx90a)
                     //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                     //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                     //                bit 6 = swz
                     //           all: volatile op (bit 31, stripped at lowering)
  [IntrReadMem, ImmArg<ArgIndex<3>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;
def int_amdgcn_raw_buffer_load_format : AMDGPURawBufferLoad;
def int_amdgcn_raw_buffer_load : AMDGPURawBufferLoad;

class AMDGPURawAtomicBufferLoad<LLVMType data_ty = llvm_any_ty> : Intrinsic <
  [data_ty],
  [llvm_v4i32_ty,     // rsrc(SGPR)
   llvm_i32_ty,       // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,       // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],      // auxiliary data (imm, cachepolicy     (bit 0 = glc,
                      //                                       bit 1 = slc,
                      //                                       bit 2 = dlc on gfx10+),
                      //                      swizzled buffer (bit 3 = swz))
  [ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
````
- **L1473 EN**: Comment explains nearby logic, invariants, or intent: `which allow for improved reasoning about memory accesses.`.
  **L1473 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`which allow for improved reasoning about memory accesses.`。
- **L1474 EN**: Separator comment used for visual grouping.
  **L1474 CN**: 用于视觉分组的分隔注释。
- **L1475 EN**: Comment explains nearby logic, invariants, or intent: `Note that in the cachepolicy for all these intrinsics, bit 31 is not preserved`.
  **L1475 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note that in the cachepolicy for all these intrinsics, bit 31 is not preserved`。
- **L1476 EN**: Comment explains nearby logic, invariants, or intent: `through to final assembly selection and is used to signal that the buffer`.
  **L1476 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`through to final assembly selection and is used to signal that the buffer`。
- **L1477 EN**: Comment explains nearby logic, invariants, or intent: `operation is volatile.`.
  **L1477 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operation is volatile.`。
- **L1478 EN**: Declares class `AMDGPURawBufferLoad`.
  **L1478 CN**: 声明 class `AMDGPURawBufferLoad`。
- **L1479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty],`.
  **L1479 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty],`。
- **L1480 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1480 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1481 EN**: Continues logic associated with callable symbol `offset`.
  **L1481 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1482 EN**: Continues logic associated with callable symbol `soffset`.
  **L1482 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1483 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1483 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1484 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1484 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1485 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1485 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1486 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1486 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1487 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1487 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1488 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1488 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1489 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1489 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem, ImmArg<ArgIndex<3>>], "", [SDNPMemOperand]>,`.
  **L1490 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem, ImmArg<ArgIndex<3>>], "", [SDNPMemOperand]>,`。
- **L1491 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1491 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1492 EN**: Declares TableGen def `int_amdgcn_raw_buffer_load_format`.
  **L1492 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_load_format`。
- **L1493 EN**: Declares TableGen def `int_amdgcn_raw_buffer_load`.
  **L1493 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_load`。
- **L1494 EN**: Blank line separating nearby declarations or logic blocks.
  **L1494 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1495 EN**: Declares class `AMDGPURawAtomicBufferLoad<LLVMType`.
  **L1495 CN**: 声明 class `AMDGPURawAtomicBufferLoad<LLVMType`。
- **L1496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1496 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1497 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1497 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1498 EN**: Continues logic associated with callable symbol `offset`.
  **L1498 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1499 EN**: Continues logic associated with callable symbol `soffset`.
  **L1499 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],      // auxiliary data (imm, cachepolicy     (bit 0 = glc,`.
  **L1500 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],      // auxiliary data (imm, cachepolicy     (bit 0 = glc,`。
- **L1501 EN**: Comment explains nearby logic, invariants, or intent: `bit 1 = slc,`.
  **L1501 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 1 = slc,`。
- **L1502 EN**: Comment explains nearby logic, invariants, or intent: `bit 2 = dlc on gfx10+),`.
  **L1502 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 2 = dlc on gfx10+),`。
- **L1503 EN**: Comment explains nearby logic, invariants, or intent: `swizzled buffer (bit 3 = swz))`.
  **L1503 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`swizzled buffer (bit 3 = swz))`。
- **L1504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1504 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。

### Lines 1505-1536

````tablegen
  AMDGPURsrcIntrinsic<0>;
def int_amdgcn_raw_atomic_buffer_load : AMDGPURawAtomicBufferLoad;

class AMDGPURawPtrBufferLoad<LLVMType data_ty = llvm_any_ty> : DefaultAttrsIntrinsic <
  [data_ty],
  [AMDGPUBufferRsrcTy,    // rsrc(SGPR)
   llvm_i32_ty,           // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,           // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],          // auxiliary/cachepolicy(imm):
                          //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                          //                bit 3 = swz, bit 4 = scc (gfx90a)
                          //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                          //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                          //                bit 6 = swz
                          //           all: volatile op (bit 31, stripped at lowering)
  [IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,
  ImmArg<ArgIndex<3>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;
def int_amdgcn_raw_ptr_buffer_load_format : AMDGPURawPtrBufferLoad<llvm_anyfloat_ty>;
def int_amdgcn_raw_ptr_buffer_load : AMDGPURawPtrBufferLoad;

class AMDGPURawPtrAtomicBufferLoad<LLVMType data_ty = llvm_any_ty> : Intrinsic <
  [data_ty],
  [AMDGPUBufferRsrcTy,// rsrc(SGPR)
   llvm_i32_ty,       // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,       // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],      // auxiliary data (imm, cachepolicy     (bit 0 = glc,
                      //                                       bit 1 = slc,
                      //                                       bit 2 = dlc on gfx10+),
                      //                      swizzled buffer (bit 3 = swz))
  [IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;
````
- **L1505 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1505 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1506 EN**: Declares TableGen def `int_amdgcn_raw_atomic_buffer_load`.
  **L1506 CN**: 声明 TableGen def `int_amdgcn_raw_atomic_buffer_load`。
- **L1507 EN**: Blank line separating nearby declarations or logic blocks.
  **L1507 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1508 EN**: Declares class `AMDGPURawPtrBufferLoad<LLVMType`.
  **L1508 CN**: 声明 class `AMDGPURawPtrBufferLoad<LLVMType`。
- **L1509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1509 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1510 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1510 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1511 EN**: Continues logic associated with callable symbol `offset`.
  **L1511 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1512 EN**: Continues logic associated with callable symbol `soffset`.
  **L1512 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1513 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1513 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1514 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1514 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1515 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1515 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1516 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1516 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1517 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1517 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1518 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1518 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1519 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1519 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`.
  **L1520 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`。
- **L1521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<3>>], "", [SDNPMemOperand]>,`.
  **L1521 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<3>>], "", [SDNPMemOperand]>,`。
- **L1522 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1522 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1523 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_load_format`.
  **L1523 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_load_format`。
- **L1524 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_load`.
  **L1524 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_load`。
- **L1525 EN**: Blank line separating nearby declarations or logic blocks.
  **L1525 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1526 EN**: Declares class `AMDGPURawPtrAtomicBufferLoad<LLVMType`.
  **L1526 CN**: 声明 class `AMDGPURawPtrAtomicBufferLoad<LLVMType`。
- **L1527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1527 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1528 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1528 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1529 EN**: Continues logic associated with callable symbol `offset`.
  **L1529 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1530 EN**: Continues logic associated with callable symbol `soffset`.
  **L1530 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],      // auxiliary data (imm, cachepolicy     (bit 0 = glc,`.
  **L1531 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],      // auxiliary data (imm, cachepolicy     (bit 0 = glc,`。
- **L1532 EN**: Comment explains nearby logic, invariants, or intent: `bit 1 = slc,`.
  **L1532 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 1 = slc,`。
- **L1533 EN**: Comment explains nearby logic, invariants, or intent: `bit 2 = dlc on gfx10+),`.
  **L1533 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 2 = dlc on gfx10+),`。
- **L1534 EN**: Comment explains nearby logic, invariants, or intent: `swizzled buffer (bit 3 = swz))`.
  **L1534 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`swizzled buffer (bit 3 = swz))`。
- **L1535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1535 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1536 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1536 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。

### Lines 1537-1568

````tablegen
def int_amdgcn_raw_ptr_atomic_buffer_load : AMDGPURawPtrAtomicBufferLoad;

class AMDGPUStructBufferLoad<LLVMType data_ty = llvm_any_ty> : DefaultAttrsIntrinsic <
  [data_ty],
  [llvm_v4i32_ty,    // rsrc(SGPR)
   llvm_i32_ty,      // vindex(VGPR)
   llvm_i32_ty,      // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,      // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],     // auxiliary/cachepolicy(imm):
                     //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                     //                bit 3 = swz, bit 4 = scc (gfx90a)
                     //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                     //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                     //                bit 6 = swz
                     //           all: volatile op (bit 31, stripped at lowering)
  [IntrReadMem, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;
def int_amdgcn_struct_buffer_load_format : AMDGPUStructBufferLoad;
def int_amdgcn_struct_buffer_load : AMDGPUStructBufferLoad;

class AMDGPUStructAtomicBufferLoad<LLVMType data_ty = llvm_any_ty> : Intrinsic <
  [data_ty],
  [llvm_v4i32_ty,    // rsrc(SGPR)
   llvm_i32_ty,      // vindex(VGPR)
   llvm_i32_ty,      // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,      // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],     // auxiliary/cachepolicy(imm):
                     //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                     //                bit 3 = swz, bit 4 = scc (gfx90a)
                     //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                     //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                     //                bit 6 = swz
````
- **L1537 EN**: Declares TableGen def `int_amdgcn_raw_ptr_atomic_buffer_load`.
  **L1537 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_atomic_buffer_load`。
- **L1538 EN**: Blank line separating nearby declarations or logic blocks.
  **L1538 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1539 EN**: Declares class `AMDGPUStructBufferLoad<LLVMType`.
  **L1539 CN**: 声明 class `AMDGPUStructBufferLoad<LLVMType`。
- **L1540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1540 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1541 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1541 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1542 EN**: Continues logic associated with callable symbol `vindex`.
  **L1542 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1543 EN**: Continues logic associated with callable symbol `offset`.
  **L1543 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1544 EN**: Continues logic associated with callable symbol `soffset`.
  **L1544 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1545 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1545 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1546 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1546 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1547 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1547 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1548 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1548 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1549 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1549 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1550 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1550 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1551 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1551 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`.
  **L1552 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`。
- **L1553 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1553 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1554 EN**: Declares TableGen def `int_amdgcn_struct_buffer_load_format`.
  **L1554 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_load_format`。
- **L1555 EN**: Declares TableGen def `int_amdgcn_struct_buffer_load`.
  **L1555 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_load`。
- **L1556 EN**: Blank line separating nearby declarations or logic blocks.
  **L1556 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1557 EN**: Declares class `AMDGPUStructAtomicBufferLoad<LLVMType`.
  **L1557 CN**: 声明 class `AMDGPUStructAtomicBufferLoad<LLVMType`。
- **L1558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1558 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1559 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1559 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1560 EN**: Continues logic associated with callable symbol `vindex`.
  **L1560 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1561 EN**: Continues logic associated with callable symbol `offset`.
  **L1561 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1562 EN**: Continues logic associated with callable symbol `soffset`.
  **L1562 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1563 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1563 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1564 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1564 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1565 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1565 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1566 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1566 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1567 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1567 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1568 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1568 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。

### Lines 1569-1600

````tablegen
                     //           all: volatile op (bit 31, stripped at lowering)
  [ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;
def int_amdgcn_struct_atomic_buffer_load : AMDGPUStructAtomicBufferLoad;

class AMDGPUStructPtrBufferLoad<LLVMType data_ty = llvm_any_ty> : DefaultAttrsIntrinsic <
  [data_ty],
  [AMDGPUBufferRsrcTy,    // rsrc(SGPR)
   llvm_i32_ty,           // vindex(VGPR)
   llvm_i32_ty,           // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,           // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],          // auxiliary/cachepolicy(imm):
                          //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                          //                bit 3 = swz, bit 4 = scc (gfx90a)
                          //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                          //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                          //                bit 6 = swz
                          //           all: volatile op (bit 31, stripped at lowering)
  [IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,
   ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;
def int_amdgcn_struct_ptr_buffer_load_format : AMDGPUStructPtrBufferLoad;
def int_amdgcn_struct_ptr_buffer_load : AMDGPUStructPtrBufferLoad;

class AMDGPUStructPtrAtomicBufferLoad<LLVMType data_ty = llvm_any_ty> : Intrinsic <
  [data_ty],
  [AMDGPUBufferRsrcTy,    // rsrc(SGPR)
   llvm_i32_ty,           // vindex(VGPR)
   llvm_i32_ty,           // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,           // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],          // auxiliary/cachepolicy(imm):
                          //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
````
- **L1569 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1569 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1570 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1571 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1571 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1572 EN**: Declares TableGen def `int_amdgcn_struct_atomic_buffer_load`.
  **L1572 CN**: 声明 TableGen def `int_amdgcn_struct_atomic_buffer_load`。
- **L1573 EN**: Blank line separating nearby declarations or logic blocks.
  **L1573 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1574 EN**: Declares class `AMDGPUStructPtrBufferLoad<LLVMType`.
  **L1574 CN**: 声明 class `AMDGPUStructPtrBufferLoad<LLVMType`。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1576 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1576 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1577 EN**: Continues logic associated with callable symbol `vindex`.
  **L1577 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1578 EN**: Continues logic associated with callable symbol `offset`.
  **L1578 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1579 EN**: Continues logic associated with callable symbol `soffset`.
  **L1579 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1580 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1580 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1581 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1581 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1582 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1582 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1583 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1583 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1584 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1584 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1585 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1585 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1586 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1586 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`.
  **L1587 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`。
- **L1588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`.
  **L1588 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`。
- **L1589 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1589 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1590 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_load_format`.
  **L1590 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_load_format`。
- **L1591 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_load`.
  **L1591 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_load`。
- **L1592 EN**: Blank line separating nearby declarations or logic blocks.
  **L1592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1593 EN**: Declares class `AMDGPUStructPtrAtomicBufferLoad<LLVMType`.
  **L1593 CN**: 声明 class `AMDGPUStructPtrAtomicBufferLoad<LLVMType`。
- **L1594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1594 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1595 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1595 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1596 EN**: Continues logic associated with callable symbol `vindex`.
  **L1596 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1597 EN**: Continues logic associated with callable symbol `offset`.
  **L1597 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1598 EN**: Continues logic associated with callable symbol `soffset`.
  **L1598 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1599 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1599 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1600 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1600 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。

### Lines 1601-1632

````tablegen
                          //                bit 3 = swz, bit 4 = scc (gfx90a)
                          //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                          //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                          //                bit 6 = swz
                          //           all: volatile op (bit 31, stripped at lowering)
  [IntrArgMemOnly, NoCapture<ArgIndex<0>>,
   ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;
def int_amdgcn_struct_ptr_atomic_buffer_load : AMDGPUStructPtrAtomicBufferLoad;

class AMDGPURawBufferStore<LLVMType data_ty = llvm_any_ty> : DefaultAttrsIntrinsic <
  [],
  [data_ty,          // vdata(VGPR)
   llvm_v4i32_ty,    // rsrc(SGPR)
   llvm_i32_ty,      // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,      // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],     // auxiliary/cachepolicy(imm):
                     //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                     //                bit 3 = swz, bit 4 = scc (gfx90a)
                     //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                     //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                     //                bit 6 = swz
                     //           all: volatile op (bit 31, stripped at lowering)
  [IntrWriteMem, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1>;
def int_amdgcn_raw_buffer_store_format : AMDGPURawBufferStore<llvm_anyfloat_ty>;
def int_amdgcn_raw_buffer_store : AMDGPURawBufferStore;

class AMDGPURawPtrBufferStore<LLVMType data_ty = llvm_any_ty> : DefaultAttrsIntrinsic <
  [],
  [data_ty,               // vdata(VGPR)
   AMDGPUBufferRsrcTy,    // rsrc(SGPR)
````
- **L1601 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1601 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1602 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1602 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1603 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1603 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1604 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1604 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1605 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1605 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>,`.
  **L1606 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>,`。
- **L1607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1607 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1608 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1608 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1609 EN**: Declares TableGen def `int_amdgcn_struct_ptr_atomic_buffer_load`.
  **L1609 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_atomic_buffer_load`。
- **L1610 EN**: Blank line separating nearby declarations or logic blocks.
  **L1610 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1611 EN**: Declares class `AMDGPURawBufferStore<LLVMType`.
  **L1611 CN**: 声明 class `AMDGPURawBufferStore<LLVMType`。
- **L1612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L1612 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L1613 EN**: Continues logic associated with callable symbol `vdata`.
  **L1613 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1614 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1614 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1615 EN**: Continues logic associated with callable symbol `offset`.
  **L1615 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1616 EN**: Continues logic associated with callable symbol `soffset`.
  **L1616 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1617 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1617 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1618 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1618 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1619 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1619 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1620 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1620 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1621 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1621 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1622 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1622 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1623 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1623 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`.
  **L1624 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`。
- **L1625 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1>;`.
  **L1625 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1>;`。
- **L1626 EN**: Declares TableGen def `int_amdgcn_raw_buffer_store_format`.
  **L1626 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_store_format`。
- **L1627 EN**: Declares TableGen def `int_amdgcn_raw_buffer_store`.
  **L1627 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_store`。
- **L1628 EN**: Blank line separating nearby declarations or logic blocks.
  **L1628 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1629 EN**: Declares class `AMDGPURawPtrBufferStore<LLVMType`.
  **L1629 CN**: 声明 class `AMDGPURawPtrBufferStore<LLVMType`。
- **L1630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L1630 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L1631 EN**: Continues logic associated with callable symbol `vdata`.
  **L1631 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1632 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1632 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。

### Lines 1633-1664

````tablegen
   llvm_i32_ty,           // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,           // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],          // auxiliary/cachepolicy(imm):
                          //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                          //                bit 3 = swz, bit 4 = scc (gfx90a)
                          //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                          //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                          //                bit 6 = swz
                          //           all: volatile op (bit 31, stripped at lowering)
  [IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,
  ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1>;
def int_amdgcn_raw_ptr_buffer_store_format : AMDGPURawPtrBufferStore<llvm_anyfloat_ty>;
def int_amdgcn_raw_ptr_buffer_store : AMDGPURawPtrBufferStore;

class AMDGPUStructBufferStore<LLVMType data_ty = llvm_any_ty> : DefaultAttrsIntrinsic <
  [],
  [data_ty,          // vdata(VGPR)
   llvm_v4i32_ty,    // rsrc(SGPR)
   llvm_i32_ty,      // vindex(VGPR)
   llvm_i32_ty,      // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,      // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],     // auxiliary/cachepolicy(imm):
                     //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                     //                bit 3 = swz, bit 4 = scc (gfx90a)
                     //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                     //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                     //                bit 6 = swz
                     //           all: volatile op (bit 31, stripped at lowering)
  [IntrWriteMem, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1>;
def int_amdgcn_struct_buffer_store_format : AMDGPUStructBufferStore;
````
- **L1633 EN**: Continues logic associated with callable symbol `offset`.
  **L1633 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1634 EN**: Continues logic associated with callable symbol `soffset`.
  **L1634 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1635 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1635 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1636 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1636 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1637 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1637 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1638 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1638 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1639 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1639 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1640 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1640 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1641 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1641 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`.
  **L1642 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`。
- **L1643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`.
  **L1643 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`。
- **L1644 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1>;`.
  **L1644 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1>;`。
- **L1645 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_store_format`.
  **L1645 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_store_format`。
- **L1646 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_store`.
  **L1646 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_store`。
- **L1647 EN**: Blank line separating nearby declarations or logic blocks.
  **L1647 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1648 EN**: Declares class `AMDGPUStructBufferStore<LLVMType`.
  **L1648 CN**: 声明 class `AMDGPUStructBufferStore<LLVMType`。
- **L1649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L1649 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L1650 EN**: Continues logic associated with callable symbol `vdata`.
  **L1650 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1651 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1651 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1652 EN**: Continues logic associated with callable symbol `vindex`.
  **L1652 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1653 EN**: Continues logic associated with callable symbol `offset`.
  **L1653 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1654 EN**: Continues logic associated with callable symbol `soffset`.
  **L1654 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1655 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1655 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1656 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1656 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1657 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1657 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1658 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1658 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1659 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1659 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1660 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1660 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1661 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1661 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`.
  **L1662 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`。
- **L1663 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1>;`.
  **L1663 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1>;`。
- **L1664 EN**: Declares TableGen def `int_amdgcn_struct_buffer_store_format`.
  **L1664 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_store_format`。

### Lines 1665-1696

````tablegen
def int_amdgcn_struct_buffer_store : AMDGPUStructBufferStore;

class AMDGPUStructPtrBufferStore<LLVMType data_ty = llvm_any_ty> : DefaultAttrsIntrinsic <
  [],
  [data_ty,               // vdata(VGPR)
   AMDGPUBufferRsrcTy,    // rsrc(SGPR)
   llvm_i32_ty,           // vindex(VGPR)
   llvm_i32_ty,           // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,           // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],          // auxiliary/cachepolicy(imm):
                          //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                          //                bit 3 = swz, bit 4 = scc (gfx90a)
                          //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                          //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                          //                bit 6 = swz
                          //           all: volatile op (bit 31, stripped at lowering)
  [IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,
   ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1>;
def int_amdgcn_struct_ptr_buffer_store_format : AMDGPUStructPtrBufferStore;
def int_amdgcn_struct_ptr_buffer_store : AMDGPUStructPtrBufferStore;

class AMDGPURawBufferAtomic<LLVMType data_ty = llvm_any_ty> : Intrinsic <
  [data_ty],
  [LLVMMatchType<0>,  // vdata(VGPR)
   llvm_v4i32_ty,     // rsrc(SGPR)
   llvm_i32_ty,       // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,       // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],      // cachepolicy(imm; bit 1 = slc, ..., bit 31 = volatile)
  [ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1, 0>;
def int_amdgcn_raw_buffer_atomic_swap : AMDGPURawBufferAtomic;
````
- **L1665 EN**: Declares TableGen def `int_amdgcn_struct_buffer_store`.
  **L1665 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_store`。
- **L1666 EN**: Blank line separating nearby declarations or logic blocks.
  **L1666 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1667 EN**: Declares class `AMDGPUStructPtrBufferStore<LLVMType`.
  **L1667 CN**: 声明 class `AMDGPUStructPtrBufferStore<LLVMType`。
- **L1668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L1668 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L1669 EN**: Continues logic associated with callable symbol `vdata`.
  **L1669 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1670 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1670 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1671 EN**: Continues logic associated with callable symbol `vindex`.
  **L1671 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1672 EN**: Continues logic associated with callable symbol `offset`.
  **L1672 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1673 EN**: Continues logic associated with callable symbol `soffset`.
  **L1673 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1674 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1674 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1675 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1675 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1676 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1676 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1677 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1677 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1678 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1678 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1679 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1679 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1680 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1680 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`.
  **L1681 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`。
- **L1682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`.
  **L1682 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`。
- **L1683 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1>;`.
  **L1683 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1>;`。
- **L1684 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_store_format`.
  **L1684 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_store_format`。
- **L1685 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_store`.
  **L1685 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_store`。
- **L1686 EN**: Blank line separating nearby declarations or logic blocks.
  **L1686 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1687 EN**: Declares class `AMDGPURawBufferAtomic<LLVMType`.
  **L1687 CN**: 声明 class `AMDGPURawBufferAtomic<LLVMType`。
- **L1688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1688 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1689 EN**: Continues logic associated with callable symbol `vdata`.
  **L1689 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1690 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1690 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1691 EN**: Continues logic associated with callable symbol `offset`.
  **L1691 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1692 EN**: Continues logic associated with callable symbol `soffset`.
  **L1692 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1693 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1693 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1694 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1695 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1, 0>;`.
  **L1695 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1, 0>;`。
- **L1696 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_swap`.
  **L1696 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_swap`。

### Lines 1697-1728

````tablegen
def int_amdgcn_raw_buffer_atomic_add : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_sub : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_smin : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_umin : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_fmin : AMDGPURawBufferAtomic<llvm_anyfloat_ty>;
def int_amdgcn_raw_buffer_atomic_smax : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_umax : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_fmax : AMDGPURawBufferAtomic<llvm_anyfloat_ty>;
def int_amdgcn_raw_buffer_atomic_and : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_or : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_xor : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_inc : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_dec : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_cond_sub_u32 : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_sub_clamp_u32 : AMDGPURawBufferAtomic;
def int_amdgcn_raw_buffer_atomic_cmpswap : Intrinsic<
  [llvm_anyint_ty],
  [LLVMMatchType<0>,  // src(VGPR)
   LLVMMatchType<0>,  // cmp(VGPR)
   llvm_v4i32_ty,     // rsrc(SGPR)
   llvm_i32_ty,       // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,       // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],      // cachepolicy(imm; bit 1 = slc, ..., bit 31 = volatile)
  [ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<2, 0>;

class AMDGPURawPtrBufferAtomic<LLVMType data_ty = llvm_any_ty> : Intrinsic <
  [data_ty],
  [LLVMMatchType<0>,            // vdata(VGPR)
   AMDGPUBufferRsrcTy,          // rsrc(SGPR)
   llvm_i32_ty,                 // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,                 // soffset(SGPR/imm, excluded from bounds checking and swizzling)
````
- **L1697 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_add`.
  **L1697 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_add`。
- **L1698 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_sub`.
  **L1698 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_sub`。
- **L1699 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_smin`.
  **L1699 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_smin`。
- **L1700 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_umin`.
  **L1700 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_umin`。
- **L1701 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_fmin`.
  **L1701 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_fmin`。
- **L1702 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_smax`.
  **L1702 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_smax`。
- **L1703 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_umax`.
  **L1703 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_umax`。
- **L1704 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_fmax`.
  **L1704 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_fmax`。
- **L1705 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_and`.
  **L1705 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_and`。
- **L1706 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_or`.
  **L1706 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_or`。
- **L1707 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_xor`.
  **L1707 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_xor`。
- **L1708 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_inc`.
  **L1708 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_inc`。
- **L1709 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_dec`.
  **L1709 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_dec`。
- **L1710 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_cond_sub_u32`.
  **L1710 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_cond_sub_u32`。
- **L1711 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_sub_clamp_u32`.
  **L1711 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_sub_clamp_u32`。
- **L1712 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_cmpswap`.
  **L1712 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_cmpswap`。
- **L1713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyint_ty],`.
  **L1713 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyint_ty],`。
- **L1714 EN**: Continues logic associated with callable symbol `src`.
  **L1714 CN**: 继续与可调用符号 `src` 相关的逻辑。
- **L1715 EN**: Continues logic associated with callable symbol `cmp`.
  **L1715 CN**: 继续与可调用符号 `cmp` 相关的逻辑。
- **L1716 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1716 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1717 EN**: Continues logic associated with callable symbol `offset`.
  **L1717 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1718 EN**: Continues logic associated with callable symbol `soffset`.
  **L1718 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1719 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1719 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1720 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1721 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<2, 0>;`.
  **L1721 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<2, 0>;`。
- **L1722 EN**: Blank line separating nearby declarations or logic blocks.
  **L1722 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1723 EN**: Declares class `AMDGPURawPtrBufferAtomic<LLVMType`.
  **L1723 CN**: 声明 class `AMDGPURawPtrBufferAtomic<LLVMType`。
- **L1724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1724 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1725 EN**: Continues logic associated with callable symbol `vdata`.
  **L1725 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1726 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1726 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1727 EN**: Continues logic associated with callable symbol `offset`.
  **L1727 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1728 EN**: Continues logic associated with callable symbol `soffset`.
  **L1728 CN**: 继续与可调用符号 `soffset` 相关的逻辑。

### Lines 1729-1760

````tablegen
   llvm_i32_ty],                // cachepolicy(imm; bit 1 = slc, ..., bit 31 = volatile)
  [IntrArgMemOnly, NoCapture<ArgIndex<1>>,
   ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1, 0>;

def int_amdgcn_raw_ptr_buffer_atomic_swap : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_add : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_sub : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_smin : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_umin : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_fmin : AMDGPURawPtrBufferAtomic<llvm_anyfloat_ty>;
def int_amdgcn_raw_ptr_buffer_atomic_smax : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_umax : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_fmax : AMDGPURawPtrBufferAtomic<llvm_anyfloat_ty>;
def int_amdgcn_raw_ptr_buffer_atomic_and : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_or : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_xor : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_inc : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_dec : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_cond_sub_u32 : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_sub_clamp_u32 : AMDGPURawPtrBufferAtomic;
def int_amdgcn_raw_ptr_buffer_atomic_cmpswap : Intrinsic<
  [llvm_anyint_ty],
  [LLVMMatchType<0>,  // src(VGPR)
   LLVMMatchType<0>,  // cmp(VGPR)
   AMDGPUBufferRsrcTy, // rsrc(SGPR)
   llvm_i32_ty,       // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,       // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],      // cachepolicy(imm; bit 1 = slc, ..., bit 31 = volatile)
  [IntrArgMemOnly, NoCapture<ArgIndex<2>>,
   ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<2, 0>;
````
- **L1729 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1729 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, NoCapture<ArgIndex<1>>,`.
  **L1730 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, NoCapture<ArgIndex<1>>,`。
- **L1731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1731 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1732 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1, 0>;`.
  **L1732 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1, 0>;`。
- **L1733 EN**: Blank line separating nearby declarations or logic blocks.
  **L1733 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1734 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_swap`.
  **L1734 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_swap`。
- **L1735 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_add`.
  **L1735 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_add`。
- **L1736 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_sub`.
  **L1736 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_sub`。
- **L1737 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_smin`.
  **L1737 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_smin`。
- **L1738 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_umin`.
  **L1738 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_umin`。
- **L1739 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_fmin`.
  **L1739 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_fmin`。
- **L1740 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_smax`.
  **L1740 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_smax`。
- **L1741 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_umax`.
  **L1741 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_umax`。
- **L1742 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_fmax`.
  **L1742 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_fmax`。
- **L1743 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_and`.
  **L1743 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_and`。
- **L1744 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_or`.
  **L1744 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_or`。
- **L1745 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_xor`.
  **L1745 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_xor`。
- **L1746 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_inc`.
  **L1746 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_inc`。
- **L1747 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_dec`.
  **L1747 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_dec`。
- **L1748 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_cond_sub_u32`.
  **L1748 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_cond_sub_u32`。
- **L1749 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_sub_clamp_u32`.
  **L1749 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_sub_clamp_u32`。
- **L1750 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_cmpswap`.
  **L1750 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_cmpswap`。
- **L1751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyint_ty],`.
  **L1751 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyint_ty],`。
- **L1752 EN**: Continues logic associated with callable symbol `src`.
  **L1752 CN**: 继续与可调用符号 `src` 相关的逻辑。
- **L1753 EN**: Continues logic associated with callable symbol `cmp`.
  **L1753 CN**: 继续与可调用符号 `cmp` 相关的逻辑。
- **L1754 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1754 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1755 EN**: Continues logic associated with callable symbol `offset`.
  **L1755 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1756 EN**: Continues logic associated with callable symbol `soffset`.
  **L1756 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1757 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1757 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, NoCapture<ArgIndex<2>>,`.
  **L1758 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, NoCapture<ArgIndex<2>>,`。
- **L1759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1759 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1760 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<2, 0>;`.
  **L1760 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<2, 0>;`。

### Lines 1761-1792

````tablegen

// gfx908 intrinsic
def int_amdgcn_raw_buffer_atomic_fadd : AMDGPURawBufferAtomic<llvm_anyfloat_ty>;

// Supports float and <2 x half> on gfx908. Supports v2bf16 on gfx90a, gfx942, gfx950, gfx12+.
def int_amdgcn_raw_ptr_buffer_atomic_fadd : AMDGPURawPtrBufferAtomic<llvm_anyfloat_ty>;

class AMDGPUStructBufferAtomic<LLVMType data_ty = llvm_any_ty> : Intrinsic <
  [data_ty],
  [LLVMMatchType<0>,  // vdata(VGPR)
   llvm_v4i32_ty,     // rsrc(SGPR)
   llvm_i32_ty,       // vindex(VGPR)
   llvm_i32_ty,       // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,       // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],      // cachepolicy(imm; bit 1 = slc, ..., bit 31 = volatile)
  [ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1, 0>;
def int_amdgcn_struct_buffer_atomic_swap : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_add : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_sub : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_smin : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_umin : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_smax : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_umax : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_and : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_or : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_xor : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_inc : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_dec : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_cond_sub_u32 : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_sub_clamp_u32 : AMDGPUStructBufferAtomic;
def int_amdgcn_struct_buffer_atomic_cmpswap : Intrinsic<
````
- **L1761 EN**: Blank line separating nearby declarations or logic blocks.
  **L1761 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1762 EN**: Comment explains nearby logic, invariants, or intent: `gfx908 intrinsic`.
  **L1762 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx908 intrinsic`。
- **L1763 EN**: Declares TableGen def `int_amdgcn_raw_buffer_atomic_fadd`.
  **L1763 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_atomic_fadd`。
- **L1764 EN**: Blank line separating nearby declarations or logic blocks.
  **L1764 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1765 EN**: Comment explains nearby logic, invariants, or intent: `Supports float and <2 x half> on gfx908. Supports v2bf16 on gfx90a, gfx942, gfx950, gfx12+.`.
  **L1765 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Supports float and <2 x half> on gfx908. Supports v2bf16 on gfx90a, gfx942, gfx950, gfx12+.`。
- **L1766 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_atomic_fadd`.
  **L1766 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_atomic_fadd`。
- **L1767 EN**: Blank line separating nearby declarations or logic blocks.
  **L1767 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1768 EN**: Declares class `AMDGPUStructBufferAtomic<LLVMType`.
  **L1768 CN**: 声明 class `AMDGPUStructBufferAtomic<LLVMType`。
- **L1769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1769 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1770 EN**: Continues logic associated with callable symbol `vdata`.
  **L1770 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1771 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1771 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1772 EN**: Continues logic associated with callable symbol `vindex`.
  **L1772 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1773 EN**: Continues logic associated with callable symbol `offset`.
  **L1773 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1774 EN**: Continues logic associated with callable symbol `soffset`.
  **L1774 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1775 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1775 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1776 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1777 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1, 0>;`.
  **L1777 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1, 0>;`。
- **L1778 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_swap`.
  **L1778 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_swap`。
- **L1779 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_add`.
  **L1779 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_add`。
- **L1780 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_sub`.
  **L1780 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_sub`。
- **L1781 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_smin`.
  **L1781 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_smin`。
- **L1782 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_umin`.
  **L1782 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_umin`。
- **L1783 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_smax`.
  **L1783 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_smax`。
- **L1784 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_umax`.
  **L1784 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_umax`。
- **L1785 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_and`.
  **L1785 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_and`。
- **L1786 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_or`.
  **L1786 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_or`。
- **L1787 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_xor`.
  **L1787 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_xor`。
- **L1788 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_inc`.
  **L1788 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_inc`。
- **L1789 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_dec`.
  **L1789 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_dec`。
- **L1790 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_cond_sub_u32`.
  **L1790 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_cond_sub_u32`。
- **L1791 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_sub_clamp_u32`.
  **L1791 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_sub_clamp_u32`。
- **L1792 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_cmpswap`.
  **L1792 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_cmpswap`。

### Lines 1793-1824

````tablegen
  [llvm_anyint_ty],
  [LLVMMatchType<0>,  // src(VGPR)
   LLVMMatchType<0>,  // cmp(VGPR)
   llvm_v4i32_ty,     // rsrc(SGPR)
   llvm_i32_ty,       // vindex(VGPR)
   llvm_i32_ty,       // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,       // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],      // cachepolicy(imm; bit 1 = slc, ..., bit 31 = volatile)
  [ImmArg<ArgIndex<6>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<2, 0>;

class AMDGPUStructPtrBufferAtomic<LLVMType data_ty = llvm_any_ty> : Intrinsic <
  [data_ty],
  [LLVMMatchType<0>,            // vdata(VGPR)
   AMDGPUBufferRsrcTy,          // rsrc(SGPR)
   llvm_i32_ty,                 // vindex(VGPR)
   llvm_i32_ty,                 // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,                 // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],                // cachepolicy(imm; bit 1 = slc, ..., bit 31 = volatile)
  [IntrArgMemOnly, NoCapture<ArgIndex<1>>,
   ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1, 0>;
def int_amdgcn_struct_ptr_buffer_atomic_swap : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_add : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_sub : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_smin : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_umin : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_smax : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_umax : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_and : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_or : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_xor : AMDGPUStructPtrBufferAtomic;
````
- **L1793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyint_ty],`.
  **L1793 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyint_ty],`。
- **L1794 EN**: Continues logic associated with callable symbol `src`.
  **L1794 CN**: 继续与可调用符号 `src` 相关的逻辑。
- **L1795 EN**: Continues logic associated with callable symbol `cmp`.
  **L1795 CN**: 继续与可调用符号 `cmp` 相关的逻辑。
- **L1796 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1796 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1797 EN**: Continues logic associated with callable symbol `vindex`.
  **L1797 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1798 EN**: Continues logic associated with callable symbol `offset`.
  **L1798 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1799 EN**: Continues logic associated with callable symbol `soffset`.
  **L1799 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1800 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1800 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<6>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1801 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<6>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1802 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<2, 0>;`.
  **L1802 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<2, 0>;`。
- **L1803 EN**: Blank line separating nearby declarations or logic blocks.
  **L1803 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1804 EN**: Declares class `AMDGPUStructPtrBufferAtomic<LLVMType`.
  **L1804 CN**: 声明 class `AMDGPUStructPtrBufferAtomic<LLVMType`。
- **L1805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L1805 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L1806 EN**: Continues logic associated with callable symbol `vdata`.
  **L1806 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1807 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1807 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1808 EN**: Continues logic associated with callable symbol `vindex`.
  **L1808 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1809 EN**: Continues logic associated with callable symbol `offset`.
  **L1809 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1810 EN**: Continues logic associated with callable symbol `soffset`.
  **L1810 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1811 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1811 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, NoCapture<ArgIndex<1>>,`.
  **L1812 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, NoCapture<ArgIndex<1>>,`。
- **L1813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1813 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1814 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1, 0>;`.
  **L1814 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1, 0>;`。
- **L1815 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_swap`.
  **L1815 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_swap`。
- **L1816 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_add`.
  **L1816 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_add`。
- **L1817 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_sub`.
  **L1817 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_sub`。
- **L1818 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_smin`.
  **L1818 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_smin`。
- **L1819 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_umin`.
  **L1819 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_umin`。
- **L1820 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_smax`.
  **L1820 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_smax`。
- **L1821 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_umax`.
  **L1821 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_umax`。
- **L1822 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_and`.
  **L1822 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_and`。
- **L1823 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_or`.
  **L1823 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_or`。
- **L1824 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_xor`.
  **L1824 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_xor`。

### Lines 1825-1856

````tablegen
def int_amdgcn_struct_ptr_buffer_atomic_inc : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_dec : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_cond_sub_u32 : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_sub_clamp_u32 : AMDGPUStructPtrBufferAtomic;
def int_amdgcn_struct_ptr_buffer_atomic_cmpswap : Intrinsic<
  [llvm_anyint_ty],
  [LLVMMatchType<0>,  // src(VGPR)
   LLVMMatchType<0>,  // cmp(VGPR)
   AMDGPUBufferRsrcTy, // rsrc(SGPR)
   llvm_i32_ty,       // vindex(VGPR)
   llvm_i32_ty,       // offset(VGPR/imm, included in bounds checking and swizzling)
   llvm_i32_ty,       // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty],      // cachepolicy(imm; bit 1 = slc, ..., bit 31 = volatile)
  [IntrArgMemOnly, NoCapture<ArgIndex<2>>,
   ImmArg<ArgIndex<6>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<2, 0>;

// gfx908 intrinsic. Supports v2bf16 on gfx12+ and gfx950
def int_amdgcn_struct_buffer_atomic_fadd : AMDGPUStructBufferAtomic<llvm_anyfloat_ty>;
def int_amdgcn_struct_ptr_buffer_atomic_fadd : AMDGPUStructPtrBufferAtomic<llvm_anyfloat_ty>;

// gfx90a intrinsics
def int_amdgcn_struct_buffer_atomic_fmin : AMDGPUStructBufferAtomic<llvm_anyfloat_ty>;
def int_amdgcn_struct_buffer_atomic_fmax : AMDGPUStructBufferAtomic<llvm_anyfloat_ty>;

def int_amdgcn_struct_ptr_buffer_atomic_fmin : AMDGPUStructPtrBufferAtomic<llvm_anyfloat_ty>;
def int_amdgcn_struct_ptr_buffer_atomic_fmax : AMDGPUStructPtrBufferAtomic<llvm_anyfloat_ty>;

// tbuffer intrinsics, with:
// - raw and struct variants
// - joint format field
// - joint cachepolicy field
````
- **L1825 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_inc`.
  **L1825 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_inc`。
- **L1826 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_dec`.
  **L1826 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_dec`。
- **L1827 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_cond_sub_u32`.
  **L1827 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_cond_sub_u32`。
- **L1828 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_sub_clamp_u32`.
  **L1828 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_sub_clamp_u32`。
- **L1829 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_cmpswap`.
  **L1829 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_cmpswap`。
- **L1830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyint_ty],`.
  **L1830 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyint_ty],`。
- **L1831 EN**: Continues logic associated with callable symbol `src`.
  **L1831 CN**: 继续与可调用符号 `src` 相关的逻辑。
- **L1832 EN**: Continues logic associated with callable symbol `cmp`.
  **L1832 CN**: 继续与可调用符号 `cmp` 相关的逻辑。
- **L1833 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1833 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1834 EN**: Continues logic associated with callable symbol `vindex`.
  **L1834 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1835 EN**: Continues logic associated with callable symbol `offset`.
  **L1835 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1836 EN**: Continues logic associated with callable symbol `soffset`.
  **L1836 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1837 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1837 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, NoCapture<ArgIndex<2>>,`.
  **L1838 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, NoCapture<ArgIndex<2>>,`。
- **L1839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<6>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`.
  **L1839 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<6>>, IntrWillReturn, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>,`。
- **L1840 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<2, 0>;`.
  **L1840 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<2, 0>;`。
- **L1841 EN**: Blank line separating nearby declarations or logic blocks.
  **L1841 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1842 EN**: Comment explains nearby logic, invariants, or intent: `gfx908 intrinsic. Supports v2bf16 on gfx12+ and gfx950`.
  **L1842 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx908 intrinsic. Supports v2bf16 on gfx12+ and gfx950`。
- **L1843 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_fadd`.
  **L1843 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_fadd`。
- **L1844 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_fadd`.
  **L1844 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_fadd`。
- **L1845 EN**: Blank line separating nearby declarations or logic blocks.
  **L1845 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1846 EN**: Comment explains nearby logic, invariants, or intent: `gfx90a intrinsics`.
  **L1846 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx90a intrinsics`。
- **L1847 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_fmin`.
  **L1847 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_fmin`。
- **L1848 EN**: Declares TableGen def `int_amdgcn_struct_buffer_atomic_fmax`.
  **L1848 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_atomic_fmax`。
- **L1849 EN**: Blank line separating nearby declarations or logic blocks.
  **L1849 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1850 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_fmin`.
  **L1850 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_fmin`。
- **L1851 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_atomic_fmax`.
  **L1851 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_atomic_fmax`。
- **L1852 EN**: Blank line separating nearby declarations or logic blocks.
  **L1852 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1853 EN**: Comment explains nearby logic, invariants, or intent: `tbuffer intrinsics, with:`.
  **L1853 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tbuffer intrinsics, with:`。
- **L1854 EN**: Comment explains nearby logic, invariants, or intent: `- raw and struct variants`.
  **L1854 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`- raw and struct variants`。
- **L1855 EN**: Comment explains nearby logic, invariants, or intent: `- joint format field`.
  **L1855 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`- joint format field`。
- **L1856 EN**: Comment explains nearby logic, invariants, or intent: `- joint cachepolicy field`.
  **L1856 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`- joint cachepolicy field`。

### Lines 1857-1888

````tablegen
def int_amdgcn_raw_tbuffer_load : DefaultAttrsIntrinsic <
    [llvm_any_ty],    // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32
    [llvm_v4i32_ty,   // rsrc(SGPR)
     llvm_i32_ty,     // offset(VGPR/imm, included in bounds checking and swizzling)
     llvm_i32_ty,     // soffset(SGPR/imm, excluded from bounds checking and swizzling)
     llvm_i32_ty,     // format(imm; bits 3..0 = dfmt, bits 6..4 = nfmt)
     llvm_i32_ty],    // auxiliary/cachepolicy(imm):
                      //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                      //                bit 3 = swz, bit 4 = scc (gfx90a)
                      //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                      //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                      //                bit 6 = swz
    [IntrReadMem,
     ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;

def int_amdgcn_raw_ptr_tbuffer_load : DefaultAttrsIntrinsic <
    [llvm_any_ty],       // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32
    [AMDGPUBufferRsrcTy, // rsrc(SGPR)
     llvm_i32_ty,       // offset(VGPR/imm, included in bounds` checking and swizzling)
     llvm_i32_ty,       // soffset(SGPR/imm, excluded from bounds checking and swizzling)
     llvm_i32_ty,       // format(imm; bits 3..0 = dfmt, bits 6..4 = nfmt)
     llvm_i32_ty],      // auxiliary/cachepolicy(imm):
                        //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                        //                bit 3 = swz, bit 4 = scc (gfx90a)
                        //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                        //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                        //                bit 6 = swz
                        //           all: volatile op (bit 31, stripped at lowering)
    [IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,
     ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;
````
- **L1857 EN**: Declares TableGen def `int_amdgcn_raw_tbuffer_load`.
  **L1857 CN**: 声明 TableGen def `int_amdgcn_raw_tbuffer_load`。
- **L1858 EN**: Continues the surrounding expression or declaration: `[llvm_any_ty],    // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32`.
  **L1858 CN**: 继续构造周围的表达式或声明：`[llvm_any_ty],    // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32`。
- **L1859 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1859 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1860 EN**: Continues logic associated with callable symbol `offset`.
  **L1860 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1861 EN**: Continues logic associated with callable symbol `soffset`.
  **L1861 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1862 EN**: Continues logic associated with callable symbol `format`.
  **L1862 CN**: 继续与可调用符号 `format` 相关的逻辑。
- **L1863 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1863 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1864 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1864 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1865 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1865 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1866 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1866 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1867 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1867 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1868 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1868 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem,`.
  **L1869 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem,`。
- **L1870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`.
  **L1870 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`。
- **L1871 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1871 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1872 EN**: Blank line separating nearby declarations or logic blocks.
  **L1872 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1873 EN**: Declares TableGen def `int_amdgcn_raw_ptr_tbuffer_load`.
  **L1873 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_tbuffer_load`。
- **L1874 EN**: Continues the surrounding expression or declaration: `[llvm_any_ty],       // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32`.
  **L1874 CN**: 继续构造周围的表达式或声明：`[llvm_any_ty],       // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32`。
- **L1875 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1875 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1876 EN**: Continues logic associated with callable symbol `offset`.
  **L1876 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1877 EN**: Continues logic associated with callable symbol `soffset`.
  **L1877 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1878 EN**: Continues logic associated with callable symbol `format`.
  **L1878 CN**: 继续与可调用符号 `format` 相关的逻辑。
- **L1879 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1879 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1880 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1880 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1881 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1881 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1882 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1882 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1883 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1883 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1884 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1884 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1885 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1885 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`.
  **L1886 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`。
- **L1887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`.
  **L1887 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>], "", [SDNPMemOperand]>,`。
- **L1888 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1888 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。

### Lines 1889-1920

````tablegen

def int_amdgcn_raw_tbuffer_store : DefaultAttrsIntrinsic <
    [],
    [llvm_any_ty,    // vdata(VGPR), overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32
     llvm_v4i32_ty,  // rsrc(SGPR)
     llvm_i32_ty,    // offset(VGPR/imm, included in bounds checking and swizzling)
     llvm_i32_ty,    // soffset(SGPR/imm, excluded from bounds checking and swizzling)
     llvm_i32_ty,    // format(imm; bits 3..0 = dfmt, bits 6..4 = nfmt)
     llvm_i32_ty],   // auxiliary/cachepolicy(imm):
                     //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                     //                bit 3 = swz, bit 4 = scc (gfx90a)
                     //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                     //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                     //                bit 6 = swz
                     //           all: volatile op (bit 31, stripped at lowering)
    [IntrWriteMem,
     ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1>;

def int_amdgcn_raw_ptr_tbuffer_store : DefaultAttrsIntrinsic <
    [],
    [llvm_any_ty,    // vdata(VGPR), overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32
     AMDGPUBufferRsrcTy, // rsrc(SGPR)
     llvm_i32_ty,    // offset(VGPR/imm, included in bounds checking and swizzling)
     llvm_i32_ty,    // soffset(SGPR/imm, excluded from bounds checking and swizzling)
     llvm_i32_ty,    // format(imm; bits 3..0 = dfmt, bits 6..4 = nfmt)
     llvm_i32_ty],   // auxiliary/cachepolicy(imm):
                     //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                     //                bit 3 = swz, bit 4 = scc (gfx90a)
                     //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                     //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                     //                bit 6 = swz
````
- **L1889 EN**: Blank line separating nearby declarations or logic blocks.
  **L1889 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1890 EN**: Declares TableGen def `int_amdgcn_raw_tbuffer_store`.
  **L1890 CN**: 声明 TableGen def `int_amdgcn_raw_tbuffer_store`。
- **L1891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L1891 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L1892 EN**: Continues logic associated with callable symbol `vdata`.
  **L1892 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1893 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1893 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1894 EN**: Continues logic associated with callable symbol `offset`.
  **L1894 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1895 EN**: Continues logic associated with callable symbol `soffset`.
  **L1895 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1896 EN**: Continues logic associated with callable symbol `format`.
  **L1896 CN**: 继续与可调用符号 `format` 相关的逻辑。
- **L1897 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1897 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1898 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1898 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1899 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1899 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1900 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1900 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1901 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1901 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1902 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1902 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1903 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1903 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem,`.
  **L1904 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem,`。
- **L1905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`.
  **L1905 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`。
- **L1906 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1>;`.
  **L1906 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1>;`。
- **L1907 EN**: Blank line separating nearby declarations or logic blocks.
  **L1907 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1908 EN**: Declares TableGen def `int_amdgcn_raw_ptr_tbuffer_store`.
  **L1908 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_tbuffer_store`。
- **L1909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L1909 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L1910 EN**: Continues logic associated with callable symbol `vdata`.
  **L1910 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1911 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1911 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1912 EN**: Continues logic associated with callable symbol `offset`.
  **L1912 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1913 EN**: Continues logic associated with callable symbol `soffset`.
  **L1913 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1914 EN**: Continues logic associated with callable symbol `format`.
  **L1914 CN**: 继续与可调用符号 `format` 相关的逻辑。
- **L1915 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1915 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1916 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1916 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1917 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1917 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1918 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1918 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1919 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1919 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1920 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1920 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。

### Lines 1921-1952

````tablegen
                     //           all: volatile op (bit 31, stripped at lowering)
    [IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,
     ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1>;

def int_amdgcn_struct_tbuffer_load : DefaultAttrsIntrinsic <
    [llvm_any_ty],    // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32
    [llvm_v4i32_ty,   // rsrc(SGPR)
     llvm_i32_ty,     // vindex(VGPR)
     llvm_i32_ty,     // offset(VGPR/imm, included in bounds checking and swizzling)
     llvm_i32_ty,     // soffset(SGPR/imm, excluded from bounds checking and swizzling)
     llvm_i32_ty,     // format(imm; bits 3..0 = dfmt, bits 6..4 = nfmt)
     llvm_i32_ty],    // auxiliary/cachepolicy(imm):
                      //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                      //                bit 3 = swz, bit 4 = scc (gfx90a)
                      //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                      //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                      //                bit 6 = swz
                      //           all: volatile op (bit 31, stripped at lowering)
    [IntrReadMem,
     ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;

def int_amdgcn_struct_ptr_tbuffer_load : DefaultAttrsIntrinsic <
    [llvm_any_ty],       // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32
    [AMDGPUBufferRsrcTy, // rsrc(SGPR)
     llvm_i32_ty,        // vindex(VGPR)
     llvm_i32_ty,        // offset(VGPR/imm, included in bounds checking and swizzling)
     llvm_i32_ty,        // soffset(SGPR/imm, excluded from bounds checking and swizzling)
     llvm_i32_ty,        // format(imm; bits 3..0 = dfmt, bits 6..4 = nfmt)
     llvm_i32_ty],       // auxiliary/cachepolicy(imm):
                         //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
````
- **L1921 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1921 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`.
  **L1922 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`。
- **L1923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`.
  **L1923 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`。
- **L1924 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1>;`.
  **L1924 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1>;`。
- **L1925 EN**: Blank line separating nearby declarations or logic blocks.
  **L1925 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1926 EN**: Declares TableGen def `int_amdgcn_struct_tbuffer_load`.
  **L1926 CN**: 声明 TableGen def `int_amdgcn_struct_tbuffer_load`。
- **L1927 EN**: Continues the surrounding expression or declaration: `[llvm_any_ty],    // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32`.
  **L1927 CN**: 继续构造周围的表达式或声明：`[llvm_any_ty],    // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32`。
- **L1928 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1928 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1929 EN**: Continues logic associated with callable symbol `vindex`.
  **L1929 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1930 EN**: Continues logic associated with callable symbol `offset`.
  **L1930 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1931 EN**: Continues logic associated with callable symbol `soffset`.
  **L1931 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1932 EN**: Continues logic associated with callable symbol `format`.
  **L1932 CN**: 继续与可调用符号 `format` 相关的逻辑。
- **L1933 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1933 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1934 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1934 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1935 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1935 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1936 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1936 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1937 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1937 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1938 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1938 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1939 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1939 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem,`.
  **L1940 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem,`。
- **L1941 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`.
  **L1941 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`。
- **L1942 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1942 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1943 EN**: Blank line separating nearby declarations or logic blocks.
  **L1943 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1944 EN**: Declares TableGen def `int_amdgcn_struct_ptr_tbuffer_load`.
  **L1944 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_tbuffer_load`。
- **L1945 EN**: Continues the surrounding expression or declaration: `[llvm_any_ty],       // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32`.
  **L1945 CN**: 继续构造周围的表达式或声明：`[llvm_any_ty],       // overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32`。
- **L1946 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1946 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1947 EN**: Continues logic associated with callable symbol `vindex`.
  **L1947 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1948 EN**: Continues logic associated with callable symbol `offset`.
  **L1948 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1949 EN**: Continues logic associated with callable symbol `soffset`.
  **L1949 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1950 EN**: Continues logic associated with callable symbol `format`.
  **L1950 CN**: 继续与可调用符号 `format` 相关的逻辑。
- **L1951 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1951 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1952 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1952 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。

### Lines 1953-1984

````tablegen
                         //                bit 3 = swz, bit 4 = scc (gfx90a)
                         //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                         //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                         //                bit 6 = swz
                         //           all: volatile op (bit 31, stripped at lowering)
    [IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,
     ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>;

def int_amdgcn_struct_ptr_tbuffer_store : DefaultAttrsIntrinsic <
    [],
    [llvm_any_ty,        // vdata(VGPR), overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32
     AMDGPUBufferRsrcTy, // rsrc(SGPR)
     llvm_i32_ty,        // vindex(VGPR)
     llvm_i32_ty,        // offset(VGPR/imm, included in bounds checking and swizzling)
     llvm_i32_ty,        // soffset(SGPR/imm, excluded from bounds checking and swizzling)
     llvm_i32_ty,        // format(imm; bits 3..0 = dfmt, bits 6..4 = nfmt)
     llvm_i32_ty],       // auxiliary/cachepolicy(imm):
                         //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                         //                bit 3 = swz, bit 4 = scc (gfx90a)
                         //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                         //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                         //                bit 6 = swz
                         //           all: volatile op (bit 31, stripped at lowering)
    [IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,
     ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1>;

def int_amdgcn_struct_tbuffer_store : DefaultAttrsIntrinsic <
    [],
    [llvm_any_ty,    // vdata(VGPR), overloaded for types f32/i32, v2f32/v2i32, v4f32/v4i32
     llvm_v4i32_ty,  // rsrc(SGPR)
````
- **L1953 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1953 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1954 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1954 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1955 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1955 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1956 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1956 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1957 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1957 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1958 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`.
  **L1958 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`。
- **L1959 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`.
  **L1959 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>], "", [SDNPMemOperand]>,`。
- **L1960 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<0>;`.
  **L1960 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<0>;`。
- **L1961 EN**: Blank line separating nearby declarations or logic blocks.
  **L1961 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1962 EN**: Declares TableGen def `int_amdgcn_struct_ptr_tbuffer_store`.
  **L1962 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_tbuffer_store`。
- **L1963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L1963 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L1964 EN**: Continues logic associated with callable symbol `vdata`.
  **L1964 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1965 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1965 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L1966 EN**: Continues logic associated with callable symbol `vindex`.
  **L1966 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1967 EN**: Continues logic associated with callable symbol `offset`.
  **L1967 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1968 EN**: Continues logic associated with callable symbol `soffset`.
  **L1968 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1969 EN**: Continues logic associated with callable symbol `format`.
  **L1969 CN**: 继续与可调用符号 `format` 相关的逻辑。
- **L1970 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1970 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1971 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1971 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1972 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1972 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1973 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1973 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1974 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1974 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1975 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1975 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1976 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1976 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`.
  **L1977 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`。
- **L1978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>], "", [SDNPMemOperand]>,`.
  **L1978 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>], "", [SDNPMemOperand]>,`。
- **L1979 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1>;`.
  **L1979 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1>;`。
- **L1980 EN**: Blank line separating nearby declarations or logic blocks.
  **L1980 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1981 EN**: Declares TableGen def `int_amdgcn_struct_tbuffer_store`.
  **L1981 CN**: 声明 TableGen def `int_amdgcn_struct_tbuffer_store`。
- **L1982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L1982 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L1983 EN**: Continues logic associated with callable symbol `vdata`.
  **L1983 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L1984 EN**: Continues logic associated with callable symbol `rsrc`.
  **L1984 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。

### Lines 1985-2016

````tablegen
     llvm_i32_ty,    // vindex(VGPR)
     llvm_i32_ty,    // offset(VGPR/imm, included in bounds checking and swizzling)
     llvm_i32_ty,    // soffset(SGPR/imm, excluded from bounds checking and swizzling)
     llvm_i32_ty,    // format(imm; bits 3..0 = dfmt, bits 6..4 = nfmt)
     llvm_i32_ty],   // auxiliary/cachepolicy(imm):
                     //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                     //                bit 3 = swz, bit 4 = scc (gfx90a)
                     //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                     //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                     //                bit 6 = swz
                     //           all: volatile op (bit 31, stripped at lowering)
    [IntrWriteMem,
     ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<1>;

class AMDGPURawBufferLoadLDS : Intrinsic <
  [],
  [llvm_v4i32_ty,             // rsrc(SGPR)
   LLVMQualPointerType<3>,    // LDS base offset
   llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)
   llvm_i32_ty,               // voffset(VGPR, included in bounds checking and swizzling)
   llvm_i32_ty,               // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty,               // imm offset(imm, included in bounds checking and swizzling)
   llvm_i32_ty],              // auxiliary/cachepolicy(imm):
                              //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                              //                bit 3 = swz, bit 4 = scc (gfx90a)
                              //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                              //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                              //                bit 6 = swz
                              //           all: volatile op (bit 31, stripped at lowering)
  [IntrWillReturn, NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>,
   ImmArg<ArgIndex<6>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;
````
- **L1985 EN**: Continues logic associated with callable symbol `vindex`.
  **L1985 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L1986 EN**: Continues logic associated with callable symbol `offset`.
  **L1986 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L1987 EN**: Continues logic associated with callable symbol `soffset`.
  **L1987 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L1988 EN**: Continues logic associated with callable symbol `format`.
  **L1988 CN**: 继续与可调用符号 `format` 相关的逻辑。
- **L1989 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L1989 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L1990 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L1990 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L1991 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L1991 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L1992 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L1992 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L1993 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L1993 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L1994 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L1994 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L1995 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L1995 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L1996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem,`.
  **L1996 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem,`。
- **L1997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>], "", [SDNPMemOperand]>,`.
  **L1997 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>], "", [SDNPMemOperand]>,`。
- **L1998 EN**: Executes a standalone statement or declaration: `AMDGPURsrcIntrinsic<1>;`.
  **L1998 CN**: 执行一条独立语句或声明：`AMDGPURsrcIntrinsic<1>;`。
- **L1999 EN**: Blank line separating nearby declarations or logic blocks.
  **L1999 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2000 EN**: Declares class `AMDGPURawBufferLoadLDS`.
  **L2000 CN**: 声明 class `AMDGPURawBufferLoadLDS`。
- **L2001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L2001 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L2002 EN**: Continues logic associated with callable symbol `rsrc`.
  **L2002 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L2003 EN**: Continues the surrounding expression or declaration: `LLVMQualPointerType<3>,    // LDS base offset`.
  **L2003 CN**: 继续构造周围的表达式或声明：`LLVMQualPointerType<3>,    // LDS base offset`。
- **L2004 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)`.
  **L2004 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)`。
- **L2005 EN**: Continues logic associated with callable symbol `voffset`.
  **L2005 CN**: 继续与可调用符号 `voffset` 相关的逻辑。
- **L2006 EN**: Continues logic associated with callable symbol `soffset`.
  **L2006 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L2007 EN**: Continues logic associated with callable symbol `offset`.
  **L2007 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L2008 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L2008 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L2009 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L2009 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L2010 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L2010 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L2011 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L2011 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L2012 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L2012 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L2013 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L2013 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L2014 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L2014 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L2015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWillReturn, NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>,`.
  **L2015 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWillReturn, NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>,`。
- **L2016 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<6>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;`.
  **L2016 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<6>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;`。

### Lines 2017-2048

````tablegen
def int_amdgcn_raw_buffer_load_lds : AMDGPURawBufferLoadLDS;
def int_amdgcn_raw_buffer_load_async_lds : AMDGPURawBufferLoadLDS;

class AMDGPURawPtrBufferLoadLDS :
  Intrinsic <
  [],
  [AMDGPUBufferRsrcTy,        // rsrc(SGPR)
   LLVMQualPointerType<3>,    // LDS base offset
   llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)
   llvm_i32_ty,               // voffset(VGPR, included in bounds checking and swizzling)
   llvm_i32_ty,               // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty,               // imm offset(imm, included in bounds checking and swizzling)
   llvm_i32_ty],              // auxiliary/cachepolicy(imm):
                              //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                              //                bit 3 = swz, bit 4 = scc (gfx90a)
                              //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                              //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                              //                bit 6 = swz
                              //           all: volatile op (bit 31, stripped at lowering)
  [IntrWillReturn, IntrArgMemOnly,
   ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,
   WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,
   ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>,
   ImmArg<ArgIndex<6>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;
def int_amdgcn_raw_ptr_buffer_load_lds : AMDGPURawPtrBufferLoadLDS,
      ClangBuiltin<"__builtin_amdgcn_raw_ptr_buffer_load_lds">;
def int_amdgcn_raw_ptr_buffer_load_async_lds : AMDGPURawPtrBufferLoadLDS,
      ClangBuiltin<"__builtin_amdgcn_raw_ptr_buffer_load_async_lds">;


class AMDGPUStructBufferLoadLDS : Intrinsic <
  [],
````
- **L2017 EN**: Declares TableGen def `int_amdgcn_raw_buffer_load_lds`.
  **L2017 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_load_lds`。
- **L2018 EN**: Declares TableGen def `int_amdgcn_raw_buffer_load_async_lds`.
  **L2018 CN**: 声明 TableGen def `int_amdgcn_raw_buffer_load_async_lds`。
- **L2019 EN**: Blank line separating nearby declarations or logic blocks.
  **L2019 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2020 EN**: Declares class `AMDGPURawPtrBufferLoadLDS`.
  **L2020 CN**: 声明 class `AMDGPURawPtrBufferLoadLDS`。
- **L2021 EN**: Continues the surrounding expression or declaration: `Intrinsic <`.
  **L2021 CN**: 继续构造周围的表达式或声明：`Intrinsic <`。
- **L2022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L2022 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L2023 EN**: Continues logic associated with callable symbol `rsrc`.
  **L2023 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L2024 EN**: Continues the surrounding expression or declaration: `LLVMQualPointerType<3>,    // LDS base offset`.
  **L2024 CN**: 继续构造周围的表达式或声明：`LLVMQualPointerType<3>,    // LDS base offset`。
- **L2025 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)`.
  **L2025 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)`。
- **L2026 EN**: Continues logic associated with callable symbol `voffset`.
  **L2026 CN**: 继续与可调用符号 `voffset` 相关的逻辑。
- **L2027 EN**: Continues logic associated with callable symbol `soffset`.
  **L2027 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L2028 EN**: Continues logic associated with callable symbol `offset`.
  **L2028 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L2029 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L2029 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L2030 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L2030 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L2031 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L2031 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L2032 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L2032 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L2033 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L2033 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L2034 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L2034 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L2035 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L2035 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L2036 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWillReturn, IntrArgMemOnly,`.
  **L2036 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWillReturn, IntrArgMemOnly,`。
- **L2037 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`.
  **L2037 CN**: 继续一个多行参数列表、初始化器或聚合项：`ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`。
- **L2038 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`.
  **L2038 CN**: 继续一个多行参数列表、初始化器或聚合项：`WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`。
- **L2039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>,`.
  **L2039 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>,`。
- **L2040 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<6>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;`.
  **L2040 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<6>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;`。
- **L2041 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_load_lds`.
  **L2041 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_load_lds`。
- **L2042 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_raw_ptr_buffer_load_lds">;`.
  **L2042 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_raw_ptr_buffer_load_lds">;`。
- **L2043 EN**: Declares TableGen def `int_amdgcn_raw_ptr_buffer_load_async_lds`.
  **L2043 CN**: 声明 TableGen def `int_amdgcn_raw_ptr_buffer_load_async_lds`。
- **L2044 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_raw_ptr_buffer_load_async_lds">;`.
  **L2044 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_raw_ptr_buffer_load_async_lds">;`。
- **L2045 EN**: Blank line separating nearby declarations or logic blocks.
  **L2045 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2046 EN**: Blank line separating nearby declarations or logic blocks.
  **L2046 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2047 EN**: Declares class `AMDGPUStructBufferLoadLDS`.
  **L2047 CN**: 声明 class `AMDGPUStructBufferLoadLDS`。
- **L2048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L2048 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。

### Lines 2049-2080

````tablegen
  [llvm_v4i32_ty,             // rsrc(SGPR)
   LLVMQualPointerType<3>,    // LDS base offset
   llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)
   llvm_i32_ty,               // vindex(VGPR)
   llvm_i32_ty,               // voffset(VGPR, included in bounds checking and swizzling)
   llvm_i32_ty,               // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty,               // imm offset(imm, included in bounds checking and swizzling)
   llvm_i32_ty],              // auxiliary/cachepolicy(imm):
                              //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                              //                bit 3 = swz, bit 4 = scc (gfx90a)
                              //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                              //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                              //                bit 6 = swz
                              //           all: volatile op (bit 31, stripped at lowering)
  [IntrWillReturn, NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>,
   ImmArg<ArgIndex<7>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;
def int_amdgcn_struct_buffer_load_lds : AMDGPUStructBufferLoadLDS;
def int_amdgcn_struct_buffer_load_async_lds : AMDGPUStructBufferLoadLDS;

class AMDGPUStructPtrBufferLoadLDS :
  Intrinsic <
  [],
  [AMDGPUBufferRsrcTy,        // rsrc(SGPR)
   LLVMQualPointerType<3>,    // LDS base offset
   llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)
   llvm_i32_ty,               // vindex(VGPR)
   llvm_i32_ty,               // voffset(VGPR, included in bounds checking and swizzling)
   llvm_i32_ty,               // soffset(SGPR/imm, excluded from bounds checking and swizzling)
   llvm_i32_ty,               // imm offset(imm, included in bounds checking and swizzling)
   llvm_i32_ty],              // auxiliary/cachepolicy(imm):
                              //                bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),
                              //                bit 3 = swz, bit 4 = scc (gfx90a)
````
- **L2049 EN**: Continues logic associated with callable symbol `rsrc`.
  **L2049 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L2050 EN**: Continues the surrounding expression or declaration: `LLVMQualPointerType<3>,    // LDS base offset`.
  **L2050 CN**: 继续构造周围的表达式或声明：`LLVMQualPointerType<3>,    // LDS base offset`。
- **L2051 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)`.
  **L2051 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)`。
- **L2052 EN**: Continues logic associated with callable symbol `vindex`.
  **L2052 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L2053 EN**: Continues logic associated with callable symbol `voffset`.
  **L2053 CN**: 继续与可调用符号 `voffset` 相关的逻辑。
- **L2054 EN**: Continues logic associated with callable symbol `soffset`.
  **L2054 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L2055 EN**: Continues logic associated with callable symbol `offset`.
  **L2055 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L2056 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L2056 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L2057 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L2057 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L2058 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L2058 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。
- **L2059 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L2059 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L2060 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L2060 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L2061 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L2061 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L2062 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L2062 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L2063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWillReturn, NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>,`.
  **L2063 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWillReturn, NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>,`。
- **L2064 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<7>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;`.
  **L2064 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<7>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;`。
- **L2065 EN**: Declares TableGen def `int_amdgcn_struct_buffer_load_lds`.
  **L2065 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_load_lds`。
- **L2066 EN**: Declares TableGen def `int_amdgcn_struct_buffer_load_async_lds`.
  **L2066 CN**: 声明 TableGen def `int_amdgcn_struct_buffer_load_async_lds`。
- **L2067 EN**: Blank line separating nearby declarations or logic blocks.
  **L2067 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2068 EN**: Declares class `AMDGPUStructPtrBufferLoadLDS`.
  **L2068 CN**: 声明 class `AMDGPUStructPtrBufferLoadLDS`。
- **L2069 EN**: Continues the surrounding expression or declaration: `Intrinsic <`.
  **L2069 CN**: 继续构造周围的表达式或声明：`Intrinsic <`。
- **L2070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L2070 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L2071 EN**: Continues logic associated with callable symbol `rsrc`.
  **L2071 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L2072 EN**: Continues the surrounding expression or declaration: `LLVMQualPointerType<3>,    // LDS base offset`.
  **L2072 CN**: 继续构造周围的表达式或声明：`LLVMQualPointerType<3>,    // LDS base offset`。
- **L2073 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)`.
  **L2073 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,               // Data byte size: 1/2/4 (/12/16 for gfx950)`。
- **L2074 EN**: Continues logic associated with callable symbol `vindex`.
  **L2074 CN**: 继续与可调用符号 `vindex` 相关的逻辑。
- **L2075 EN**: Continues logic associated with callable symbol `voffset`.
  **L2075 CN**: 继续与可调用符号 `voffset` 相关的逻辑。
- **L2076 EN**: Continues logic associated with callable symbol `soffset`.
  **L2076 CN**: 继续与可调用符号 `soffset` 相关的逻辑。
- **L2077 EN**: Continues logic associated with callable symbol `offset`.
  **L2077 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L2078 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L2078 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L2079 EN**: Comment explains nearby logic, invariants, or intent: `bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`.
  **L2079 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 0 = glc, bit 1 = slc, bit 2 = dlc (gfx10/gfx11),`。
- **L2080 EN**: Comment explains nearby logic, invariants, or intent: `bit 3 = swz, bit 4 = scc (gfx90a)`.
  **L2080 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 3 = swz, bit 4 = scc (gfx90a)`。

### Lines 2081-2112

````tablegen
                              //        gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1
                              //        gfx12+: bits [0-2] = th, bits [3-4] = scope,
                              //                bit 6 = swz
                              //           all: volatile op (bit 31, stripped at lowering)
  [IntrWillReturn, IntrArgMemOnly,
   ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,
   WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,
   ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>,
   ImmArg<ArgIndex<7>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;
def int_amdgcn_struct_ptr_buffer_load_lds : AMDGPUStructPtrBufferLoadLDS,
  ClangBuiltin<"__builtin_amdgcn_struct_ptr_buffer_load_lds">;
def int_amdgcn_struct_ptr_buffer_load_async_lds : AMDGPUStructPtrBufferLoadLDS,
  ClangBuiltin<"__builtin_amdgcn_struct_ptr_buffer_load_async_lds">;

def int_amdgcn_s_buffer_prefetch_data : DefaultAttrsIntrinsic <
  [],
  [AMDGPUBufferRsrcTy, // rsrc(SGPR)
   llvm_i32_ty,        // offset (imm)
   llvm_i32_ty],       // len (SGPR/imm)
  [IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<1>>], "", [SDNPMemOperand]>,
  AMDGPURsrcIntrinsic<0>,
  ClangBuiltin<"__builtin_amdgcn_s_buffer_prefetch_data">;

} // defset AMDGPUBufferIntrinsics

// Uses that do not set the done bit should set IntrWriteMem on the
// call site.
def int_amdgcn_exp : DefaultAttrsIntrinsic <[], [
  llvm_i32_ty,       // tgt,
  llvm_i32_ty,       // en
  llvm_any_ty,       // src0 (f32 or i32)
  LLVMMatchType<0>,  // src1
````
- **L2081 EN**: Comment explains nearby logic, invariants, or intent: `gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`.
  **L2081 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942: bit 0 = sc0, bit 1 = nt, bit 3 = swz, bit 4 = sc1`。
- **L2082 EN**: Comment explains nearby logic, invariants, or intent: `gfx12+: bits [0-2] = th, bits [3-4] = scope,`.
  **L2082 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx12+: bits [0-2] = th, bits [3-4] = scope,`。
- **L2083 EN**: Comment explains nearby logic, invariants, or intent: `bit 6 = swz`.
  **L2083 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 6 = swz`。
- **L2084 EN**: Comment explains nearby logic, invariants, or intent: `all: volatile op (bit 31, stripped at lowering)`.
  **L2084 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`all: volatile op (bit 31, stripped at lowering)`。
- **L2085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWillReturn, IntrArgMemOnly,`.
  **L2085 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWillReturn, IntrArgMemOnly,`。
- **L2086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`.
  **L2086 CN**: 继续一个多行参数列表、初始化器或聚合项：`ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,`。
- **L2087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`.
  **L2087 CN**: 继续一个多行参数列表、初始化器或聚合项：`WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<1>>,`。
- **L2088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>,`.
  **L2088 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>,`。
- **L2089 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<7>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;`.
  **L2089 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<7>>, IntrNoCallback, IntrNoFree], "", [SDNPMemOperand]>, AMDGPURsrcIntrinsic<0>;`。
- **L2090 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_load_lds`.
  **L2090 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_load_lds`。
- **L2091 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_struct_ptr_buffer_load_lds">;`.
  **L2091 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_struct_ptr_buffer_load_lds">;`。
- **L2092 EN**: Declares TableGen def `int_amdgcn_struct_ptr_buffer_load_async_lds`.
  **L2092 CN**: 声明 TableGen def `int_amdgcn_struct_ptr_buffer_load_async_lds`。
- **L2093 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_struct_ptr_buffer_load_async_lds">;`.
  **L2093 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_struct_ptr_buffer_load_async_lds">;`。
- **L2094 EN**: Blank line separating nearby declarations or logic blocks.
  **L2094 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2095 EN**: Declares TableGen def `int_amdgcn_s_buffer_prefetch_data`.
  **L2095 CN**: 声明 TableGen def `int_amdgcn_s_buffer_prefetch_data`。
- **L2096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L2096 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L2097 EN**: Continues logic associated with callable symbol `rsrc`.
  **L2097 CN**: 继续与可调用符号 `rsrc` 相关的逻辑。
- **L2098 EN**: Continues logic associated with callable symbol `offset`.
  **L2098 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L2099 EN**: Continues logic associated with callable symbol `len`.
  **L2099 CN**: 继续与可调用符号 `len` 相关的逻辑。
- **L2100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<1>>], "", [SDNPMemOperand]>,`.
  **L2100 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<1>>], "", [SDNPMemOperand]>,`。
- **L2101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AMDGPURsrcIntrinsic<0>,`.
  **L2101 CN**: 继续一个多行参数列表、初始化器或聚合项：`AMDGPURsrcIntrinsic<0>,`。
- **L2102 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_s_buffer_prefetch_data">;`.
  **L2102 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_s_buffer_prefetch_data">;`。
- **L2103 EN**: Blank line separating nearby declarations or logic blocks.
  **L2103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2104 EN**: Continues the surrounding expression or declaration: `} // defset AMDGPUBufferIntrinsics`.
  **L2104 CN**: 继续构造周围的表达式或声明：`} // defset AMDGPUBufferIntrinsics`。
- **L2105 EN**: Blank line separating nearby declarations or logic blocks.
  **L2105 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2106 EN**: Comment explains nearby logic, invariants, or intent: `Uses that do not set the done bit should set IntrWriteMem on the`.
  **L2106 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Uses that do not set the done bit should set IntrWriteMem on the`。
- **L2107 EN**: Comment explains nearby logic, invariants, or intent: `call site.`.
  **L2107 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`call site.`。
- **L2108 EN**: Declares TableGen def `int_amdgcn_exp`.
  **L2108 CN**: 声明 TableGen def `int_amdgcn_exp`。
- **L2109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,       // tgt,`.
  **L2109 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,       // tgt,`。
- **L2110 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,       // en`.
  **L2110 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,       // en`。
- **L2111 EN**: Continues logic associated with callable symbol `src0`.
  **L2111 CN**: 继续与可调用符号 `src0` 相关的逻辑。
- **L2112 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,  // src1`.
  **L2112 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,  // src1`。

### Lines 2113-2144

````tablegen
  LLVMMatchType<0>,  // src2
  LLVMMatchType<0>,  // src3
  llvm_i1_ty,        // done
  llvm_i1_ty         // vm (ignored on GFX11+)
  ],
  [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<6>>,
   ImmArg<ArgIndex<7>>, IntrWriteMem, IntrInaccessibleMemOnly]
>;

// exp with row_en bit set. Only supported on GFX11+.
def int_amdgcn_exp_row : DefaultAttrsIntrinsic <[], [
  llvm_i32_ty,       // tgt,
  llvm_i32_ty,       // en
  llvm_any_ty,       // src0 (f32 or i32)
  LLVMMatchType<0>,  // src1
  LLVMMatchType<0>,  // src2
  LLVMMatchType<0>,  // src3
  llvm_i1_ty,        // done
  llvm_i32_ty],      // row number
  [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<6>>,
   IntrWriteMem, IntrInaccessibleMemOnly]
>;

// exp with compr bit set. Not supported on GFX11+.
def int_amdgcn_exp_compr : DefaultAttrsIntrinsic <[], [
  llvm_i32_ty,       // tgt,
  llvm_i32_ty,       // en
  llvm_anyvector_ty, // src0 (v2f16 or v2i16)
  LLVMMatchType<0>,  // src1
  llvm_i1_ty,        // done
  llvm_i1_ty],       // vm
  [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<4>>,
````
- **L2113 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,  // src2`.
  **L2113 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,  // src2`。
- **L2114 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,  // src3`.
  **L2114 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,  // src3`。
- **L2115 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,        // done`.
  **L2115 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,        // done`。
- **L2116 EN**: Continues logic associated with callable symbol `vm`.
  **L2116 CN**: 继续与可调用符号 `vm` 相关的逻辑。
- **L2117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L2117 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L2118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<6>>,`.
  **L2118 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<6>>,`。
- **L2119 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<7>>, IntrWriteMem, IntrInaccessibleMemOnly]`.
  **L2119 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<7>>, IntrWriteMem, IntrInaccessibleMemOnly]`。
- **L2120 EN**: Executes a standalone statement or declaration: `>;`.
  **L2120 CN**: 执行一条独立语句或声明：`>;`。
- **L2121 EN**: Blank line separating nearby declarations or logic blocks.
  **L2121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2122 EN**: Comment explains nearby logic, invariants, or intent: `exp with row_en bit set. Only supported on GFX11+.`.
  **L2122 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`exp with row_en bit set. Only supported on GFX11+.`。
- **L2123 EN**: Declares TableGen def `int_amdgcn_exp_row`.
  **L2123 CN**: 声明 TableGen def `int_amdgcn_exp_row`。
- **L2124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,       // tgt,`.
  **L2124 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,       // tgt,`。
- **L2125 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,       // en`.
  **L2125 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,       // en`。
- **L2126 EN**: Continues logic associated with callable symbol `src0`.
  **L2126 CN**: 继续与可调用符号 `src0` 相关的逻辑。
- **L2127 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,  // src1`.
  **L2127 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,  // src1`。
- **L2128 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,  // src2`.
  **L2128 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,  // src2`。
- **L2129 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,  // src3`.
  **L2129 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,  // src3`。
- **L2130 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,        // done`.
  **L2130 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,        // done`。
- **L2131 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],      // row number`.
  **L2131 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],      // row number`。
- **L2132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<6>>,`.
  **L2132 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<6>>,`。
- **L2133 EN**: Continues the surrounding expression or declaration: `IntrWriteMem, IntrInaccessibleMemOnly]`.
  **L2133 CN**: 继续构造周围的表达式或声明：`IntrWriteMem, IntrInaccessibleMemOnly]`。
- **L2134 EN**: Executes a standalone statement or declaration: `>;`.
  **L2134 CN**: 执行一条独立语句或声明：`>;`。
- **L2135 EN**: Blank line separating nearby declarations or logic blocks.
  **L2135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2136 EN**: Comment explains nearby logic, invariants, or intent: `exp with compr bit set. Not supported on GFX11+.`.
  **L2136 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`exp with compr bit set. Not supported on GFX11+.`。
- **L2137 EN**: Declares TableGen def `int_amdgcn_exp_compr`.
  **L2137 CN**: 声明 TableGen def `int_amdgcn_exp_compr`。
- **L2138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,       // tgt,`.
  **L2138 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,       // tgt,`。
- **L2139 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,       // en`.
  **L2139 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,       // en`。
- **L2140 EN**: Continues logic associated with callable symbol `src0`.
  **L2140 CN**: 继续与可调用符号 `src0` 相关的逻辑。
- **L2141 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,  // src1`.
  **L2141 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,  // src1`。
- **L2142 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,        // done`.
  **L2142 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,        // done`。
- **L2143 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty],       // vm`.
  **L2143 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty],       // vm`。
- **L2144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<4>>,`.
  **L2144 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<4>>,`。

### Lines 2145-2176

````tablegen
   ImmArg<ArgIndex<5>>, IntrWriteMem, IntrInaccessibleMemOnly]
>;

def int_amdgcn_buffer_wbinvl1_sc :
  ClangBuiltin<"__builtin_amdgcn_buffer_wbinvl1_sc">,
  DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;

def int_amdgcn_buffer_wbinvl1 :
  ClangBuiltin<"__builtin_amdgcn_buffer_wbinvl1">,
  DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;

def int_amdgcn_s_dcache_inv :
  ClangBuiltin<"__builtin_amdgcn_s_dcache_inv">,
  DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;

def int_amdgcn_s_memtime :
  ClangBuiltin<"__builtin_amdgcn_s_memtime">,
  DefaultAttrsIntrinsic<[llvm_i64_ty], [], [IntrNoMem, IntrHasSideEffects]>;

def int_amdgcn_s_sleep :
  ClangBuiltin<"__builtin_amdgcn_s_sleep">,
  DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,
                                IntrHasSideEffects]> {
}

def int_amdgcn_s_sleep_var
    : ClangBuiltin<"__builtin_amdgcn_s_sleep_var">,
      Intrinsic<[], [llvm_i32_ty],
                [IntrNoMem, IntrHasSideEffects, IntrWillReturn]> {
}

def int_amdgcn_s_nop :
````
- **L2145 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<5>>, IntrWriteMem, IntrInaccessibleMemOnly]`.
  **L2145 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<5>>, IntrWriteMem, IntrInaccessibleMemOnly]`。
- **L2146 EN**: Executes a standalone statement or declaration: `>;`.
  **L2146 CN**: 执行一条独立语句或声明：`>;`。
- **L2147 EN**: Blank line separating nearby declarations or logic blocks.
  **L2147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2148 EN**: Declares TableGen def `int_amdgcn_buffer_wbinvl1_sc`.
  **L2148 CN**: 声明 TableGen def `int_amdgcn_buffer_wbinvl1_sc`。
- **L2149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_buffer_wbinvl1_sc">,`.
  **L2149 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_buffer_wbinvl1_sc">,`。
- **L2150 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L2150 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L2151 EN**: Blank line separating nearby declarations or logic blocks.
  **L2151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2152 EN**: Declares TableGen def `int_amdgcn_buffer_wbinvl1`.
  **L2152 CN**: 声明 TableGen def `int_amdgcn_buffer_wbinvl1`。
- **L2153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_buffer_wbinvl1">,`.
  **L2153 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_buffer_wbinvl1">,`。
- **L2154 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L2154 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L2155 EN**: Blank line separating nearby declarations or logic blocks.
  **L2155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2156 EN**: Declares TableGen def `int_amdgcn_s_dcache_inv`.
  **L2156 CN**: 声明 TableGen def `int_amdgcn_s_dcache_inv`。
- **L2157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_dcache_inv">,`.
  **L2157 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_dcache_inv">,`。
- **L2158 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L2158 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L2159 EN**: Blank line separating nearby declarations or logic blocks.
  **L2159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2160 EN**: Declares TableGen def `int_amdgcn_s_memtime`.
  **L2160 CN**: 声明 TableGen def `int_amdgcn_s_memtime`。
- **L2161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_memtime">,`.
  **L2161 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_memtime">,`。
- **L2162 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i64_ty], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L2162 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i64_ty], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L2163 EN**: Blank line separating nearby declarations or logic blocks.
  **L2163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2164 EN**: Declares TableGen def `int_amdgcn_s_sleep`.
  **L2164 CN**: 声明 TableGen def `int_amdgcn_s_sleep`。
- **L2165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_sleep">,`.
  **L2165 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_sleep">,`。
- **L2166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`.
  **L2166 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`。
- **L2167 EN**: Continues the surrounding expression or declaration: `IntrHasSideEffects]> {`.
  **L2167 CN**: 继续构造周围的表达式或声明：`IntrHasSideEffects]> {`。
- **L2168 EN**: Closes the current lexical scope or compound statement.
  **L2168 CN**: 结束当前词法作用域或复合语句块。
- **L2169 EN**: Blank line separating nearby declarations or logic blocks.
  **L2169 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2170 EN**: Declares TableGen def `int_amdgcn_s_sleep_var`.
  **L2170 CN**: 声明 TableGen def `int_amdgcn_s_sleep_var`。
- **L2171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_amdgcn_s_sleep_var">,`.
  **L2171 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_amdgcn_s_sleep_var">,`。
- **L2172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i32_ty],`.
  **L2172 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i32_ty],`。
- **L2173 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrHasSideEffects, IntrWillReturn]> {`.
  **L2173 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrHasSideEffects, IntrWillReturn]> {`。
- **L2174 EN**: Closes the current lexical scope or compound statement.
  **L2174 CN**: 结束当前词法作用域或复合语句块。
- **L2175 EN**: Blank line separating nearby declarations or logic blocks.
  **L2175 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2176 EN**: Declares TableGen def `int_amdgcn_s_nop`.
  **L2176 CN**: 声明 TableGen def `int_amdgcn_s_nop`。

### Lines 2177-2208

````tablegen
  DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,
                                IntrHasSideEffects]> {
}

def int_amdgcn_s_incperflevel :
  ClangBuiltin<"__builtin_amdgcn_s_incperflevel">,
  DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,
                                IntrHasSideEffects]> {
}

def int_amdgcn_s_decperflevel :
  ClangBuiltin<"__builtin_amdgcn_s_decperflevel">,
  DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,
                                IntrHasSideEffects]> {
}

def int_amdgcn_s_sethalt :
  DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,
                                IntrHasSideEffects]>;

def int_amdgcn_s_setprio :
  ClangBuiltin<"__builtin_amdgcn_s_setprio">,
  DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,
                                IntrHasSideEffects]>;

def int_amdgcn_s_setprio_inc_wg :
  ClangBuiltin<"__builtin_amdgcn_s_setprio_inc_wg">,
  DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,
                                IntrHasSideEffects]>;

def int_amdgcn_s_ttracedata :
  ClangBuiltin<"__builtin_amdgcn_s_ttracedata">,
````
- **L2177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`.
  **L2177 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`。
- **L2178 EN**: Continues the surrounding expression or declaration: `IntrHasSideEffects]> {`.
  **L2178 CN**: 继续构造周围的表达式或声明：`IntrHasSideEffects]> {`。
- **L2179 EN**: Closes the current lexical scope or compound statement.
  **L2179 CN**: 结束当前词法作用域或复合语句块。
- **L2180 EN**: Blank line separating nearby declarations or logic blocks.
  **L2180 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2181 EN**: Declares TableGen def `int_amdgcn_s_incperflevel`.
  **L2181 CN**: 声明 TableGen def `int_amdgcn_s_incperflevel`。
- **L2182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_incperflevel">,`.
  **L2182 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_incperflevel">,`。
- **L2183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`.
  **L2183 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`。
- **L2184 EN**: Continues the surrounding expression or declaration: `IntrHasSideEffects]> {`.
  **L2184 CN**: 继续构造周围的表达式或声明：`IntrHasSideEffects]> {`。
- **L2185 EN**: Closes the current lexical scope or compound statement.
  **L2185 CN**: 结束当前词法作用域或复合语句块。
- **L2186 EN**: Blank line separating nearby declarations or logic blocks.
  **L2186 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2187 EN**: Declares TableGen def `int_amdgcn_s_decperflevel`.
  **L2187 CN**: 声明 TableGen def `int_amdgcn_s_decperflevel`。
- **L2188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_decperflevel">,`.
  **L2188 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_decperflevel">,`。
- **L2189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`.
  **L2189 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`。
- **L2190 EN**: Continues the surrounding expression or declaration: `IntrHasSideEffects]> {`.
  **L2190 CN**: 继续构造周围的表达式或声明：`IntrHasSideEffects]> {`。
- **L2191 EN**: Closes the current lexical scope or compound statement.
  **L2191 CN**: 结束当前词法作用域或复合语句块。
- **L2192 EN**: Blank line separating nearby declarations or logic blocks.
  **L2192 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2193 EN**: Declares TableGen def `int_amdgcn_s_sethalt`.
  **L2193 CN**: 声明 TableGen def `int_amdgcn_s_sethalt`。
- **L2194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`.
  **L2194 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`。
- **L2195 EN**: Executes a standalone statement or declaration: `IntrHasSideEffects]>;`.
  **L2195 CN**: 执行一条独立语句或声明：`IntrHasSideEffects]>;`。
- **L2196 EN**: Blank line separating nearby declarations or logic blocks.
  **L2196 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2197 EN**: Declares TableGen def `int_amdgcn_s_setprio`.
  **L2197 CN**: 声明 TableGen def `int_amdgcn_s_setprio`。
- **L2198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_setprio">,`.
  **L2198 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_setprio">,`。
- **L2199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`.
  **L2199 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`。
- **L2200 EN**: Executes a standalone statement or declaration: `IntrHasSideEffects]>;`.
  **L2200 CN**: 执行一条独立语句或声明：`IntrHasSideEffects]>;`。
- **L2201 EN**: Blank line separating nearby declarations or logic blocks.
  **L2201 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2202 EN**: Declares TableGen def `int_amdgcn_s_setprio_inc_wg`.
  **L2202 CN**: 声明 TableGen def `int_amdgcn_s_setprio_inc_wg`。
- **L2203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_setprio_inc_wg">,`.
  **L2203 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_setprio_inc_wg">,`。
- **L2204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`.
  **L2204 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`。
- **L2205 EN**: Executes a standalone statement or declaration: `IntrHasSideEffects]>;`.
  **L2205 CN**: 执行一条独立语句或声明：`IntrHasSideEffects]>;`。
- **L2206 EN**: Blank line separating nearby declarations or logic blocks.
  **L2206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2207 EN**: Declares TableGen def `int_amdgcn_s_ttracedata`.
  **L2207 CN**: 声明 TableGen def `int_amdgcn_s_ttracedata`。
- **L2208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_ttracedata">,`.
  **L2208 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_ttracedata">,`。

### Lines 2209-2240

````tablegen
  DefaultAttrsIntrinsic<[], [llvm_i32_ty],
                        [IntrNoMem, IntrHasSideEffects]>;

def int_amdgcn_s_ttracedata_imm :
  ClangBuiltin<"__builtin_amdgcn_s_ttracedata_imm">,
  DefaultAttrsIntrinsic<[], [llvm_i16_ty],
                        [IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<0>>]>;

// This is IntrHasSideEffects so it can be used to read cycle counters.
def int_amdgcn_s_getreg :
  ClangBuiltin<"__builtin_amdgcn_s_getreg">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty],
  [IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<0>>]
>;

// Note this can be used to set FP environment properties that are
// unsafe to change in non-strictfp functions. The register properties
// available (and value required to access them) may differ per
// subtarget. llvm.amdgcn.s.setreg(hwmode, value)
def int_amdgcn_s_setreg :
  ClangBuiltin<"__builtin_amdgcn_s_setreg">,
  DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_i32_ty],
  [IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<0>>]
>;

// int_amdgcn_s_getpc is provided to allow a specific style of position
// independent code to determine the high part of its address when it is
// known (through convention) that the code and any data of interest does
// not cross a 4Gb address boundary. Use for any other purpose may not
// produce the desired results as optimizations may cause code movement,
// especially as we explicitly use IntrNoMem to allow optimizations.
// This intrinsic always returns PC sign-extended from 48 bits even if the
````
- **L2209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty],`.
  **L2209 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty],`。
- **L2210 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L2210 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L2211 EN**: Blank line separating nearby declarations or logic blocks.
  **L2211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2212 EN**: Declares TableGen def `int_amdgcn_s_ttracedata_imm`.
  **L2212 CN**: 声明 TableGen def `int_amdgcn_s_ttracedata_imm`。
- **L2213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_ttracedata_imm">,`.
  **L2213 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_ttracedata_imm">,`。
- **L2214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i16_ty],`.
  **L2214 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i16_ty],`。
- **L2215 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<0>>]>;`.
  **L2215 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<0>>]>;`。
- **L2216 EN**: Blank line separating nearby declarations or logic blocks.
  **L2216 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2217 EN**: Comment explains nearby logic, invariants, or intent: `This is IntrHasSideEffects so it can be used to read cycle counters.`.
  **L2217 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is IntrHasSideEffects so it can be used to read cycle counters.`。
- **L2218 EN**: Declares TableGen def `int_amdgcn_s_getreg`.
  **L2218 CN**: 声明 TableGen def `int_amdgcn_s_getreg`。
- **L2219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_getreg">,`.
  **L2219 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_getreg">,`。
- **L2220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty],`.
  **L2220 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty],`。
- **L2221 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<0>>]`.
  **L2221 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<0>>]`。
- **L2222 EN**: Executes a standalone statement or declaration: `>;`.
  **L2222 CN**: 执行一条独立语句或声明：`>;`。
- **L2223 EN**: Blank line separating nearby declarations or logic blocks.
  **L2223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2224 EN**: Comment explains nearby logic, invariants, or intent: `Note this can be used to set FP environment properties that are`.
  **L2224 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note this can be used to set FP environment properties that are`。
- **L2225 EN**: Comment explains nearby logic, invariants, or intent: `unsafe to change in non-strictfp functions. The register properties`.
  **L2225 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`unsafe to change in non-strictfp functions. The register properties`。
- **L2226 EN**: Comment explains nearby logic, invariants, or intent: `available (and value required to access them) may differ per`.
  **L2226 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`available (and value required to access them) may differ per`。
- **L2227 EN**: Comment explains nearby logic, invariants, or intent: `subtarget. llvm.amdgcn.s.setreg(hwmode, value)`.
  **L2227 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`subtarget. llvm.amdgcn.s.setreg(hwmode, value)`。
- **L2228 EN**: Declares TableGen def `int_amdgcn_s_setreg`.
  **L2228 CN**: 声明 TableGen def `int_amdgcn_s_setreg`。
- **L2229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_setreg">,`.
  **L2229 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_setreg">,`。
- **L2230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_i32_ty],`.
  **L2230 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_i32_ty],`。
- **L2231 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<0>>]`.
  **L2231 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<0>>]`。
- **L2232 EN**: Executes a standalone statement or declaration: `>;`.
  **L2232 CN**: 执行一条独立语句或声明：`>;`。
- **L2233 EN**: Blank line separating nearby declarations or logic blocks.
  **L2233 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2234 EN**: Comment explains nearby logic, invariants, or intent: `int_amdgcn_s_getpc is provided to allow a specific style of position`.
  **L2234 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`int_amdgcn_s_getpc is provided to allow a specific style of position`。
- **L2235 EN**: Comment explains nearby logic, invariants, or intent: `independent code to determine the high part of its address when it is`.
  **L2235 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`independent code to determine the high part of its address when it is`。
- **L2236 EN**: Comment explains nearby logic, invariants, or intent: `known (through convention) that the code and any data of interest does`.
  **L2236 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`known (through convention) that the code and any data of interest does`。
- **L2237 EN**: Comment explains nearby logic, invariants, or intent: `not cross a 4Gb address boundary. Use for any other purpose may not`.
  **L2237 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`not cross a 4Gb address boundary. Use for any other purpose may not`。
- **L2238 EN**: Comment explains nearby logic, invariants, or intent: `produce the desired results as optimizations may cause code movement,`.
  **L2238 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`produce the desired results as optimizations may cause code movement,`。
- **L2239 EN**: Comment explains nearby logic, invariants, or intent: `especially as we explicitly use IntrNoMem to allow optimizations.`.
  **L2239 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`especially as we explicitly use IntrNoMem to allow optimizations.`。
- **L2240 EN**: Comment explains nearby logic, invariants, or intent: `This intrinsic always returns PC sign-extended from 48 bits even if the`.
  **L2240 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This intrinsic always returns PC sign-extended from 48 bits even if the`。

### Lines 2241-2272

````tablegen
// s_getpc_b64 instruction returns a zero-extended value.
def int_amdgcn_s_getpc :
  ClangBuiltin<"__builtin_amdgcn_s_getpc">,
  DefaultAttrsIntrinsic<[llvm_i64_ty], [], [NoUndef<RetIndex>, IntrNoMem,
                                IntrSpeculatable]>;

// __builtin_amdgcn_interp_mov <param>, <attr_chan>, <attr>, <m0>
// param values: 0 = P10, 1 = P20, 2 = P0
def int_amdgcn_interp_mov :
  ClangBuiltin<"__builtin_amdgcn_interp_mov">,
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrSpeculatable,
              ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;

// __builtin_amdgcn_interp_p1 <i>, <attr_chan>, <attr>, <m0>
// This intrinsic reads from lds, but the memory values are constant,
// so it behaves like IntrNoMem.
def int_amdgcn_interp_p1 :
  ClangBuiltin<"__builtin_amdgcn_interp_p1">,
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrSpeculatable,
             ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;

// __builtin_amdgcn_interp_p2 <p1>, <j>, <attr_chan>, <attr>, <m0>
def int_amdgcn_interp_p2 :
  ClangBuiltin<"__builtin_amdgcn_interp_p2">,
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrSpeculatable,
             ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;
````
- **L2241 EN**: Comment explains nearby logic, invariants, or intent: `s_getpc_b64 instruction returns a zero-extended value.`.
  **L2241 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`s_getpc_b64 instruction returns a zero-extended value.`。
- **L2242 EN**: Declares TableGen def `int_amdgcn_s_getpc`.
  **L2242 CN**: 声明 TableGen def `int_amdgcn_s_getpc`。
- **L2243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_getpc">,`.
  **L2243 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_getpc">,`。
- **L2244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [], [NoUndef<RetIndex>, IntrNoMem,`.
  **L2244 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [], [NoUndef<RetIndex>, IntrNoMem,`。
- **L2245 EN**: Executes a standalone statement or declaration: `IntrSpeculatable]>;`.
  **L2245 CN**: 执行一条独立语句或声明：`IntrSpeculatable]>;`。
- **L2246 EN**: Blank line separating nearby declarations or logic blocks.
  **L2246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2247 EN**: Comment explains nearby logic, invariants, or intent: `__builtin_amdgcn_interp_mov <param>, <attr_chan>, <attr>, <m0>`.
  **L2247 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__builtin_amdgcn_interp_mov <param>, <attr_chan>, <attr>, <m0>`。
- **L2248 EN**: Comment explains nearby logic, invariants, or intent: `param values: 0 = P10, 1 = P20, 2 = P0`.
  **L2248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`param values: 0 = P10, 1 = P20, 2 = P0`。
- **L2249 EN**: Declares TableGen def `int_amdgcn_interp_mov`.
  **L2249 CN**: 声明 TableGen def `int_amdgcn_interp_mov`。
- **L2250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_interp_mov">,`.
  **L2250 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_interp_mov">,`。
- **L2251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2251 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L2252 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L2253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2253 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2254 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`.
  **L2254 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`。
- **L2255 EN**: Blank line separating nearby declarations or logic blocks.
  **L2255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2256 EN**: Comment explains nearby logic, invariants, or intent: `__builtin_amdgcn_interp_p1 <i>, <attr_chan>, <attr>, <m0>`.
  **L2256 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__builtin_amdgcn_interp_p1 <i>, <attr_chan>, <attr>, <m0>`。
- **L2257 EN**: Comment explains nearby logic, invariants, or intent: `This intrinsic reads from lds, but the memory values are constant,`.
  **L2257 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This intrinsic reads from lds, but the memory values are constant,`。
- **L2258 EN**: Comment explains nearby logic, invariants, or intent: `so it behaves like IntrNoMem.`.
  **L2258 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`so it behaves like IntrNoMem.`。
- **L2259 EN**: Declares TableGen def `int_amdgcn_interp_p1`.
  **L2259 CN**: 声明 TableGen def `int_amdgcn_interp_p1`。
- **L2260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_interp_p1">,`.
  **L2260 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_interp_p1">,`。
- **L2261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2261 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L2262 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L2263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2263 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2264 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`.
  **L2264 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`。
- **L2265 EN**: Blank line separating nearby declarations or logic blocks.
  **L2265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2266 EN**: Comment explains nearby logic, invariants, or intent: `__builtin_amdgcn_interp_p2 <p1>, <j>, <attr_chan>, <attr>, <m0>`.
  **L2266 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__builtin_amdgcn_interp_p2 <p1>, <j>, <attr_chan>, <attr>, <m0>`。
- **L2267 EN**: Declares TableGen def `int_amdgcn_interp_p2`.
  **L2267 CN**: 声明 TableGen def `int_amdgcn_interp_p2`。
- **L2268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_interp_p2">,`.
  **L2268 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_interp_p2">,`。
- **L2269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2269 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L2270 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L2271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2271 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2272 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L2272 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。

### Lines 2273-2304

````tablegen
          // See int_amdgcn_v_interp_p1 for why this is IntrNoMem.

// __builtin_amdgcn_interp_p1_f16 <i>, <attr_chan>, <attr>, <high>, <m0>
// high selects whether high or low 16-bits are loaded from LDS
def int_amdgcn_interp_p1_f16 :
  ClangBuiltin<"__builtin_amdgcn_interp_p1_f16">,
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i32_ty],
            [IntrNoMem, IntrSpeculatable,
             ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;

// __builtin_amdgcn_interp_p2_f16 <p1>, <j>, <attr_chan>, <attr>, <high>, <m0>
// high selects whether high or low 16-bits are loaded from LDS
def int_amdgcn_interp_p2_f16 :
  ClangBuiltin<"__builtin_amdgcn_interp_p2_f16">,
  DefaultAttrsIntrinsic<[llvm_half_ty],
            [llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i32_ty],
            [IntrNoMem, IntrSpeculatable,
             ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;

// llvm.amdgcn.lds.direct.load <m0>
// The input argument is m0, which contains a packed combination of address
// offset and flags describing the data type.
def int_amdgcn_lds_direct_load :
  DefaultAttrsIntrinsic<[llvm_any_ty], // overloaded for types u8, u16, i32/f32, i8, i16
            [llvm_i32_ty],
            [IntrReadMem, IntrSpeculatable]>;

// llvm.amdgcn.lds.param.load <attr_chan>, <attr>, <m0>
// Like interp intrinsics, this reads from lds, but the memory values are constant,
// so it behaves like IntrNoMem.
def int_amdgcn_lds_param_load :
````
- **L2273 EN**: Comment explains nearby logic, invariants, or intent: `See int_amdgcn_v_interp_p1 for why this is IntrNoMem.`.
  **L2273 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See int_amdgcn_v_interp_p1 for why this is IntrNoMem.`。
- **L2274 EN**: Blank line separating nearby declarations or logic blocks.
  **L2274 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2275 EN**: Comment explains nearby logic, invariants, or intent: `__builtin_amdgcn_interp_p1_f16 <i>, <attr_chan>, <attr>, <high>, <m0>`.
  **L2275 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__builtin_amdgcn_interp_p1_f16 <i>, <attr_chan>, <attr>, <high>, <m0>`。
- **L2276 EN**: Comment explains nearby logic, invariants, or intent: `high selects whether high or low 16-bits are loaded from LDS`.
  **L2276 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`high selects whether high or low 16-bits are loaded from LDS`。
- **L2277 EN**: Declares TableGen def `int_amdgcn_interp_p1_f16`.
  **L2277 CN**: 声明 TableGen def `int_amdgcn_interp_p1_f16`。
- **L2278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_interp_p1_f16">,`.
  **L2278 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_interp_p1_f16">,`。
- **L2279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2279 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i32_ty],`.
  **L2280 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i32_ty],`。
- **L2281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2281 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2282 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L2282 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L2283 EN**: Blank line separating nearby declarations or logic blocks.
  **L2283 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2284 EN**: Comment explains nearby logic, invariants, or intent: `__builtin_amdgcn_interp_p2_f16 <p1>, <j>, <attr_chan>, <attr>, <high>, <m0>`.
  **L2284 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__builtin_amdgcn_interp_p2_f16 <p1>, <j>, <attr_chan>, <attr>, <high>, <m0>`。
- **L2285 EN**: Comment explains nearby logic, invariants, or intent: `high selects whether high or low 16-bits are loaded from LDS`.
  **L2285 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`high selects whether high or low 16-bits are loaded from LDS`。
- **L2286 EN**: Declares TableGen def `int_amdgcn_interp_p2_f16`.
  **L2286 CN**: 声明 TableGen def `int_amdgcn_interp_p2_f16`。
- **L2287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_interp_p2_f16">,`.
  **L2287 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_interp_p2_f16">,`。
- **L2288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_half_ty],`.
  **L2288 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_half_ty],`。
- **L2289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i32_ty],`.
  **L2289 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i32_ty],`。
- **L2290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2290 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2291 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`.
  **L2291 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`。
- **L2292 EN**: Blank line separating nearby declarations or logic blocks.
  **L2292 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2293 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.lds.direct.load <m0>`.
  **L2293 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.lds.direct.load <m0>`。
- **L2294 EN**: Comment explains nearby logic, invariants, or intent: `The input argument is m0, which contains a packed combination of address`.
  **L2294 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The input argument is m0, which contains a packed combination of address`。
- **L2295 EN**: Comment explains nearby logic, invariants, or intent: `offset and flags describing the data type.`.
  **L2295 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`offset and flags describing the data type.`。
- **L2296 EN**: Declares TableGen def `int_amdgcn_lds_direct_load`.
  **L2296 CN**: 声明 TableGen def `int_amdgcn_lds_direct_load`。
- **L2297 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<[llvm_any_ty], // overloaded for types u8, u16, i32/f32, i8, i16`.
  **L2297 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<[llvm_any_ty], // overloaded for types u8, u16, i32/f32, i8, i16`。
- **L2298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L2298 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L2299 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrSpeculatable]>;`.
  **L2299 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrSpeculatable]>;`。
- **L2300 EN**: Blank line separating nearby declarations or logic blocks.
  **L2300 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2301 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.lds.param.load <attr_chan>, <attr>, <m0>`.
  **L2301 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.lds.param.load <attr_chan>, <attr>, <m0>`。
- **L2302 EN**: Comment explains nearby logic, invariants, or intent: `Like interp intrinsics, this reads from lds, but the memory values are constant,`.
  **L2302 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Like interp intrinsics, this reads from lds, but the memory values are constant,`。
- **L2303 EN**: Comment explains nearby logic, invariants, or intent: `so it behaves like IntrNoMem.`.
  **L2303 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`so it behaves like IntrNoMem.`。
- **L2304 EN**: Declares TableGen def `int_amdgcn_lds_param_load`.
  **L2304 CN**: 声明 TableGen def `int_amdgcn_lds_param_load`。

### Lines 2305-2336

````tablegen
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrSpeculatable,
             ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;

// llvm.amdgcn.interp.inreg.p10 <p>, <i>, <p0>
def int_amdgcn_interp_inreg_p10 :
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_float_ty, llvm_float_ty, llvm_float_ty],
            [IntrNoMem, IntrSpeculatable]>;

// llvm.amdgcn.interp.inreg.p2 <p>, <j>, <tmp>
def int_amdgcn_interp_inreg_p2 :
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_float_ty, llvm_float_ty, llvm_float_ty],
            [IntrNoMem, IntrSpeculatable]>;

// llvm.amdgcn.interp.inreg.p10.f16 <p>, <i>, <p0>, <high>
// high selects whether high or low 16-bits are used for p and p0 operands
def int_amdgcn_interp_inreg_p10_f16:
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],
            [IntrNoMem, IntrSpeculatable,
             ImmArg<ArgIndex<3>>]>;

// llvm.amdgcn.interp.inreg.p2.f16 <p>, <j>, <tmp>, <high>
// high selects whether high or low 16-bits are used for p operand
def int_amdgcn_interp_inreg_p2_f16 :
  DefaultAttrsIntrinsic<[llvm_half_ty],
            [llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],
            [IntrNoMem, IntrSpeculatable,
             ImmArg<ArgIndex<3>>]>;
````
- **L2305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2305 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L2306 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L2307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2307 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2308 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L2308 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L2309 EN**: Blank line separating nearby declarations or logic blocks.
  **L2309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2310 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.interp.inreg.p10 <p>, <i>, <p0>`.
  **L2310 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.interp.inreg.p10 <p>, <i>, <p0>`。
- **L2311 EN**: Declares TableGen def `int_amdgcn_interp_inreg_p10`.
  **L2311 CN**: 声明 TableGen def `int_amdgcn_interp_inreg_p10`。
- **L2312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2312 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty],`.
  **L2313 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty],`。
- **L2314 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2314 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2315 EN**: Blank line separating nearby declarations or logic blocks.
  **L2315 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2316 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.interp.inreg.p2 <p>, <j>, <tmp>`.
  **L2316 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.interp.inreg.p2 <p>, <j>, <tmp>`。
- **L2317 EN**: Declares TableGen def `int_amdgcn_interp_inreg_p2`.
  **L2317 CN**: 声明 TableGen def `int_amdgcn_interp_inreg_p2`。
- **L2318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2318 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty],`.
  **L2319 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty],`。
- **L2320 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2320 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2321 EN**: Blank line separating nearby declarations or logic blocks.
  **L2321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2322 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.interp.inreg.p10.f16 <p>, <i>, <p0>, <high>`.
  **L2322 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.interp.inreg.p10.f16 <p>, <i>, <p0>, <high>`。
- **L2323 EN**: Comment explains nearby logic, invariants, or intent: `high selects whether high or low 16-bits are used for p and p0 operands`.
  **L2323 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`high selects whether high or low 16-bits are used for p and p0 operands`。
- **L2324 EN**: Declares TableGen def `int_amdgcn_interp_inreg_p10_f16`.
  **L2324 CN**: 声明 TableGen def `int_amdgcn_interp_inreg_p10_f16`。
- **L2325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2325 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],`.
  **L2326 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],`。
- **L2327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2327 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2328 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>]>;`.
  **L2328 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>]>;`。
- **L2329 EN**: Blank line separating nearby declarations or logic blocks.
  **L2329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2330 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.interp.inreg.p2.f16 <p>, <j>, <tmp>, <high>`.
  **L2330 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.interp.inreg.p2.f16 <p>, <j>, <tmp>, <high>`。
- **L2331 EN**: Comment explains nearby logic, invariants, or intent: `high selects whether high or low 16-bits are used for p operand`.
  **L2331 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`high selects whether high or low 16-bits are used for p operand`。
- **L2332 EN**: Declares TableGen def `int_amdgcn_interp_inreg_p2_f16`.
  **L2332 CN**: 声明 TableGen def `int_amdgcn_interp_inreg_p2_f16`。
- **L2333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_half_ty],`.
  **L2333 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_half_ty],`。
- **L2334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],`.
  **L2334 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],`。
- **L2335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2335 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2336 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>]>;`.
  **L2336 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>]>;`。

### Lines 2337-2368

````tablegen

// llvm.amdgcn.interp.p10.rtz.f16 <p>, <i>, <p0>, <high>
// gfx11+ fp16 interpolation intrinsic, with round-toward-zero rounding mode.
// high selects whether high or low 16-bits are used for p and p0 operands
def int_amdgcn_interp_p10_rtz_f16:
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],
            [IntrNoMem, IntrSpeculatable,
             ImmArg<ArgIndex<3>>]>;

// llvm.amdgcn.interp.p2.rtz.f16 <p>, <j>, <tmp>, <high>
// gfx11+ fp16 interpolation intrinsic, with round-toward-zero rounding mode.
// high selects whether high or low 16-bits are used for p operand
def int_amdgcn_interp_p2_rtz_f16 :
  DefaultAttrsIntrinsic<[llvm_half_ty],
            [llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],
            [IntrNoMem, IntrSpeculatable,
             ImmArg<ArgIndex<3>>]>;

// Deprecated: use llvm.amdgcn.live.mask instead.
def int_amdgcn_ps_live : DefaultAttrsIntrinsic <
  [llvm_i1_ty],
  [],
  [IntrNoMem]>;

// Query currently live lanes.
// Returns true if lane is live (and not a helper lane).
def int_amdgcn_live_mask : DefaultAttrsIntrinsic <[llvm_i1_ty],
  [], [NoUndef<RetIndex>, IntrReadMem, IntrInaccessibleMemOnly]
>;

def int_amdgcn_mbcnt_lo :
````
- **L2337 EN**: Blank line separating nearby declarations or logic blocks.
  **L2337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2338 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.interp.p10.rtz.f16 <p>, <i>, <p0>, <high>`.
  **L2338 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.interp.p10.rtz.f16 <p>, <i>, <p0>, <high>`。
- **L2339 EN**: Comment explains nearby logic, invariants, or intent: `gfx11+ fp16 interpolation intrinsic, with round-toward-zero rounding mode.`.
  **L2339 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx11+ fp16 interpolation intrinsic, with round-toward-zero rounding mode.`。
- **L2340 EN**: Comment explains nearby logic, invariants, or intent: `high selects whether high or low 16-bits are used for p and p0 operands`.
  **L2340 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`high selects whether high or low 16-bits are used for p and p0 operands`。
- **L2341 EN**: Declares TableGen def `int_amdgcn_interp_p10_rtz_f16`.
  **L2341 CN**: 声明 TableGen def `int_amdgcn_interp_p10_rtz_f16`。
- **L2342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2342 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],`.
  **L2343 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],`。
- **L2344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2344 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2345 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>]>;`.
  **L2345 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>]>;`。
- **L2346 EN**: Blank line separating nearby declarations or logic blocks.
  **L2346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2347 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.interp.p2.rtz.f16 <p>, <j>, <tmp>, <high>`.
  **L2347 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.interp.p2.rtz.f16 <p>, <j>, <tmp>, <high>`。
- **L2348 EN**: Comment explains nearby logic, invariants, or intent: `gfx11+ fp16 interpolation intrinsic, with round-toward-zero rounding mode.`.
  **L2348 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx11+ fp16 interpolation intrinsic, with round-toward-zero rounding mode.`。
- **L2349 EN**: Comment explains nearby logic, invariants, or intent: `high selects whether high or low 16-bits are used for p operand`.
  **L2349 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`high selects whether high or low 16-bits are used for p operand`。
- **L2350 EN**: Declares TableGen def `int_amdgcn_interp_p2_rtz_f16`.
  **L2350 CN**: 声明 TableGen def `int_amdgcn_interp_p2_rtz_f16`。
- **L2351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_half_ty],`.
  **L2351 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_half_ty],`。
- **L2352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],`.
  **L2352 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_i1_ty],`。
- **L2353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrSpeculatable,`.
  **L2353 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrSpeculatable,`。
- **L2354 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>]>;`.
  **L2354 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>]>;`。
- **L2355 EN**: Blank line separating nearby declarations or logic blocks.
  **L2355 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2356 EN**: Comment explains nearby logic, invariants, or intent: `Deprecated: use llvm.amdgcn.live.mask instead.`.
  **L2356 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Deprecated: use llvm.amdgcn.live.mask instead.`。
- **L2357 EN**: Declares TableGen def `int_amdgcn_ps_live`.
  **L2357 CN**: 声明 TableGen def `int_amdgcn_ps_live`。
- **L2358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i1_ty],`.
  **L2358 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i1_ty],`。
- **L2359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L2359 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L2360 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2360 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2361 EN**: Blank line separating nearby declarations or logic blocks.
  **L2361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2362 EN**: Comment explains nearby logic, invariants, or intent: `Query currently live lanes.`.
  **L2362 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Query currently live lanes.`。
- **L2363 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if lane is live (and not a helper lane).`.
  **L2363 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if lane is live (and not a helper lane).`。
- **L2364 EN**: Declares TableGen def `int_amdgcn_live_mask`.
  **L2364 CN**: 声明 TableGen def `int_amdgcn_live_mask`。
- **L2365 EN**: Continues the surrounding expression or declaration: `[], [NoUndef<RetIndex>, IntrReadMem, IntrInaccessibleMemOnly]`.
  **L2365 CN**: 继续构造周围的表达式或声明：`[], [NoUndef<RetIndex>, IntrReadMem, IntrInaccessibleMemOnly]`。
- **L2366 EN**: Executes a standalone statement or declaration: `>;`.
  **L2366 CN**: 执行一条独立语句或声明：`>;`。
- **L2367 EN**: Blank line separating nearby declarations or logic blocks.
  **L2367 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2368 EN**: Declares TableGen def `int_amdgcn_mbcnt_lo`.
  **L2368 CN**: 声明 TableGen def `int_amdgcn_mbcnt_lo`。

### Lines 2369-2400

````tablegen
  ClangBuiltin<"__builtin_amdgcn_mbcnt_lo">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
   [IntrNoMem, IntrNoCreateUndefOrPoison]>;

def int_amdgcn_mbcnt_hi :
  ClangBuiltin<"__builtin_amdgcn_mbcnt_hi">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrNoCreateUndefOrPoison]>;

// llvm.amdgcn.ds.swizzle src offset
def int_amdgcn_ds_swizzle :
  ClangBuiltin<"__builtin_amdgcn_ds_swizzle">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree,
             ImmArg<ArgIndex<1>>]>;

def int_amdgcn_ubfe : PureIntrinsic<[llvm_anyint_ty],
    [LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty]>;

def int_amdgcn_sbfe : PureIntrinsic<[llvm_anyint_ty],
    [LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty]>;

def int_amdgcn_lerp :
  ClangBuiltin<"__builtin_amdgcn_lerp">,
  PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

def int_amdgcn_sad_u8 :
  ClangBuiltin<"__builtin_amdgcn_sad_u8">,
  PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

def int_amdgcn_msad_u8 :
  ClangBuiltin<"__builtin_amdgcn_msad_u8">,
````
- **L2369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_mbcnt_lo">,`.
  **L2369 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_mbcnt_lo">,`。
- **L2370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L2370 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L2371 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrNoCreateUndefOrPoison]>;`.
  **L2371 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrNoCreateUndefOrPoison]>;`。
- **L2372 EN**: Blank line separating nearby declarations or logic blocks.
  **L2372 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2373 EN**: Declares TableGen def `int_amdgcn_mbcnt_hi`.
  **L2373 CN**: 声明 TableGen def `int_amdgcn_mbcnt_hi`。
- **L2374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_mbcnt_hi">,`.
  **L2374 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_mbcnt_hi">,`。
- **L2375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L2375 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L2376 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrNoCreateUndefOrPoison]>;`.
  **L2376 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrNoCreateUndefOrPoison]>;`。
- **L2377 EN**: Blank line separating nearby declarations or logic blocks.
  **L2377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2378 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.swizzle src offset`.
  **L2378 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.swizzle src offset`。
- **L2379 EN**: Declares TableGen def `int_amdgcn_ds_swizzle`.
  **L2379 CN**: 声明 TableGen def `int_amdgcn_ds_swizzle`。
- **L2380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_swizzle">,`.
  **L2380 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_swizzle">,`。
- **L2381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L2381 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L2382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree,`.
  **L2382 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree,`。
- **L2383 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<1>>]>;`.
  **L2383 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<1>>]>;`。
- **L2384 EN**: Blank line separating nearby declarations or logic blocks.
  **L2384 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2385 EN**: Declares TableGen def `int_amdgcn_ubfe`.
  **L2385 CN**: 声明 TableGen def `int_amdgcn_ubfe`。
- **L2386 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2386 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2387 EN**: Blank line separating nearby declarations or logic blocks.
  **L2387 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2388 EN**: Declares TableGen def `int_amdgcn_sbfe`.
  **L2388 CN**: 声明 TableGen def `int_amdgcn_sbfe`。
- **L2389 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2389 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2390 EN**: Blank line separating nearby declarations or logic blocks.
  **L2390 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2391 EN**: Declares TableGen def `int_amdgcn_lerp`.
  **L2391 CN**: 声明 TableGen def `int_amdgcn_lerp`。
- **L2392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_lerp">,`.
  **L2392 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_lerp">,`。
- **L2393 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2393 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2394 EN**: Blank line separating nearby declarations or logic blocks.
  **L2394 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2395 EN**: Declares TableGen def `int_amdgcn_sad_u8`.
  **L2395 CN**: 声明 TableGen def `int_amdgcn_sad_u8`。
- **L2396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_sad_u8">,`.
  **L2396 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_sad_u8">,`。
- **L2397 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2397 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2398 EN**: Blank line separating nearby declarations or logic blocks.
  **L2398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2399 EN**: Declares TableGen def `int_amdgcn_msad_u8`.
  **L2399 CN**: 声明 TableGen def `int_amdgcn_msad_u8`。
- **L2400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_msad_u8">,`.
  **L2400 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_msad_u8">,`。

### Lines 2401-2432

````tablegen
  PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

def int_amdgcn_sad_hi_u8 :
  ClangBuiltin<"__builtin_amdgcn_sad_hi_u8">,
  PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

def int_amdgcn_sad_u16 :
  ClangBuiltin<"__builtin_amdgcn_sad_u16">,
  PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

def int_amdgcn_qsad_pk_u16_u8 :
  ClangBuiltin<"__builtin_amdgcn_qsad_pk_u16_u8">,
  PureIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i64_ty]>;

def int_amdgcn_mqsad_pk_u16_u8 :
  ClangBuiltin<"__builtin_amdgcn_mqsad_pk_u16_u8">,
  PureIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i64_ty]>;

def int_amdgcn_mqsad_u32_u8 :
  ClangBuiltin<"__builtin_amdgcn_mqsad_u32_u8">,
  PureIntrinsic<[llvm_v4i32_ty], [llvm_i64_ty, llvm_i32_ty, llvm_v4i32_ty]>;

def int_amdgcn_cvt_pk_u8_f32 :
  ClangBuiltin<"__builtin_amdgcn_cvt_pk_u8_f32">,
  PureIntrinsic<[llvm_i32_ty], [llvm_float_ty, llvm_i32_ty, llvm_i32_ty]>;

def int_amdgcn_icmp :
  Intrinsic<[llvm_anyint_ty], [llvm_anyint_ty, LLVMMatchType<1>, llvm_i32_ty],
            [IntrNoMem, IntrConvergent,
             ImmArg<ArgIndex<2>>, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

def int_amdgcn_fcmp :
````
- **L2401 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2401 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2402 EN**: Blank line separating nearby declarations or logic blocks.
  **L2402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2403 EN**: Declares TableGen def `int_amdgcn_sad_hi_u8`.
  **L2403 CN**: 声明 TableGen def `int_amdgcn_sad_hi_u8`。
- **L2404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_sad_hi_u8">,`.
  **L2404 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_sad_hi_u8">,`。
- **L2405 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2405 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2406 EN**: Blank line separating nearby declarations or logic blocks.
  **L2406 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2407 EN**: Declares TableGen def `int_amdgcn_sad_u16`.
  **L2407 CN**: 声明 TableGen def `int_amdgcn_sad_u16`。
- **L2408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_sad_u16">,`.
  **L2408 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_sad_u16">,`。
- **L2409 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2409 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2410 EN**: Blank line separating nearby declarations or logic blocks.
  **L2410 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2411 EN**: Declares TableGen def `int_amdgcn_qsad_pk_u16_u8`.
  **L2411 CN**: 声明 TableGen def `int_amdgcn_qsad_pk_u16_u8`。
- **L2412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_qsad_pk_u16_u8">,`.
  **L2412 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_qsad_pk_u16_u8">,`。
- **L2413 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i64_ty]>;`.
  **L2413 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i64_ty]>;`。
- **L2414 EN**: Blank line separating nearby declarations or logic blocks.
  **L2414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2415 EN**: Declares TableGen def `int_amdgcn_mqsad_pk_u16_u8`.
  **L2415 CN**: 声明 TableGen def `int_amdgcn_mqsad_pk_u16_u8`。
- **L2416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_mqsad_pk_u16_u8">,`.
  **L2416 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_mqsad_pk_u16_u8">,`。
- **L2417 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i64_ty]>;`.
  **L2417 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i64_ty]>;`。
- **L2418 EN**: Blank line separating nearby declarations or logic blocks.
  **L2418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2419 EN**: Declares TableGen def `int_amdgcn_mqsad_u32_u8`.
  **L2419 CN**: 声明 TableGen def `int_amdgcn_mqsad_u32_u8`。
- **L2420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_mqsad_u32_u8">,`.
  **L2420 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_mqsad_u32_u8">,`。
- **L2421 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v4i32_ty], [llvm_i64_ty, llvm_i32_ty, llvm_v4i32_ty]>;`.
  **L2421 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v4i32_ty], [llvm_i64_ty, llvm_i32_ty, llvm_v4i32_ty]>;`。
- **L2422 EN**: Blank line separating nearby declarations or logic blocks.
  **L2422 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2423 EN**: Declares TableGen def `int_amdgcn_cvt_pk_u8_f32`.
  **L2423 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_u8_f32`。
- **L2424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_cvt_pk_u8_f32">,`.
  **L2424 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_cvt_pk_u8_f32">,`。
- **L2425 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_float_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2425 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_float_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2426 EN**: Blank line separating nearby declarations or logic blocks.
  **L2426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2427 EN**: Declares TableGen def `int_amdgcn_icmp`.
  **L2427 CN**: 声明 TableGen def `int_amdgcn_icmp`。
- **L2428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_anyint_ty], [llvm_anyint_ty, LLVMMatchType<1>, llvm_i32_ty],`.
  **L2428 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_anyint_ty], [llvm_anyint_ty, LLVMMatchType<1>, llvm_i32_ty],`。
- **L2429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent,`.
  **L2429 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent,`。
- **L2430 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2430 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2431 EN**: Blank line separating nearby declarations or logic blocks.
  **L2431 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2432 EN**: Declares TableGen def `int_amdgcn_fcmp`.
  **L2432 CN**: 声明 TableGen def `int_amdgcn_fcmp`。

### Lines 2433-2464

````tablegen
  Intrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, LLVMMatchType<1>, llvm_i32_ty],
            [IntrNoMem, IntrConvergent,
             ImmArg<ArgIndex<2>>, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// Returns a bitfield(i32 or i64) containing the result of its i1 argument
// in all active lanes, and zero in all inactive lanes.
def int_amdgcn_ballot :
  Intrinsic<[llvm_anyint_ty], [llvm_i1_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]>;

// Inverse of ballot: return the bit corresponding to the current lane from the
// given mask.
//
// This is only defined for dynamically uniform masks and therefore convergent.
def int_amdgcn_inverse_ballot :
  Intrinsic<[llvm_i1_ty], [llvm_anyint_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]>;

// Lowers to S_BITREPLICATE_B64_B32.
// The argument must be uniform; otherwise, the result is undefined.
def int_amdgcn_s_bitreplicate :
  DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty], [IntrNoMem, IntrConvergent]>;

// Lowers to S_QUADMASK_B{32,64}
// The argument must be uniform; otherwise, the result is undefined.
def int_amdgcn_s_quadmask :
  DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyint_ty], [IntrNoMem, IntrConvergent]>;

// Lowers to S_WQM_B{32,64}
// The argument must be uniform; otherwise, the result is undefined.
// Does not set WQM; merely calculates the bitmask.
def int_amdgcn_s_wqm :
````
- **L2433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, LLVMMatchType<1>, llvm_i32_ty],`.
  **L2433 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, LLVMMatchType<1>, llvm_i32_ty],`。
- **L2434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent,`.
  **L2434 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent,`。
- **L2435 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2435 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2436 EN**: Blank line separating nearby declarations or logic blocks.
  **L2436 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2437 EN**: Comment explains nearby logic, invariants, or intent: `Returns a bitfield(i32 or i64) containing the result of its i1 argument`.
  **L2437 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns a bitfield(i32 or i64) containing the result of its i1 argument`。
- **L2438 EN**: Comment explains nearby logic, invariants, or intent: `in all active lanes, and zero in all inactive lanes.`.
  **L2438 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`in all active lanes, and zero in all inactive lanes.`。
- **L2439 EN**: Declares TableGen def `int_amdgcn_ballot`.
  **L2439 CN**: 声明 TableGen def `int_amdgcn_ballot`。
- **L2440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_anyint_ty], [llvm_i1_ty],`.
  **L2440 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_anyint_ty], [llvm_i1_ty],`。
- **L2441 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]>;`.
  **L2441 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]>;`。
- **L2442 EN**: Blank line separating nearby declarations or logic blocks.
  **L2442 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2443 EN**: Comment explains nearby logic, invariants, or intent: `Inverse of ballot: return the bit corresponding to the current lane from the`.
  **L2443 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Inverse of ballot: return the bit corresponding to the current lane from the`。
- **L2444 EN**: Comment explains nearby logic, invariants, or intent: `given mask.`.
  **L2444 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`given mask.`。
- **L2445 EN**: Separator comment used for visual grouping.
  **L2445 CN**: 用于视觉分组的分隔注释。
- **L2446 EN**: Comment explains nearby logic, invariants, or intent: `This is only defined for dynamically uniform masks and therefore convergent.`.
  **L2446 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is only defined for dynamically uniform masks and therefore convergent.`。
- **L2447 EN**: Declares TableGen def `int_amdgcn_inverse_ballot`.
  **L2447 CN**: 声明 TableGen def `int_amdgcn_inverse_ballot`。
- **L2448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i1_ty], [llvm_anyint_ty],`.
  **L2448 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i1_ty], [llvm_anyint_ty],`。
- **L2449 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]>;`.
  **L2449 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]>;`。
- **L2450 EN**: Blank line separating nearby declarations or logic blocks.
  **L2450 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2451 EN**: Comment explains nearby logic, invariants, or intent: `Lowers to S_BITREPLICATE_B64_B32.`.
  **L2451 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Lowers to S_BITREPLICATE_B64_B32.`。
- **L2452 EN**: Comment explains nearby logic, invariants, or intent: `The argument must be uniform; otherwise, the result is undefined.`.
  **L2452 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The argument must be uniform; otherwise, the result is undefined.`。
- **L2453 EN**: Declares TableGen def `int_amdgcn_s_bitreplicate`.
  **L2453 CN**: 声明 TableGen def `int_amdgcn_s_bitreplicate`。
- **L2454 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty], [IntrNoMem, IntrConvergent]>;`.
  **L2454 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty], [IntrNoMem, IntrConvergent]>;`。
- **L2455 EN**: Blank line separating nearby declarations or logic blocks.
  **L2455 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2456 EN**: Comment explains nearby logic, invariants, or intent: `Lowers to S_QUADMASK_B{32,64}`.
  **L2456 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Lowers to S_QUADMASK_B{32,64}`。
- **L2457 EN**: Comment explains nearby logic, invariants, or intent: `The argument must be uniform; otherwise, the result is undefined.`.
  **L2457 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The argument must be uniform; otherwise, the result is undefined.`。
- **L2458 EN**: Declares TableGen def `int_amdgcn_s_quadmask`.
  **L2458 CN**: 声明 TableGen def `int_amdgcn_s_quadmask`。
- **L2459 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyint_ty], [IntrNoMem, IntrConvergent]>;`.
  **L2459 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyint_ty], [IntrNoMem, IntrConvergent]>;`。
- **L2460 EN**: Blank line separating nearby declarations or logic blocks.
  **L2460 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2461 EN**: Comment explains nearby logic, invariants, or intent: `Lowers to S_WQM_B{32,64}`.
  **L2461 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Lowers to S_WQM_B{32,64}`。
- **L2462 EN**: Comment explains nearby logic, invariants, or intent: `The argument must be uniform; otherwise, the result is undefined.`.
  **L2462 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The argument must be uniform; otherwise, the result is undefined.`。
- **L2463 EN**: Comment explains nearby logic, invariants, or intent: `Does not set WQM; merely calculates the bitmask.`.
  **L2463 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Does not set WQM; merely calculates the bitmask.`。
- **L2464 EN**: Declares TableGen def `int_amdgcn_s_wqm`.
  **L2464 CN**: 声明 TableGen def `int_amdgcn_s_wqm`。

### Lines 2465-2496

````tablegen
  DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyint_ty], [IntrNoMem, IntrConvergent]>;

class AMDGPUWaveReduce<LLVMType data_ty = llvm_any_ty> : Intrinsic<
    [data_ty],
    [
      LLVMMatchType<0>,   // llvm value to reduce (SGPR/VGPR)
      llvm_i32_ty         // Reduction Strategy Switch for lowering ( 0: Default,
                          //                                          1: Iterative strategy, and
                          //                                          2. DPP)
    ],
    [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, ImmArg<ArgIndex<1>>]>;

multiclass AMDGPUWaveReduceOps {
  foreach Op =
      ["umin", "fmin", "min", "umax", "fmax", "max", "add", "fadd", "sub", "fsub", "and", "or", "xor"] in {
    def Op : AMDGPUWaveReduce;
  }
}

defm int_amdgcn_wave_reduce_ : AMDGPUWaveReduceOps;

def int_amdgcn_readfirstlane :
  Intrinsic<[llvm_any_ty], [LLVMMatchType<0>],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]>;

// The lane argument must be uniform across the currently active threads of the
// current wave. Otherwise, the result is undefined.
def int_amdgcn_readlane :
  Intrinsic<[llvm_any_ty], [LLVMMatchType<0>, llvm_i32_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// The value to write and lane select arguments must be uniform across the
````
- **L2465 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyint_ty], [IntrNoMem, IntrConvergent]>;`.
  **L2465 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyint_ty], [IntrNoMem, IntrConvergent]>;`。
- **L2466 EN**: Blank line separating nearby declarations or logic blocks.
  **L2466 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2467 EN**: Declares class `AMDGPUWaveReduce<LLVMType`.
  **L2467 CN**: 声明 class `AMDGPUWaveReduce<LLVMType`。
- **L2468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[data_ty],`.
  **L2468 CN**: 继续一个多行参数列表、初始化器或聚合项：`[data_ty],`。
- **L2469 EN**: Continues the surrounding expression or declaration: `[`.
  **L2469 CN**: 继续构造周围的表达式或声明：`[`。
- **L2470 EN**: Continues logic associated with callable symbol `reduce`.
  **L2470 CN**: 继续与可调用符号 `reduce` 相关的逻辑。
- **L2471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty         // Reduction Strategy Switch for lowering ( 0: Default,`.
  **L2471 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty         // Reduction Strategy Switch for lowering ( 0: Default,`。
- **L2472 EN**: Comment explains nearby logic, invariants, or intent: `1: Iterative strategy, and`.
  **L2472 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`1: Iterative strategy, and`。
- **L2473 EN**: Comment explains nearby logic, invariants, or intent: `2. DPP)`.
  **L2473 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2. DPP)`。
- **L2474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L2474 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L2475 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, ImmArg<ArgIndex<1>>]>;`.
  **L2475 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, ImmArg<ArgIndex<1>>]>;`。
- **L2476 EN**: Blank line separating nearby declarations or logic blocks.
  **L2476 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2477 EN**: Declares TableGen multiclass `AMDGPUWaveReduceOps`.
  **L2477 CN**: 声明 TableGen multiclass `AMDGPUWaveReduceOps`。
- **L2478 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2478 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2479 EN**: Continues the surrounding expression or declaration: `["umin", "fmin", "min", "umax", "fmax", "max", "add", "fadd", "sub", "fsub", "and", "or", "xor"] in {`.
  **L2479 CN**: 继续构造周围的表达式或声明：`["umin", "fmin", "min", "umax", "fmax", "max", "add", "fadd", "sub", "fsub", "and", "or", "xor"] in {`。
- **L2480 EN**: Declares TableGen def `Op`.
  **L2480 CN**: 声明 TableGen def `Op`。
- **L2481 EN**: Closes the current lexical scope or compound statement.
  **L2481 CN**: 结束当前词法作用域或复合语句块。
- **L2482 EN**: Closes the current lexical scope or compound statement.
  **L2482 CN**: 结束当前词法作用域或复合语句块。
- **L2483 EN**: Blank line separating nearby declarations or logic blocks.
  **L2483 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2484 EN**: Declares TableGen defm `int_amdgcn_wave_reduce_`.
  **L2484 CN**: 声明 TableGen defm `int_amdgcn_wave_reduce_`。
- **L2485 EN**: Blank line separating nearby declarations or logic blocks.
  **L2485 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2486 EN**: Declares TableGen def `int_amdgcn_readfirstlane`.
  **L2486 CN**: 声明 TableGen def `int_amdgcn_readfirstlane`。
- **L2487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty], [LLVMMatchType<0>],`.
  **L2487 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty], [LLVMMatchType<0>],`。
- **L2488 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]>;`.
  **L2488 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]>;`。
- **L2489 EN**: Blank line separating nearby declarations or logic blocks.
  **L2489 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2490 EN**: Comment explains nearby logic, invariants, or intent: `The lane argument must be uniform across the currently active threads of the`.
  **L2490 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The lane argument must be uniform across the currently active threads of the`。
- **L2491 EN**: Comment explains nearby logic, invariants, or intent: `current wave. Otherwise, the result is undefined.`.
  **L2491 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`current wave. Otherwise, the result is undefined.`。
- **L2492 EN**: Declares TableGen def `int_amdgcn_readlane`.
  **L2492 CN**: 声明 TableGen def `int_amdgcn_readlane`。
- **L2493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty], [LLVMMatchType<0>, llvm_i32_ty],`.
  **L2493 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty], [LLVMMatchType<0>, llvm_i32_ty],`。
- **L2494 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2494 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2495 EN**: Blank line separating nearby declarations or logic blocks.
  **L2495 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2496 EN**: Comment explains nearby logic, invariants, or intent: `The value to write and lane select arguments must be uniform across the`.
  **L2496 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The value to write and lane select arguments must be uniform across the`。

### Lines 2497-2528

````tablegen
// currently active threads of the current wave. Otherwise, the result is
// undefined.
def int_amdgcn_writelane :
  Intrinsic<[llvm_any_ty], [
    LLVMMatchType<0>,   // uniform value to write: returned by the selected lane
    llvm_i32_ty,        // uniform lane select
    LLVMMatchType<0>    // returned by all lanes other than the selected one
  ],
  [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

def int_amdgcn_alignbyte : ClangBuiltin<"__builtin_amdgcn_alignbyte">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
  [IntrNoMem, IntrSpeculatable]
>;

// mul24 intrinsics can return i32 or i64.
// When returning i64, they're lowered to a mul24/mulhi24 pair.
def int_amdgcn_mul_i24 : PureIntrinsic<[llvm_anyint_ty],
  [llvm_i32_ty, llvm_i32_ty]
>;

def int_amdgcn_mul_u24 : PureIntrinsic<[llvm_anyint_ty],
  [llvm_i32_ty, llvm_i32_ty]
>;

def int_amdgcn_mulhi_i24 : PureIntrinsic<[llvm_i32_ty],
  [llvm_i32_ty, llvm_i32_ty]
>;

def int_amdgcn_mulhi_u24 : PureIntrinsic<[llvm_i32_ty],
  [llvm_i32_ty, llvm_i32_ty]
````
- **L2497 EN**: Comment explains nearby logic, invariants, or intent: `currently active threads of the current wave. Otherwise, the result is`.
  **L2497 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`currently active threads of the current wave. Otherwise, the result is`。
- **L2498 EN**: Comment explains nearby logic, invariants, or intent: `undefined.`.
  **L2498 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`undefined.`。
- **L2499 EN**: Declares TableGen def `int_amdgcn_writelane`.
  **L2499 CN**: 声明 TableGen def `int_amdgcn_writelane`。
- **L2500 EN**: Continues the surrounding expression or declaration: `Intrinsic<[llvm_any_ty], [`.
  **L2500 CN**: 继续构造周围的表达式或声明：`Intrinsic<[llvm_any_ty], [`。
- **L2501 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,   // uniform value to write: returned by the selected lane`.
  **L2501 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,   // uniform value to write: returned by the selected lane`。
- **L2502 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,        // uniform lane select`.
  **L2502 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,        // uniform lane select`。
- **L2503 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>    // returned by all lanes other than the selected one`.
  **L2503 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>    // returned by all lanes other than the selected one`。
- **L2504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L2504 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L2505 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L2505 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L2506 EN**: Executes a standalone statement or declaration: `>;`.
  **L2506 CN**: 执行一条独立语句或声明：`>;`。
- **L2507 EN**: Blank line separating nearby declarations or logic blocks.
  **L2507 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2508 EN**: Declares TableGen def `int_amdgcn_alignbyte`.
  **L2508 CN**: 声明 TableGen def `int_amdgcn_alignbyte`。
- **L2509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L2509 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L2510 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrSpeculatable]`.
  **L2510 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrSpeculatable]`。
- **L2511 EN**: Executes a standalone statement or declaration: `>;`.
  **L2511 CN**: 执行一条独立语句或声明：`>;`。
- **L2512 EN**: Blank line separating nearby declarations or logic blocks.
  **L2512 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2513 EN**: Comment explains nearby logic, invariants, or intent: `mul24 intrinsics can return i32 or i64.`.
  **L2513 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mul24 intrinsics can return i32 or i64.`。
- **L2514 EN**: Comment explains nearby logic, invariants, or intent: `When returning i64, they're lowered to a mul24/mulhi24 pair.`.
  **L2514 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`When returning i64, they're lowered to a mul24/mulhi24 pair.`。
- **L2515 EN**: Declares TableGen def `int_amdgcn_mul_i24`.
  **L2515 CN**: 声明 TableGen def `int_amdgcn_mul_i24`。
- **L2516 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty, llvm_i32_ty]`.
  **L2516 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty, llvm_i32_ty]`。
- **L2517 EN**: Executes a standalone statement or declaration: `>;`.
  **L2517 CN**: 执行一条独立语句或声明：`>;`。
- **L2518 EN**: Blank line separating nearby declarations or logic blocks.
  **L2518 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2519 EN**: Declares TableGen def `int_amdgcn_mul_u24`.
  **L2519 CN**: 声明 TableGen def `int_amdgcn_mul_u24`。
- **L2520 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty, llvm_i32_ty]`.
  **L2520 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty, llvm_i32_ty]`。
- **L2521 EN**: Executes a standalone statement or declaration: `>;`.
  **L2521 CN**: 执行一条独立语句或声明：`>;`。
- **L2522 EN**: Blank line separating nearby declarations or logic blocks.
  **L2522 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2523 EN**: Declares TableGen def `int_amdgcn_mulhi_i24`.
  **L2523 CN**: 声明 TableGen def `int_amdgcn_mulhi_i24`。
- **L2524 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty, llvm_i32_ty]`.
  **L2524 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty, llvm_i32_ty]`。
- **L2525 EN**: Executes a standalone statement or declaration: `>;`.
  **L2525 CN**: 执行一条独立语句或声明：`>;`。
- **L2526 EN**: Blank line separating nearby declarations or logic blocks.
  **L2526 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2527 EN**: Declares TableGen def `int_amdgcn_mulhi_u24`.
  **L2527 CN**: 声明 TableGen def `int_amdgcn_mulhi_u24`。
- **L2528 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty, llvm_i32_ty]`.
  **L2528 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty, llvm_i32_ty]`。

### Lines 2529-2560

````tablegen
>;

// llvm.amdgcn.ds.gws.init(i32 bar_val, i32 resource_id)
//
// bar_val is the total number of waves that will wait on this
// barrier, minus 1.
def int_amdgcn_ds_gws_init :
  ClangBuiltin<"__builtin_amdgcn_ds_gws_init">,
  Intrinsic<[],
  [llvm_i32_ty, llvm_i32_ty],
  [IntrConvergent, IntrWriteMem,
   IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",
  [SDNPMemOperand]
>;

// llvm.amdgcn.ds.gws.barrier(i32 vsrc0, i32 resource_id)
// bar_val is the total number of waves that will wait on this
// barrier, minus 1.
def int_amdgcn_ds_gws_barrier :
  ClangBuiltin<"__builtin_amdgcn_ds_gws_barrier">,
  Intrinsic<[],
  [llvm_i32_ty, llvm_i32_ty],
  [IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",
  [SDNPMemOperand]
>;

// llvm.amdgcn.ds.gws.sema.v(i32 resource_id)
def int_amdgcn_ds_gws_sema_v :
  ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_v">,
  Intrinsic<[],
  [llvm_i32_ty],
  [IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",
````
- **L2529 EN**: Executes a standalone statement or declaration: `>;`.
  **L2529 CN**: 执行一条独立语句或声明：`>;`。
- **L2530 EN**: Blank line separating nearby declarations or logic blocks.
  **L2530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2531 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.gws.init(i32 bar_val, i32 resource_id)`.
  **L2531 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.gws.init(i32 bar_val, i32 resource_id)`。
- **L2532 EN**: Separator comment used for visual grouping.
  **L2532 CN**: 用于视觉分组的分隔注释。
- **L2533 EN**: Comment explains nearby logic, invariants, or intent: `bar_val is the total number of waves that will wait on this`.
  **L2533 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bar_val is the total number of waves that will wait on this`。
- **L2534 EN**: Comment explains nearby logic, invariants, or intent: `barrier, minus 1.`.
  **L2534 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`barrier, minus 1.`。
- **L2535 EN**: Declares TableGen def `int_amdgcn_ds_gws_init`.
  **L2535 CN**: 声明 TableGen def `int_amdgcn_ds_gws_init`。
- **L2536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_gws_init">,`.
  **L2536 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_gws_init">,`。
- **L2537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L2537 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L2538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L2538 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L2539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrWriteMem,`.
  **L2539 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrWriteMem,`。
- **L2540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`.
  **L2540 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`。
- **L2541 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand]`.
  **L2541 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand]`。
- **L2542 EN**: Executes a standalone statement or declaration: `>;`.
  **L2542 CN**: 执行一条独立语句或声明：`>;`。
- **L2543 EN**: Blank line separating nearby declarations or logic blocks.
  **L2543 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2544 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.gws.barrier(i32 vsrc0, i32 resource_id)`.
  **L2544 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.gws.barrier(i32 vsrc0, i32 resource_id)`。
- **L2545 EN**: Comment explains nearby logic, invariants, or intent: `bar_val is the total number of waves that will wait on this`.
  **L2545 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bar_val is the total number of waves that will wait on this`。
- **L2546 EN**: Comment explains nearby logic, invariants, or intent: `barrier, minus 1.`.
  **L2546 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`barrier, minus 1.`。
- **L2547 EN**: Declares TableGen def `int_amdgcn_ds_gws_barrier`.
  **L2547 CN**: 声明 TableGen def `int_amdgcn_ds_gws_barrier`。
- **L2548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_gws_barrier">,`.
  **L2548 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_gws_barrier">,`。
- **L2549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L2549 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L2550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L2550 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L2551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`.
  **L2551 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`。
- **L2552 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand]`.
  **L2552 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand]`。
- **L2553 EN**: Executes a standalone statement or declaration: `>;`.
  **L2553 CN**: 执行一条独立语句或声明：`>;`。
- **L2554 EN**: Blank line separating nearby declarations or logic blocks.
  **L2554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2555 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.gws.sema.v(i32 resource_id)`.
  **L2555 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.gws.sema.v(i32 resource_id)`。
- **L2556 EN**: Declares TableGen def `int_amdgcn_ds_gws_sema_v`.
  **L2556 CN**: 声明 TableGen def `int_amdgcn_ds_gws_sema_v`。
- **L2557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_v">,`.
  **L2557 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_v">,`。
- **L2558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L2558 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L2559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L2559 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L2560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`.
  **L2560 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`。

### Lines 2561-2592

````tablegen
  [SDNPMemOperand]
>;

// llvm.amdgcn.ds.gws.sema.br(i32 vsrc, i32 resource_id)
def int_amdgcn_ds_gws_sema_br :
  ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_br">,
  Intrinsic<[],
  [llvm_i32_ty, llvm_i32_ty],
  [IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",
  [SDNPMemOperand]
>;

// llvm.amdgcn.ds.gws.sema.p(i32 resource_id)
def int_amdgcn_ds_gws_sema_p :
  ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_p">,
  Intrinsic<[],
  [llvm_i32_ty],
  [IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",
  [SDNPMemOperand]
>;

// llvm.amdgcn.ds.gws.sema.release.all(i32 resource_id)
def int_amdgcn_ds_gws_sema_release_all :
  ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_release_all">,
  Intrinsic<[],
  [llvm_i32_ty],
  [IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",
  [SDNPMemOperand]
>;


// Copies the source value to the destination value, with the guarantee that
````
- **L2561 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand]`.
  **L2561 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand]`。
- **L2562 EN**: Executes a standalone statement or declaration: `>;`.
  **L2562 CN**: 执行一条独立语句或声明：`>;`。
- **L2563 EN**: Blank line separating nearby declarations or logic blocks.
  **L2563 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2564 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.gws.sema.br(i32 vsrc, i32 resource_id)`.
  **L2564 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.gws.sema.br(i32 vsrc, i32 resource_id)`。
- **L2565 EN**: Declares TableGen def `int_amdgcn_ds_gws_sema_br`.
  **L2565 CN**: 声明 TableGen def `int_amdgcn_ds_gws_sema_br`。
- **L2566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_br">,`.
  **L2566 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_br">,`。
- **L2567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L2567 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L2568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L2568 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L2569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`.
  **L2569 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`。
- **L2570 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand]`.
  **L2570 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand]`。
- **L2571 EN**: Executes a standalone statement or declaration: `>;`.
  **L2571 CN**: 执行一条独立语句或声明：`>;`。
- **L2572 EN**: Blank line separating nearby declarations or logic blocks.
  **L2572 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2573 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.gws.sema.p(i32 resource_id)`.
  **L2573 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.gws.sema.p(i32 resource_id)`。
- **L2574 EN**: Declares TableGen def `int_amdgcn_ds_gws_sema_p`.
  **L2574 CN**: 声明 TableGen def `int_amdgcn_ds_gws_sema_p`。
- **L2575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_p">,`.
  **L2575 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_p">,`。
- **L2576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L2576 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L2577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L2577 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L2578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`.
  **L2578 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`。
- **L2579 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand]`.
  **L2579 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand]`。
- **L2580 EN**: Executes a standalone statement or declaration: `>;`.
  **L2580 CN**: 执行一条独立语句或声明：`>;`。
- **L2581 EN**: Blank line separating nearby declarations or logic blocks.
  **L2581 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2582 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.gws.sema.release.all(i32 resource_id)`.
  **L2582 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.gws.sema.release.all(i32 resource_id)`。
- **L2583 EN**: Declares TableGen def `int_amdgcn_ds_gws_sema_release_all`.
  **L2583 CN**: 声明 TableGen def `int_amdgcn_ds_gws_sema_release_all`。
- **L2584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_release_all">,`.
  **L2584 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_gws_sema_release_all">,`。
- **L2585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L2585 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L2586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L2586 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L2587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`.
  **L2587 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOnly, IntrWillReturn, IntrNoCallback, IntrNoFree], "",`。
- **L2588 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand]`.
  **L2588 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand]`。
- **L2589 EN**: Executes a standalone statement or declaration: `>;`.
  **L2589 CN**: 执行一条独立语句或声明：`>;`。
- **L2590 EN**: Blank line separating nearby declarations or logic blocks.
  **L2590 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2591 EN**: Blank line separating nearby declarations or logic blocks.
  **L2591 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2592 EN**: Comment explains nearby logic, invariants, or intent: `Copies the source value to the destination value, with the guarantee that`.
  **L2592 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copies the source value to the destination value, with the guarantee that`。

### Lines 2593-2624

````tablegen
// the source value is computed as if the entire program were executed in WQM.
def int_amdgcn_wqm : Intrinsic<[llvm_any_ty],
  [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

// Copies the source value to the destination value, such that the source
// is computed as if the entire program were executed in WQM if any other
// program code executes in WQM.
def int_amdgcn_softwqm : Intrinsic<[llvm_any_ty],
  [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

// Return true if at least one thread within the pixel quad passes true into
// the function.
def int_amdgcn_wqm_vote : Intrinsic<[llvm_i1_ty],
  [llvm_i1_ty], [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

// If false, set EXEC=0 for the current thread until the end of program.
// FIXME: Should this be IntrNoMem, IntrHasSideEffects, or IntrWillReturn?
def int_amdgcn_kill : Intrinsic<[], [llvm_i1_ty], [IntrNoCallback, IntrNoFree]>;

def int_amdgcn_endpgm : ClangBuiltin<"__builtin_amdgcn_endpgm">,
  Intrinsic<[], [], [IntrNoReturn, IntrCold, IntrNoMem, IntrHasSideEffects, IntrConvergent,
                     IntrNoCallback, IntrNoFree]
>;

// If false, mark all active lanes as helper lanes until the end of program.
def int_amdgcn_wqm_demote : Intrinsic<[],
  [llvm_i1_ty], [IntrWriteMem, IntrInaccessibleMemOnly, IntrNoCallback, IntrNoFree]
>;

````
- **L2593 EN**: Comment explains nearby logic, invariants, or intent: `the source value is computed as if the entire program were executed in WQM.`.
  **L2593 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the source value is computed as if the entire program were executed in WQM.`。
- **L2594 EN**: Declares TableGen def `int_amdgcn_wqm`.
  **L2594 CN**: 声明 TableGen def `int_amdgcn_wqm`。
- **L2595 EN**: Continues the surrounding expression or declaration: `[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L2595 CN**: 继续构造周围的表达式或声明：`[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L2596 EN**: Executes a standalone statement or declaration: `>;`.
  **L2596 CN**: 执行一条独立语句或声明：`>;`。
- **L2597 EN**: Blank line separating nearby declarations or logic blocks.
  **L2597 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2598 EN**: Comment explains nearby logic, invariants, or intent: `Copies the source value to the destination value, such that the source`.
  **L2598 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copies the source value to the destination value, such that the source`。
- **L2599 EN**: Comment explains nearby logic, invariants, or intent: `is computed as if the entire program were executed in WQM if any other`.
  **L2599 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is computed as if the entire program were executed in WQM if any other`。
- **L2600 EN**: Comment explains nearby logic, invariants, or intent: `program code executes in WQM.`.
  **L2600 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`program code executes in WQM.`。
- **L2601 EN**: Declares TableGen def `int_amdgcn_softwqm`.
  **L2601 CN**: 声明 TableGen def `int_amdgcn_softwqm`。
- **L2602 EN**: Continues the surrounding expression or declaration: `[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L2602 CN**: 继续构造周围的表达式或声明：`[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L2603 EN**: Executes a standalone statement or declaration: `>;`.
  **L2603 CN**: 执行一条独立语句或声明：`>;`。
- **L2604 EN**: Blank line separating nearby declarations or logic blocks.
  **L2604 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2605 EN**: Comment explains nearby logic, invariants, or intent: `Return true if at least one thread within the pixel quad passes true into`.
  **L2605 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return true if at least one thread within the pixel quad passes true into`。
- **L2606 EN**: Comment explains nearby logic, invariants, or intent: `the function.`.
  **L2606 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the function.`。
- **L2607 EN**: Declares TableGen def `int_amdgcn_wqm_vote`.
  **L2607 CN**: 声明 TableGen def `int_amdgcn_wqm_vote`。
- **L2608 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty], [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L2608 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty], [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L2609 EN**: Executes a standalone statement or declaration: `>;`.
  **L2609 CN**: 执行一条独立语句或声明：`>;`。
- **L2610 EN**: Blank line separating nearby declarations or logic blocks.
  **L2610 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2611 EN**: Comment explains nearby logic, invariants, or intent: `If false, set EXEC=0 for the current thread until the end of program.`.
  **L2611 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If false, set EXEC=0 for the current thread until the end of program.`。
- **L2612 EN**: Comment records a pending task or caution: `FIXME: Should this be IntrNoMem, IntrHasSideEffects, or IntrWillReturn?`.
  **L2612 CN**: 注释记录了待办事项或注意点：`FIXME: Should this be IntrNoMem, IntrHasSideEffects, or IntrWillReturn?`。
- **L2613 EN**: Declares TableGen def `int_amdgcn_kill`.
  **L2613 CN**: 声明 TableGen def `int_amdgcn_kill`。
- **L2614 EN**: Blank line separating nearby declarations or logic blocks.
  **L2614 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2615 EN**: Declares TableGen def `int_amdgcn_endpgm`.
  **L2615 CN**: 声明 TableGen def `int_amdgcn_endpgm`。
- **L2616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [], [IntrNoReturn, IntrCold, IntrNoMem, IntrHasSideEffects, IntrConvergent,`.
  **L2616 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [], [IntrNoReturn, IntrCold, IntrNoMem, IntrHasSideEffects, IntrConvergent,`。
- **L2617 EN**: Continues the surrounding expression or declaration: `IntrNoCallback, IntrNoFree]`.
  **L2617 CN**: 继续构造周围的表达式或声明：`IntrNoCallback, IntrNoFree]`。
- **L2618 EN**: Executes a standalone statement or declaration: `>;`.
  **L2618 CN**: 执行一条独立语句或声明：`>;`。
- **L2619 EN**: Blank line separating nearby declarations or logic blocks.
  **L2619 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2620 EN**: Comment explains nearby logic, invariants, or intent: `If false, mark all active lanes as helper lanes until the end of program.`.
  **L2620 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If false, mark all active lanes as helper lanes until the end of program.`。
- **L2621 EN**: Declares TableGen def `int_amdgcn_wqm_demote`.
  **L2621 CN**: 声明 TableGen def `int_amdgcn_wqm_demote`。
- **L2622 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty], [IntrWriteMem, IntrInaccessibleMemOnly, IntrNoCallback, IntrNoFree]`.
  **L2622 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty], [IntrWriteMem, IntrInaccessibleMemOnly, IntrNoCallback, IntrNoFree]`。
- **L2623 EN**: Executes a standalone statement or declaration: `>;`.
  **L2623 CN**: 执行一条独立语句或声明：`>;`。
- **L2624 EN**: Blank line separating nearby declarations or logic blocks.
  **L2624 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2625-2656

````tablegen
// Copies the active channels of the source value to the destination value,
// with the guarantee that the source value is computed as if the entire
// program were executed in Whole Wavefront Mode, i.e. with all channels
// enabled, with a few exceptions: - Phi nodes which require WWM return an
// undefined value.
def int_amdgcn_strict_wwm : Intrinsic<[llvm_any_ty],
  [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable,
                       IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;
// Deprecated. Use int_amdgcn_strict_wwm instead.
def int_amdgcn_wwm : Intrinsic<[llvm_any_ty],
  [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable,
                       IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;
def int_amdgcn_strict_wqm : Intrinsic<[llvm_any_ty],
  [LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable,
                       IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

// Given a value, copies it while setting all the inactive lanes to a given
// value. Note that OpenGL helper lanes are considered active, so if the
// program ever uses WQM, then the instruction and the first source will be
// computed in WQM.
def int_amdgcn_set_inactive :
  Intrinsic<[llvm_any_ty],
            [LLVMMatchType<0>, // value to be copied
             LLVMMatchType<0>], // value for the inactive lanes to take
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// Similar to int_amdgcn_set_inactive, but the value for the inactive lanes must
// be a VGPR function argument.
// Can only be used in functions with the `amdgpu_cs_chain` or
````
- **L2625 EN**: Comment explains nearby logic, invariants, or intent: `Copies the active channels of the source value to the destination value,`.
  **L2625 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copies the active channels of the source value to the destination value,`。
- **L2626 EN**: Comment explains nearby logic, invariants, or intent: `with the guarantee that the source value is computed as if the entire`.
  **L2626 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`with the guarantee that the source value is computed as if the entire`。
- **L2627 EN**: Comment explains nearby logic, invariants, or intent: `program were executed in Whole Wavefront Mode, i.e. with all channels`.
  **L2627 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`program were executed in Whole Wavefront Mode, i.e. with all channels`。
- **L2628 EN**: Comment explains nearby logic, invariants, or intent: `enabled, with a few exceptions: - Phi nodes which require WWM return an`.
  **L2628 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`enabled, with a few exceptions: - Phi nodes which require WWM return an`。
- **L2629 EN**: Comment explains nearby logic, invariants, or intent: `undefined value.`.
  **L2629 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`undefined value.`。
- **L2630 EN**: Declares TableGen def `int_amdgcn_strict_wwm`.
  **L2630 CN**: 声明 TableGen def `int_amdgcn_strict_wwm`。
- **L2631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable,`.
  **L2631 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable,`。
- **L2632 EN**: Continues the surrounding expression or declaration: `IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L2632 CN**: 继续构造周围的表达式或声明：`IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L2633 EN**: Executes a standalone statement or declaration: `>;`.
  **L2633 CN**: 执行一条独立语句或声明：`>;`。
- **L2634 EN**: Comment explains nearby logic, invariants, or intent: `Deprecated. Use int_amdgcn_strict_wwm instead.`.
  **L2634 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Deprecated. Use int_amdgcn_strict_wwm instead.`。
- **L2635 EN**: Declares TableGen def `int_amdgcn_wwm`.
  **L2635 CN**: 声明 TableGen def `int_amdgcn_wwm`。
- **L2636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable,`.
  **L2636 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable,`。
- **L2637 EN**: Continues the surrounding expression or declaration: `IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L2637 CN**: 继续构造周围的表达式或声明：`IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L2638 EN**: Executes a standalone statement or declaration: `>;`.
  **L2638 CN**: 执行一条独立语句或声明：`>;`。
- **L2639 EN**: Declares TableGen def `int_amdgcn_strict_wqm`.
  **L2639 CN**: 声明 TableGen def `int_amdgcn_strict_wqm`。
- **L2640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable,`.
  **L2640 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>], [IntrNoMem, IntrSpeculatable,`。
- **L2641 EN**: Continues the surrounding expression or declaration: `IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L2641 CN**: 继续构造周围的表达式或声明：`IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L2642 EN**: Executes a standalone statement or declaration: `>;`.
  **L2642 CN**: 执行一条独立语句或声明：`>;`。
- **L2643 EN**: Blank line separating nearby declarations or logic blocks.
  **L2643 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2644 EN**: Comment explains nearby logic, invariants, or intent: `Given a value, copies it while setting all the inactive lanes to a given`.
  **L2644 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Given a value, copies it while setting all the inactive lanes to a given`。
- **L2645 EN**: Comment explains nearby logic, invariants, or intent: `value. Note that OpenGL helper lanes are considered active, so if the`.
  **L2645 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`value. Note that OpenGL helper lanes are considered active, so if the`。
- **L2646 EN**: Comment explains nearby logic, invariants, or intent: `program ever uses WQM, then the instruction and the first source will be`.
  **L2646 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`program ever uses WQM, then the instruction and the first source will be`。
- **L2647 EN**: Comment explains nearby logic, invariants, or intent: `computed in WQM.`.
  **L2647 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`computed in WQM.`。
- **L2648 EN**: Declares TableGen def `int_amdgcn_set_inactive`.
  **L2648 CN**: 声明 TableGen def `int_amdgcn_set_inactive`。
- **L2649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty],`.
  **L2649 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty],`。
- **L2650 EN**: Continues the surrounding expression or declaration: `[LLVMMatchType<0>, // value to be copied`.
  **L2650 CN**: 继续构造周围的表达式或声明：`[LLVMMatchType<0>, // value to be copied`。
- **L2651 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>], // value for the inactive lanes to take`.
  **L2651 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>], // value for the inactive lanes to take`。
- **L2652 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2652 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2653 EN**: Blank line separating nearby declarations or logic blocks.
  **L2653 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2654 EN**: Comment explains nearby logic, invariants, or intent: `Similar to int_amdgcn_set_inactive, but the value for the inactive lanes must`.
  **L2654 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Similar to int_amdgcn_set_inactive, but the value for the inactive lanes must`。
- **L2655 EN**: Comment explains nearby logic, invariants, or intent: `be a VGPR function argument.`.
  **L2655 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`be a VGPR function argument.`。
- **L2656 EN**: Comment explains nearby logic, invariants, or intent: `Can only be used in functions with the `amdgpu_cs_chain` or`.
  **L2656 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Can only be used in functions with the `amdgpu_cs_chain` or`。

### Lines 2657-2688

````tablegen
// `amdgpu_cs_chain_preserve` calling conventions, and only in uniform control
// flow.
def int_amdgcn_set_inactive_chain_arg :
  Intrinsic<[llvm_anyint_ty],
            [LLVMMatchType<0>, // value to be copied
             LLVMMatchType<0>], // value for the inactive lanes to take
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// Return if the given flat pointer points to a local memory address.
def int_amdgcn_is_shared : ClangBuiltin<"__builtin_amdgcn_is_shared">,
  PureIntrinsic<[llvm_i1_ty], [llvm_ptr_ty]
  // FIXME: This should be captures(ret: address)
>;

// Return if the given flat pointer points to a prvate memory address.
def int_amdgcn_is_private : ClangBuiltin<"__builtin_amdgcn_is_private">,
  PureIntrinsic<[llvm_i1_ty], [llvm_ptr_ty]
  // FIXME: This should be captures(ret: address)
>;

// A uniform tail call to a function with the `amdgpu_cs_chain` or
// `amdgpu_cs_chain_preserve` calling convention. It will populate the SGPRs
// starting at s0 and the VGPRs starting at v8, set EXEC and perform a jump to
// the given function.
// Can only be used in functions with the `amdgpu_cs`, `amdgpu_cs_chain` or
// `amdgpu_cs_chain_preserve` calling conventions, and only in uniform control
// flow.
def int_amdgcn_cs_chain:
  Intrinsic<[],
            [llvm_anyptr_ty, // The function to jump to.
             llvm_anyint_ty, // Value to put in EXEC (should be i32 or i64).
             llvm_any_ty, // Arguments that will be copied into SGPRs (s0+).
````
- **L2657 EN**: Comment explains nearby logic, invariants, or intent: ``amdgpu_cs_chain_preserve` calling conventions, and only in uniform control`.
  **L2657 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``amdgpu_cs_chain_preserve` calling conventions, and only in uniform control`。
- **L2658 EN**: Comment explains nearby logic, invariants, or intent: `flow.`.
  **L2658 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`flow.`。
- **L2659 EN**: Declares TableGen def `int_amdgcn_set_inactive_chain_arg`.
  **L2659 CN**: 声明 TableGen def `int_amdgcn_set_inactive_chain_arg`。
- **L2660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_anyint_ty],`.
  **L2660 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_anyint_ty],`。
- **L2661 EN**: Continues the surrounding expression or declaration: `[LLVMMatchType<0>, // value to be copied`.
  **L2661 CN**: 继续构造周围的表达式或声明：`[LLVMMatchType<0>, // value to be copied`。
- **L2662 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>], // value for the inactive lanes to take`.
  **L2662 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>], // value for the inactive lanes to take`。
- **L2663 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2663 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2664 EN**: Blank line separating nearby declarations or logic blocks.
  **L2664 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2665 EN**: Comment explains nearby logic, invariants, or intent: `Return if the given flat pointer points to a local memory address.`.
  **L2665 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return if the given flat pointer points to a local memory address.`。
- **L2666 EN**: Declares TableGen def `int_amdgcn_is_shared`.
  **L2666 CN**: 声明 TableGen def `int_amdgcn_is_shared`。
- **L2667 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<[llvm_i1_ty], [llvm_ptr_ty]`.
  **L2667 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<[llvm_i1_ty], [llvm_ptr_ty]`。
- **L2668 EN**: Comment records a pending task or caution: `FIXME: This should be captures(ret: address)`.
  **L2668 CN**: 注释记录了待办事项或注意点：`FIXME: This should be captures(ret: address)`。
- **L2669 EN**: Executes a standalone statement or declaration: `>;`.
  **L2669 CN**: 执行一条独立语句或声明：`>;`。
- **L2670 EN**: Blank line separating nearby declarations or logic blocks.
  **L2670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2671 EN**: Comment explains nearby logic, invariants, or intent: `Return if the given flat pointer points to a prvate memory address.`.
  **L2671 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return if the given flat pointer points to a prvate memory address.`。
- **L2672 EN**: Declares TableGen def `int_amdgcn_is_private`.
  **L2672 CN**: 声明 TableGen def `int_amdgcn_is_private`。
- **L2673 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<[llvm_i1_ty], [llvm_ptr_ty]`.
  **L2673 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<[llvm_i1_ty], [llvm_ptr_ty]`。
- **L2674 EN**: Comment records a pending task or caution: `FIXME: This should be captures(ret: address)`.
  **L2674 CN**: 注释记录了待办事项或注意点：`FIXME: This should be captures(ret: address)`。
- **L2675 EN**: Executes a standalone statement or declaration: `>;`.
  **L2675 CN**: 执行一条独立语句或声明：`>;`。
- **L2676 EN**: Blank line separating nearby declarations or logic blocks.
  **L2676 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2677 EN**: Comment explains nearby logic, invariants, or intent: `A uniform tail call to a function with the `amdgpu_cs_chain` or`.
  **L2677 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A uniform tail call to a function with the `amdgpu_cs_chain` or`。
- **L2678 EN**: Comment explains nearby logic, invariants, or intent: ``amdgpu_cs_chain_preserve` calling convention. It will populate the SGPRs`.
  **L2678 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``amdgpu_cs_chain_preserve` calling convention. It will populate the SGPRs`。
- **L2679 EN**: Comment explains nearby logic, invariants, or intent: `starting at s0 and the VGPRs starting at v8, set EXEC and perform a jump to`.
  **L2679 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`starting at s0 and the VGPRs starting at v8, set EXEC and perform a jump to`。
- **L2680 EN**: Comment explains nearby logic, invariants, or intent: `the given function.`.
  **L2680 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the given function.`。
- **L2681 EN**: Comment explains nearby logic, invariants, or intent: `Can only be used in functions with the `amdgpu_cs`, `amdgpu_cs_chain` or`.
  **L2681 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Can only be used in functions with the `amdgpu_cs`, `amdgpu_cs_chain` or`。
- **L2682 EN**: Comment explains nearby logic, invariants, or intent: ``amdgpu_cs_chain_preserve` calling conventions, and only in uniform control`.
  **L2682 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``amdgpu_cs_chain_preserve` calling conventions, and only in uniform control`。
- **L2683 EN**: Comment explains nearby logic, invariants, or intent: `flow.`.
  **L2683 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`flow.`。
- **L2684 EN**: Declares TableGen def `int_amdgcn_cs_chain`.
  **L2684 CN**: 声明 TableGen def `int_amdgcn_cs_chain`。
- **L2685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L2685 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L2686 EN**: Continues the surrounding expression or declaration: `[llvm_anyptr_ty, // The function to jump to.`.
  **L2686 CN**: 继续构造周围的表达式或声明：`[llvm_anyptr_ty, // The function to jump to.`。
- **L2687 EN**: Continues logic associated with callable symbol `EXEC`.
  **L2687 CN**: 继续与可调用符号 `EXEC` 相关的逻辑。
- **L2688 EN**: Continues logic associated with callable symbol `SGPRs`.
  **L2688 CN**: 继续与可调用符号 `SGPRs` 相关的逻辑。

### Lines 2689-2720

````tablegen
                          // Must be uniform.
             llvm_any_ty, // Arguments that will be copied into VGPRs (v8+).
                          // Need not be uniform.
             llvm_i32_ty, // Flags.
             llvm_vararg_ty // Additional arguments. Only present if Flags is
                            // non-zero.
            ],
            [IntrConvergent, IntrNoReturn, ImmArg<ArgIndex<4>>]>;

// Run a function with all the lanes enabled. Only direct calls are allowed. The
// first argument is the callee, which must have the `amdgpu_gfx_whole_wave`
// calling convention and must not be variadic. The remaining arguments to the
// callee are taken from the arguments passed to the intrinsic. Lanes that are
// inactive at the point of the call will receive poison. The return value is
// the return value of the callee for the active lanes (there is no return
// value in the inactive ones).
def int_amdgcn_call_whole_wave:
  Intrinsic<[llvm_any_ty],    // The return type of the callee.
            [llvm_anyptr_ty,  // The callee.
             llvm_vararg_ty], // The arguments to the callee.
            [IntrConvergent]>;

// <result> llvm.amdgcn.wave.shuffle <value> <id>
// value and result can be a 32bit floating-point or
// integer type, and must be the same type. Any index
// value that's outside the valid range will wrap around,
// and reading from an inactive lane will return poison.
def int_amdgcn_wave_shuffle :
  DefaultAttrsIntrinsic<[llvm_any_ty],                    // return type
                        [LLVMMatchType<0>, llvm_i32_ty],  // arg types
                        [IntrConvergent, IntrNoMem]>;     // flags

````
- **L2689 EN**: Comment explains nearby logic, invariants, or intent: `Must be uniform.`.
  **L2689 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Must be uniform.`。
- **L2690 EN**: Continues logic associated with callable symbol `VGPRs`.
  **L2690 CN**: 继续与可调用符号 `VGPRs` 相关的逻辑。
- **L2691 EN**: Comment explains nearby logic, invariants, or intent: `Need not be uniform.`.
  **L2691 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Need not be uniform.`。
- **L2692 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // Flags.`.
  **L2692 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // Flags.`。
- **L2693 EN**: Continues the surrounding expression or declaration: `llvm_vararg_ty // Additional arguments. Only present if Flags is`.
  **L2693 CN**: 继续构造周围的表达式或声明：`llvm_vararg_ty // Additional arguments. Only present if Flags is`。
- **L2694 EN**: Comment explains nearby logic, invariants, or intent: `non-zero.`.
  **L2694 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`non-zero.`。
- **L2695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L2695 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L2696 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrNoReturn, ImmArg<ArgIndex<4>>]>;`.
  **L2696 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrNoReturn, ImmArg<ArgIndex<4>>]>;`。
- **L2697 EN**: Blank line separating nearby declarations or logic blocks.
  **L2697 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2698 EN**: Comment explains nearby logic, invariants, or intent: `Run a function with all the lanes enabled. Only direct calls are allowed. The`.
  **L2698 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Run a function with all the lanes enabled. Only direct calls are allowed. The`。
- **L2699 EN**: Comment explains nearby logic, invariants, or intent: `first argument is the callee, which must have the `amdgpu_gfx_whole_wave``.
  **L2699 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`first argument is the callee, which must have the `amdgpu_gfx_whole_wave``。
- **L2700 EN**: Comment explains nearby logic, invariants, or intent: `calling convention and must not be variadic. The remaining arguments to the`.
  **L2700 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`calling convention and must not be variadic. The remaining arguments to the`。
- **L2701 EN**: Comment explains nearby logic, invariants, or intent: `callee are taken from the arguments passed to the intrinsic. Lanes that are`.
  **L2701 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`callee are taken from the arguments passed to the intrinsic. Lanes that are`。
- **L2702 EN**: Comment explains nearby logic, invariants, or intent: `inactive at the point of the call will receive poison. The return value is`.
  **L2702 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`inactive at the point of the call will receive poison. The return value is`。
- **L2703 EN**: Comment explains nearby logic, invariants, or intent: `the return value of the callee for the active lanes (there is no return`.
  **L2703 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the return value of the callee for the active lanes (there is no return`。
- **L2704 EN**: Comment explains nearby logic, invariants, or intent: `value in the inactive ones).`.
  **L2704 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`value in the inactive ones).`。
- **L2705 EN**: Declares TableGen def `int_amdgcn_call_whole_wave`.
  **L2705 CN**: 声明 TableGen def `int_amdgcn_call_whole_wave`。
- **L2706 EN**: Continues the surrounding expression or declaration: `Intrinsic<[llvm_any_ty],    // The return type of the callee.`.
  **L2706 CN**: 继续构造周围的表达式或声明：`Intrinsic<[llvm_any_ty],    // The return type of the callee.`。
- **L2707 EN**: Continues the surrounding expression or declaration: `[llvm_anyptr_ty,  // The callee.`.
  **L2707 CN**: 继续构造周围的表达式或声明：`[llvm_anyptr_ty,  // The callee.`。
- **L2708 EN**: Continues the surrounding expression or declaration: `llvm_vararg_ty], // The arguments to the callee.`.
  **L2708 CN**: 继续构造周围的表达式或声明：`llvm_vararg_ty], // The arguments to the callee.`。
- **L2709 EN**: Executes a standalone statement or declaration: `[IntrConvergent]>;`.
  **L2709 CN**: 执行一条独立语句或声明：`[IntrConvergent]>;`。
- **L2710 EN**: Blank line separating nearby declarations or logic blocks.
  **L2710 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2711 EN**: Comment explains nearby logic, invariants, or intent: `<result> llvm.amdgcn.wave.shuffle <value> <id>`.
  **L2711 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<result> llvm.amdgcn.wave.shuffle <value> <id>`。
- **L2712 EN**: Comment explains nearby logic, invariants, or intent: `value and result can be a 32bit floating-point or`.
  **L2712 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`value and result can be a 32bit floating-point or`。
- **L2713 EN**: Comment explains nearby logic, invariants, or intent: `integer type, and must be the same type. Any index`.
  **L2713 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`integer type, and must be the same type. Any index`。
- **L2714 EN**: Comment explains nearby logic, invariants, or intent: `value that's outside the valid range will wrap around,`.
  **L2714 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`value that's outside the valid range will wrap around,`。
- **L2715 EN**: Comment explains nearby logic, invariants, or intent: `and reading from an inactive lane will return poison.`.
  **L2715 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and reading from an inactive lane will return poison.`。
- **L2716 EN**: Declares TableGen def `int_amdgcn_wave_shuffle`.
  **L2716 CN**: 声明 TableGen def `int_amdgcn_wave_shuffle`。
- **L2717 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<[llvm_any_ty],                    // return type`.
  **L2717 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<[llvm_any_ty],                    // return type`。
- **L2718 EN**: Continues the surrounding expression or declaration: `[LLVMMatchType<0>, llvm_i32_ty],  // arg types`.
  **L2718 CN**: 继续构造周围的表达式或声明：`[LLVMMatchType<0>, llvm_i32_ty],  // arg types`。
- **L2719 EN**: Continues the surrounding expression or declaration: `[IntrConvergent, IntrNoMem]>;     // flags`.
  **L2719 CN**: 继续构造周围的表达式或声明：`[IntrConvergent, IntrNoMem]>;     // flags`。
- **L2720 EN**: Blank line separating nearby declarations or logic blocks.
  **L2720 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2721-2752

````tablegen
//===----------------------------------------------------------------------===//
// CI+ Intrinsics
//===----------------------------------------------------------------------===//

def int_amdgcn_s_dcache_inv_vol :
  ClangBuiltin<"__builtin_amdgcn_s_dcache_inv_vol">,
  DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;

def int_amdgcn_buffer_wbinvl1_vol :
  ClangBuiltin<"__builtin_amdgcn_buffer_wbinvl1_vol">,
  DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;

//===----------------------------------------------------------------------===//
// VI Intrinsics
//===----------------------------------------------------------------------===//

// The llvm.amdgcn.mov.dpp intrinsic represents the mov.dpp operation in AMDGPU.
// This operation is being deprecated and can be replaced with
// llvm.amdgcn.update.dpp.
// llvm.amdgcn.mov.dpp <src> <dpp_ctrl> <row_mask> <bank_mask> <bound_ctrl>
def int_amdgcn_mov_dpp :
  Intrinsic<[llvm_anyint_ty],
            [LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
             llvm_i1_ty],
             [IntrNoMem, IntrConvergent, IntrWillReturn,
             ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>,
             ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree]>;

// The llvm.amdgcn.update.dpp intrinsic represents the update.dpp operation in
// AMDGPU. It takes an old value, a source operand, a DPP control operand, a row
// mask, a bank mask, and a bound control. This operation is equivalent to a
// sequence of v_mov_b32 operations. It is preferred over llvm.amdgcn.mov.dpp
````
- **L2721 EN**: Banner comment marking a file or section boundary.
  **L2721 CN**: 横幅注释，用于标记文件或章节边界。
- **L2722 EN**: Comment explains nearby logic, invariants, or intent: `CI+ Intrinsics`.
  **L2722 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CI+ Intrinsics`。
- **L2723 EN**: Banner comment marking a file or section boundary.
  **L2723 CN**: 横幅注释，用于标记文件或章节边界。
- **L2724 EN**: Blank line separating nearby declarations or logic blocks.
  **L2724 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2725 EN**: Declares TableGen def `int_amdgcn_s_dcache_inv_vol`.
  **L2725 CN**: 声明 TableGen def `int_amdgcn_s_dcache_inv_vol`。
- **L2726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_dcache_inv_vol">,`.
  **L2726 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_dcache_inv_vol">,`。
- **L2727 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L2727 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L2728 EN**: Blank line separating nearby declarations or logic blocks.
  **L2728 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2729 EN**: Declares TableGen def `int_amdgcn_buffer_wbinvl1_vol`.
  **L2729 CN**: 声明 TableGen def `int_amdgcn_buffer_wbinvl1_vol`。
- **L2730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_buffer_wbinvl1_vol">,`.
  **L2730 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_buffer_wbinvl1_vol">,`。
- **L2731 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L2731 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L2732 EN**: Blank line separating nearby declarations or logic blocks.
  **L2732 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2733 EN**: Banner comment marking a file or section boundary.
  **L2733 CN**: 横幅注释，用于标记文件或章节边界。
- **L2734 EN**: Comment explains nearby logic, invariants, or intent: `VI Intrinsics`.
  **L2734 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VI Intrinsics`。
- **L2735 EN**: Banner comment marking a file or section boundary.
  **L2735 CN**: 横幅注释，用于标记文件或章节边界。
- **L2736 EN**: Blank line separating nearby declarations or logic blocks.
  **L2736 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2737 EN**: Comment explains nearby logic, invariants, or intent: `The llvm.amdgcn.mov.dpp intrinsic represents the mov.dpp operation in AMDGPU.`.
  **L2737 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The llvm.amdgcn.mov.dpp intrinsic represents the mov.dpp operation in AMDGPU.`。
- **L2738 EN**: Comment explains nearby logic, invariants, or intent: `This operation is being deprecated and can be replaced with`.
  **L2738 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This operation is being deprecated and can be replaced with`。
- **L2739 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.update.dpp.`.
  **L2739 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.update.dpp.`。
- **L2740 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.mov.dpp <src> <dpp_ctrl> <row_mask> <bank_mask> <bound_ctrl>`.
  **L2740 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.mov.dpp <src> <dpp_ctrl> <row_mask> <bank_mask> <bound_ctrl>`。
- **L2741 EN**: Declares TableGen def `int_amdgcn_mov_dpp`.
  **L2741 CN**: 声明 TableGen def `int_amdgcn_mov_dpp`。
- **L2742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_anyint_ty],`.
  **L2742 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_anyint_ty],`。
- **L2743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L2743 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L2744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i1_ty],`.
  **L2744 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i1_ty],`。
- **L2745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,`.
  **L2745 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,`。
- **L2746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>,`.
  **L2746 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>,`。
- **L2747 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree]>;`.
  **L2747 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree]>;`。
- **L2748 EN**: Blank line separating nearby declarations or logic blocks.
  **L2748 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2749 EN**: Comment explains nearby logic, invariants, or intent: `The llvm.amdgcn.update.dpp intrinsic represents the update.dpp operation in`.
  **L2749 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The llvm.amdgcn.update.dpp intrinsic represents the update.dpp operation in`。
- **L2750 EN**: Comment explains nearby logic, invariants, or intent: `AMDGPU. It takes an old value, a source operand, a DPP control operand, a row`.
  **L2750 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AMDGPU. It takes an old value, a source operand, a DPP control operand, a row`。
- **L2751 EN**: Comment explains nearby logic, invariants, or intent: `mask, a bank mask, and a bound control. This operation is equivalent to a`.
  **L2751 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mask, a bank mask, and a bound control. This operation is equivalent to a`。
- **L2752 EN**: Comment explains nearby logic, invariants, or intent: `sequence of v_mov_b32 operations. It is preferred over llvm.amdgcn.mov.dpp`.
  **L2752 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sequence of v_mov_b32 operations. It is preferred over llvm.amdgcn.mov.dpp`。

### Lines 2753-2784

````tablegen
// for future use.
// llvm.amdgcn.update.dpp <old> <src> <dpp_ctrl> <row_mask> <bank_mask> <bound_ctrl>
// Should be equivalent to:
// v_mov_b32 <dest> <old>
// v_mov_b32 <dest> <src> <dpp_ctrl> <row_mask> <bank_mask> <bound_ctrl>
def int_amdgcn_update_dpp :
  Intrinsic<[llvm_any_ty],
            [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty,
            llvm_i32_ty, llvm_i32_ty, llvm_i1_ty],
             [IntrNoMem, IntrConvergent, IntrWillReturn,
              ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>,
              ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, IntrNoCallback, IntrNoFree]>;

def int_amdgcn_s_dcache_wb :
  ClangBuiltin<"__builtin_amdgcn_s_dcache_wb">,
  Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

def int_amdgcn_s_dcache_wb_vol :
  ClangBuiltin<"__builtin_amdgcn_s_dcache_wb_vol">,
  Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

def int_amdgcn_s_memrealtime :
  ClangBuiltin<"__builtin_amdgcn_s_memrealtime">,
  Intrinsic<[llvm_i64_ty], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.ds.permute <index> <src>
def int_amdgcn_ds_permute :
  ClangBuiltin<"__builtin_amdgcn_ds_permute">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
    [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.ds.bpermute <index> <src>
````
- **L2753 EN**: Comment explains nearby logic, invariants, or intent: `for future use.`.
  **L2753 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for future use.`。
- **L2754 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.update.dpp <old> <src> <dpp_ctrl> <row_mask> <bank_mask> <bound_ctrl>`.
  **L2754 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.update.dpp <old> <src> <dpp_ctrl> <row_mask> <bank_mask> <bound_ctrl>`。
- **L2755 EN**: Comment explains nearby logic, invariants, or intent: `Should be equivalent to:`.
  **L2755 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Should be equivalent to:`。
- **L2756 EN**: Comment explains nearby logic, invariants, or intent: `v_mov_b32 <dest> <old>`.
  **L2756 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v_mov_b32 <dest> <old>`。
- **L2757 EN**: Comment explains nearby logic, invariants, or intent: `v_mov_b32 <dest> <src> <dpp_ctrl> <row_mask> <bank_mask> <bound_ctrl>`.
  **L2757 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v_mov_b32 <dest> <src> <dpp_ctrl> <row_mask> <bank_mask> <bound_ctrl>`。
- **L2758 EN**: Declares TableGen def `int_amdgcn_update_dpp`.
  **L2758 CN**: 声明 TableGen def `int_amdgcn_update_dpp`。
- **L2759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty],`.
  **L2759 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty],`。
- **L2760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty,`.
  **L2760 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty,`。
- **L2761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty, llvm_i1_ty],`.
  **L2761 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty, llvm_i1_ty],`。
- **L2762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,`.
  **L2762 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,`。
- **L2763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>,`.
  **L2763 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>,`。
- **L2764 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, IntrNoCallback, IntrNoFree]>;`.
  **L2764 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, IntrNoCallback, IntrNoFree]>;`。
- **L2765 EN**: Blank line separating nearby declarations or logic blocks.
  **L2765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2766 EN**: Declares TableGen def `int_amdgcn_s_dcache_wb`.
  **L2766 CN**: 声明 TableGen def `int_amdgcn_s_dcache_wb`。
- **L2767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_dcache_wb">,`.
  **L2767 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_dcache_wb">,`。
- **L2768 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2768 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2769 EN**: Blank line separating nearby declarations or logic blocks.
  **L2769 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2770 EN**: Declares TableGen def `int_amdgcn_s_dcache_wb_vol`.
  **L2770 CN**: 声明 TableGen def `int_amdgcn_s_dcache_wb_vol`。
- **L2771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_dcache_wb_vol">,`.
  **L2771 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_dcache_wb_vol">,`。
- **L2772 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2772 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2773 EN**: Blank line separating nearby declarations or logic blocks.
  **L2773 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2774 EN**: Declares TableGen def `int_amdgcn_s_memrealtime`.
  **L2774 CN**: 声明 TableGen def `int_amdgcn_s_memrealtime`。
- **L2775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_memrealtime">,`.
  **L2775 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_memrealtime">,`。
- **L2776 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2776 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2777 EN**: Blank line separating nearby declarations or logic blocks.
  **L2777 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2778 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.permute <index> <src>`.
  **L2778 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.permute <index> <src>`。
- **L2779 EN**: Declares TableGen def `int_amdgcn_ds_permute`.
  **L2779 CN**: 声明 TableGen def `int_amdgcn_ds_permute`。
- **L2780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_permute">,`.
  **L2780 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_permute">,`。
- **L2781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L2781 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L2782 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2782 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2783 EN**: Blank line separating nearby declarations or logic blocks.
  **L2783 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2784 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.bpermute <index> <src>`.
  **L2784 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.bpermute <index> <src>`。

### Lines 2785-2816

````tablegen
def int_amdgcn_ds_bpermute :
  ClangBuiltin<"__builtin_amdgcn_ds_bpermute">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
     [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.perm <src0> <src1> <selector>
def int_amdgcn_perm :
  ClangBuiltin<"__builtin_amdgcn_perm">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
     [IntrNoMem, IntrSpeculatable, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

//===----------------------------------------------------------------------===//
// GFX9 Intrinsics
//===----------------------------------------------------------------------===//

/// This is a general-purpose intrinsic for all operations that take a pointer
/// a base location in LDS, and a data size and use it to perform a gather to LDS.
/// This allows abstracting over both global pointers (address space 1) and
/// the buffer-resource-wrapper pointers (address space 7 and 9).
/// TODO: add support for address space 5 and scratch_load_lds.
class AMDGPULoadToLDS :
  Intrinsic <
    [],
    [llvm_anyptr_ty,                    // Base pointer to load from. Varies per lane.
     LLVMQualPointerType<3>,            // LDS base pointer to store to. Must be wave-uniform.
     llvm_i32_ty,                       // Data byte size: 1/2/4 (/12/16 for gfx950)
     llvm_i32_ty,                       // imm offset (applied to both input and LDS address)
     llvm_i32_ty],                      // auxiliary data (imm, cachepolicy (bit 0 = sc0,
                                        //                                   bit 1 = sc1,
                                        //                                   bit 4 = scc))
    [IntrWillReturn, IntrArgMemOnly,
     NoCapture<ArgIndex<0>>, ReadOnly<ArgIndex<0>>,
````
- **L2785 EN**: Declares TableGen def `int_amdgcn_ds_bpermute`.
  **L2785 CN**: 声明 TableGen def `int_amdgcn_ds_bpermute`。
- **L2786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_bpermute">,`.
  **L2786 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_bpermute">,`。
- **L2787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L2787 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L2788 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2788 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2789 EN**: Blank line separating nearby declarations or logic blocks.
  **L2789 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2790 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.perm <src0> <src1> <selector>`.
  **L2790 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.perm <src0> <src1> <selector>`。
- **L2791 EN**: Declares TableGen def `int_amdgcn_perm`.
  **L2791 CN**: 声明 TableGen def `int_amdgcn_perm`。
- **L2792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_perm">,`.
  **L2792 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_perm">,`。
- **L2793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L2793 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L2794 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2794 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2795 EN**: Blank line separating nearby declarations or logic blocks.
  **L2795 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2796 EN**: Banner comment marking a file or section boundary.
  **L2796 CN**: 横幅注释，用于标记文件或章节边界。
- **L2797 EN**: Comment explains nearby logic, invariants, or intent: `GFX9 Intrinsics`.
  **L2797 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GFX9 Intrinsics`。
- **L2798 EN**: Banner comment marking a file or section boundary.
  **L2798 CN**: 横幅注释，用于标记文件或章节边界。
- **L2799 EN**: Blank line separating nearby declarations or logic blocks.
  **L2799 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2800 EN**: Comment explains nearby logic, invariants, or intent: `This is a general-purpose intrinsic for all operations that take a pointer`.
  **L2800 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is a general-purpose intrinsic for all operations that take a pointer`。
- **L2801 EN**: Comment explains nearby logic, invariants, or intent: `a base location in LDS, and a data size and use it to perform a gather to LDS.`.
  **L2801 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a base location in LDS, and a data size and use it to perform a gather to LDS.`。
- **L2802 EN**: Comment explains nearby logic, invariants, or intent: `This allows abstracting over both global pointers (address space 1) and`.
  **L2802 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This allows abstracting over both global pointers (address space 1) and`。
- **L2803 EN**: Comment explains nearby logic, invariants, or intent: `the buffer-resource-wrapper pointers (address space 7 and 9).`.
  **L2803 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the buffer-resource-wrapper pointers (address space 7 and 9).`。
- **L2804 EN**: Comment records a pending task or caution: `TODO: add support for address space 5 and scratch_load_lds.`.
  **L2804 CN**: 注释记录了待办事项或注意点：`TODO: add support for address space 5 and scratch_load_lds.`。
- **L2805 EN**: Declares class `AMDGPULoadToLDS`.
  **L2805 CN**: 声明 class `AMDGPULoadToLDS`。
- **L2806 EN**: Continues the surrounding expression or declaration: `Intrinsic <`.
  **L2806 CN**: 继续构造周围的表达式或声明：`Intrinsic <`。
- **L2807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L2807 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L2808 EN**: Continues the surrounding expression or declaration: `[llvm_anyptr_ty,                    // Base pointer to load from. Varies per lane.`.
  **L2808 CN**: 继续构造周围的表达式或声明：`[llvm_anyptr_ty,                    // Base pointer to load from. Varies per lane.`。
- **L2809 EN**: Continues the surrounding expression or declaration: `LLVMQualPointerType<3>,            // LDS base pointer to store to. Must be wave-uniform.`.
  **L2809 CN**: 继续构造周围的表达式或声明：`LLVMQualPointerType<3>,            // LDS base pointer to store to. Must be wave-uniform.`。
- **L2810 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,                       // Data byte size: 1/2/4 (/12/16 for gfx950)`.
  **L2810 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,                       // Data byte size: 1/2/4 (/12/16 for gfx950)`。
- **L2811 EN**: Continues logic associated with callable symbol `offset`.
  **L2811 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L2812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],                      // auxiliary data (imm, cachepolicy (bit 0 = sc0,`.
  **L2812 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],                      // auxiliary data (imm, cachepolicy (bit 0 = sc0,`。
- **L2813 EN**: Comment explains nearby logic, invariants, or intent: `bit 1 = sc1,`.
  **L2813 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 1 = sc1,`。
- **L2814 EN**: Comment explains nearby logic, invariants, or intent: `bit 4 = scc))`.
  **L2814 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 4 = scc))`。
- **L2815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWillReturn, IntrArgMemOnly,`.
  **L2815 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWillReturn, IntrArgMemOnly,`。
- **L2816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NoCapture<ArgIndex<0>>, ReadOnly<ArgIndex<0>>,`.
  **L2816 CN**: 继续一个多行参数列表、初始化器或聚合项：`NoCapture<ArgIndex<0>>, ReadOnly<ArgIndex<0>>,`。

### Lines 2817-2848

````tablegen
     NoCapture<ArgIndex<1>>, WriteOnly<ArgIndex<1>>,
     ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree],
     "", [SDNPMemOperand]>;
def int_amdgcn_load_to_lds : AMDGPULoadToLDS;
def int_amdgcn_load_async_to_lds : AMDGPULoadToLDS;

class AMDGPUGlobalLoadLDS : Intrinsic<
          [],
          [LLVMQualPointerType<1>, // Base global pointer to load from
           LLVMQualPointerType<3>, // LDS base pointer to store to
           llvm_i32_ty,            // Data byte size: 1/2/4 (/12/16 for gfx950)
           llvm_i32_ty,  // imm offset (applied to both global and LDS address)
           llvm_i32_ty], // auxiliary data (imm, cachepolicy (bit 0 = sc0,
                         //                                   bit 1 = sc1,
                         //                                   bit 4 = scc,
                         //                                   bit 31 = volatile
                         //                                   (compiler
                         //                                   implemented)))
          [IntrWillReturn, NoCapture<ArgIndex<0>>, NoCapture<ArgIndex<1>>,
           ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>,
           IntrNoCallback, IntrNoFree],
          "", [SDNPMemOperand]>;
def int_amdgcn_global_load_lds : AMDGPUGlobalLoadLDS, ClangBuiltin<"__builtin_amdgcn_global_load_lds">;
def int_amdgcn_global_load_async_lds : AMDGPUGlobalLoadLDS, ClangBuiltin<"__builtin_amdgcn_global_load_async_lds">;

// This is IntrHasSideEffects because it reads from a volatile hardware register.
def int_amdgcn_pops_exiting_wave_id :
  DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrHasSideEffects]>;

// Sets a marker in the stream of async requests. Modelled as InaccessibleMem.
def int_amdgcn_asyncmark : ClangBuiltin<"__builtin_amdgcn_asyncmark">,
  Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;
````
- **L2817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NoCapture<ArgIndex<1>>, WriteOnly<ArgIndex<1>>,`.
  **L2817 CN**: 继续一个多行参数列表、初始化器或聚合项：`NoCapture<ArgIndex<1>>, WriteOnly<ArgIndex<1>>,`。
- **L2818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree],`.
  **L2818 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree],`。
- **L2819 EN**: Executes a standalone statement or declaration: `"", [SDNPMemOperand]>;`.
  **L2819 CN**: 执行一条独立语句或声明：`"", [SDNPMemOperand]>;`。
- **L2820 EN**: Declares TableGen def `int_amdgcn_load_to_lds`.
  **L2820 CN**: 声明 TableGen def `int_amdgcn_load_to_lds`。
- **L2821 EN**: Declares TableGen def `int_amdgcn_load_async_to_lds`.
  **L2821 CN**: 声明 TableGen def `int_amdgcn_load_async_to_lds`。
- **L2822 EN**: Blank line separating nearby declarations or logic blocks.
  **L2822 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2823 EN**: Declares class `AMDGPUGlobalLoadLDS`.
  **L2823 CN**: 声明 class `AMDGPUGlobalLoadLDS`。
- **L2824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L2824 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L2825 EN**: Continues the surrounding expression or declaration: `[LLVMQualPointerType<1>, // Base global pointer to load from`.
  **L2825 CN**: 继续构造周围的表达式或声明：`[LLVMQualPointerType<1>, // Base global pointer to load from`。
- **L2826 EN**: Continues the surrounding expression or declaration: `LLVMQualPointerType<3>, // LDS base pointer to store to`.
  **L2826 CN**: 继续构造周围的表达式或声明：`LLVMQualPointerType<3>, // LDS base pointer to store to`。
- **L2827 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,            // Data byte size: 1/2/4 (/12/16 for gfx950)`.
  **L2827 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,            // Data byte size: 1/2/4 (/12/16 for gfx950)`。
- **L2828 EN**: Continues logic associated with callable symbol `offset`.
  **L2828 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L2829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty], // auxiliary data (imm, cachepolicy (bit 0 = sc0,`.
  **L2829 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty], // auxiliary data (imm, cachepolicy (bit 0 = sc0,`。
- **L2830 EN**: Comment explains nearby logic, invariants, or intent: `bit 1 = sc1,`.
  **L2830 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 1 = sc1,`。
- **L2831 EN**: Comment explains nearby logic, invariants, or intent: `bit 4 = scc,`.
  **L2831 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 4 = scc,`。
- **L2832 EN**: Comment explains nearby logic, invariants, or intent: `bit 31 = volatile`.
  **L2832 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit 31 = volatile`。
- **L2833 EN**: Comment explains nearby logic, invariants, or intent: `(compiler`.
  **L2833 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(compiler`。
- **L2834 EN**: Comment explains nearby logic, invariants, or intent: `implemented)))`.
  **L2834 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`implemented)))`。
- **L2835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWillReturn, NoCapture<ArgIndex<0>>, NoCapture<ArgIndex<1>>,`.
  **L2835 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWillReturn, NoCapture<ArgIndex<0>>, NoCapture<ArgIndex<1>>,`。
- **L2836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>,`.
  **L2836 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>,`。
- **L2837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrNoCallback, IntrNoFree],`.
  **L2837 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrNoCallback, IntrNoFree],`。
- **L2838 EN**: Executes a standalone statement or declaration: `"", [SDNPMemOperand]>;`.
  **L2838 CN**: 执行一条独立语句或声明：`"", [SDNPMemOperand]>;`。
- **L2839 EN**: Declares TableGen def `int_amdgcn_global_load_lds`.
  **L2839 CN**: 声明 TableGen def `int_amdgcn_global_load_lds`。
- **L2840 EN**: Declares TableGen def `int_amdgcn_global_load_async_lds`.
  **L2840 CN**: 声明 TableGen def `int_amdgcn_global_load_async_lds`。
- **L2841 EN**: Blank line separating nearby declarations or logic blocks.
  **L2841 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2842 EN**: Comment explains nearby logic, invariants, or intent: `This is IntrHasSideEffects because it reads from a volatile hardware register.`.
  **L2842 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is IntrHasSideEffects because it reads from a volatile hardware register.`。
- **L2843 EN**: Declares TableGen def `int_amdgcn_pops_exiting_wave_id`.
  **L2843 CN**: 声明 TableGen def `int_amdgcn_pops_exiting_wave_id`。
- **L2844 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L2844 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L2845 EN**: Blank line separating nearby declarations or logic blocks.
  **L2845 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2846 EN**: Comment explains nearby logic, invariants, or intent: `Sets a marker in the stream of async requests. Modelled as InaccessibleMem.`.
  **L2846 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sets a marker in the stream of async requests. Modelled as InaccessibleMem.`。
- **L2847 EN**: Declares TableGen def `int_amdgcn_asyncmark`.
  **L2847 CN**: 声明 TableGen def `int_amdgcn_asyncmark`。
- **L2848 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L2848 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;`。

### Lines 2849-2880

````tablegen

// Waits until the Nth previous marker is completed, if it exists.
def int_amdgcn_wait_asyncmark :
    ClangBuiltin<"__builtin_amdgcn_wait_asyncmark">,
    Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects]>;

//===----------------------------------------------------------------------===//
// GFX10 Intrinsics
//===----------------------------------------------------------------------===//

// llvm.amdgcn.permlane16 <old> <src0> <src1> <src2> <fi> <bound_control>
def int_amdgcn_permlane16 :
  Intrinsic<[llvm_any_ty],
            [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn,
             ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.permlanex16 <old> <src0> <src1> <src2> <fi> <bound_control>
def int_amdgcn_permlanex16 :
  Intrinsic<[llvm_any_ty],
            [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn,
             ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.mov.dpp8 <src> <sel>
// <sel> is a 32-bit constant whose high 8 bits must be zero which selects
// the lanes to read from.
def int_amdgcn_mov_dpp8 :
  Intrinsic<[llvm_any_ty],
            [LLVMMatchType<0>, llvm_i32_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn,
             ImmArg<ArgIndex<1>>, IntrNoCallback, IntrNoFree]>;
````
- **L2849 EN**: Blank line separating nearby declarations or logic blocks.
  **L2849 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2850 EN**: Comment explains nearby logic, invariants, or intent: `Waits until the Nth previous marker is completed, if it exists.`.
  **L2850 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Waits until the Nth previous marker is completed, if it exists.`。
- **L2851 EN**: Declares TableGen def `int_amdgcn_wait_asyncmark`.
  **L2851 CN**: 声明 TableGen def `int_amdgcn_wait_asyncmark`。
- **L2852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_wait_asyncmark">,`.
  **L2852 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_wait_asyncmark">,`。
- **L2853 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects]>;`.
  **L2853 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem, IntrHasSideEffects]>;`。
- **L2854 EN**: Blank line separating nearby declarations or logic blocks.
  **L2854 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2855 EN**: Banner comment marking a file or section boundary.
  **L2855 CN**: 横幅注释，用于标记文件或章节边界。
- **L2856 EN**: Comment explains nearby logic, invariants, or intent: `GFX10 Intrinsics`.
  **L2856 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GFX10 Intrinsics`。
- **L2857 EN**: Banner comment marking a file or section boundary.
  **L2857 CN**: 横幅注释，用于标记文件或章节边界。
- **L2858 EN**: Blank line separating nearby declarations or logic blocks.
  **L2858 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2859 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlane16 <old> <src0> <src1> <src2> <fi> <bound_control>`.
  **L2859 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlane16 <old> <src0> <src1> <src2> <fi> <bound_control>`。
- **L2860 EN**: Declares TableGen def `int_amdgcn_permlane16`.
  **L2860 CN**: 声明 TableGen def `int_amdgcn_permlane16`。
- **L2861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty],`.
  **L2861 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty],`。
- **L2862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],`.
  **L2862 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],`。
- **L2863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,`.
  **L2863 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,`。
- **L2864 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, IntrNoCallback, IntrNoFree]>;`.
  **L2864 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, IntrNoCallback, IntrNoFree]>;`。
- **L2865 EN**: Blank line separating nearby declarations or logic blocks.
  **L2865 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2866 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlanex16 <old> <src0> <src1> <src2> <fi> <bound_control>`.
  **L2866 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlanex16 <old> <src0> <src1> <src2> <fi> <bound_control>`。
- **L2867 EN**: Declares TableGen def `int_amdgcn_permlanex16`.
  **L2867 CN**: 声明 TableGen def `int_amdgcn_permlanex16`。
- **L2868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty],`.
  **L2868 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty],`。
- **L2869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],`.
  **L2869 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],`。
- **L2870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,`.
  **L2870 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,`。
- **L2871 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, IntrNoCallback, IntrNoFree]>;`.
  **L2871 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, IntrNoCallback, IntrNoFree]>;`。
- **L2872 EN**: Blank line separating nearby declarations or logic blocks.
  **L2872 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2873 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.mov.dpp8 <src> <sel>`.
  **L2873 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.mov.dpp8 <src> <sel>`。
- **L2874 EN**: Comment explains nearby logic, invariants, or intent: `<sel> is a 32-bit constant whose high 8 bits must be zero which selects`.
  **L2874 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<sel> is a 32-bit constant whose high 8 bits must be zero which selects`。
- **L2875 EN**: Comment explains nearby logic, invariants, or intent: `the lanes to read from.`.
  **L2875 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the lanes to read from.`。
- **L2876 EN**: Declares TableGen def `int_amdgcn_mov_dpp8`.
  **L2876 CN**: 声明 TableGen def `int_amdgcn_mov_dpp8`。
- **L2877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty],`.
  **L2877 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty],`。
- **L2878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty],`.
  **L2878 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty],`。
- **L2879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,`.
  **L2879 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,`。
- **L2880 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<1>>, IntrNoCallback, IntrNoFree]>;`.
  **L2880 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<1>>, IntrNoCallback, IntrNoFree]>;`。

### Lines 2881-2912

````tablegen

def int_amdgcn_s_get_waveid_in_workgroup :
  ClangBuiltin<"__builtin_amdgcn_s_get_waveid_in_workgroup">,
  Intrinsic<[llvm_i32_ty], [],
    [NoUndef<RetIndex>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

class AMDGPUAtomicRtn<LLVMType vt, LLVMType pt = llvm_anyptr_ty> : Intrinsic <
  [vt],
  [pt,  // vaddr
   vt], // vdata(VGPR)
  [IntrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>, IntrNoCallback, IntrNoFree], "",
  [SDNPMemOperand]>;

// uint4 llvm.amdgcn.image.bvh.intersect.ray <node_ptr>, <ray_extent>, <ray_origin>,
//                                           <ray_dir>, <ray_inv_dir>, <texture_descr>
// <node_ptr> is i32 or i64.
// <ray_dir> and <ray_inv_dir> are both v3f16 or both v3f32.
def int_amdgcn_image_bvh_intersect_ray :
  DefaultAttrsIntrinsic<[llvm_v4i32_ty],
            [llvm_anyint_ty, llvm_float_ty, llvm_v3f32_ty, llvm_anyvector_ty,
             LLVMMatchType<1>, llvm_v4i32_ty],
            [IntrReadMem]>;

//===----------------------------------------------------------------------===//
// GFX11 Intrinsics
//===----------------------------------------------------------------------===//

// llvm.amdgcn.permlane64 <src0>
def int_amdgcn_permlane64 :
  Intrinsic<[llvm_any_ty], [LLVMMatchType<0>],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

````
- **L2881 EN**: Blank line separating nearby declarations or logic blocks.
  **L2881 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2882 EN**: Declares TableGen def `int_amdgcn_s_get_waveid_in_workgroup`.
  **L2882 CN**: 声明 TableGen def `int_amdgcn_s_get_waveid_in_workgroup`。
- **L2883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_get_waveid_in_workgroup">,`.
  **L2883 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_get_waveid_in_workgroup">,`。
- **L2884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [],`.
  **L2884 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [],`。
- **L2885 EN**: Executes a standalone statement or declaration: `[NoUndef<RetIndex>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2885 CN**: 执行一条独立语句或声明：`[NoUndef<RetIndex>, IntrNoMem, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2886 EN**: Blank line separating nearby declarations or logic blocks.
  **L2886 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2887 EN**: Declares class `AMDGPUAtomicRtn<LLVMType`.
  **L2887 CN**: 声明 class `AMDGPUAtomicRtn<LLVMType`。
- **L2888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[vt],`.
  **L2888 CN**: 继续一个多行参数列表、初始化器或聚合项：`[vt],`。
- **L2889 EN**: Continues the surrounding expression or declaration: `[pt,  // vaddr`.
  **L2889 CN**: 继续构造周围的表达式或声明：`[pt,  // vaddr`。
- **L2890 EN**: Continues logic associated with callable symbol `vdata`.
  **L2890 CN**: 继续与可调用符号 `vdata` 相关的逻辑。
- **L2891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>, IntrNoCallback, IntrNoFree], "",`.
  **L2891 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>, IntrNoCallback, IntrNoFree], "",`。
- **L2892 EN**: Executes a standalone statement or declaration: `[SDNPMemOperand]>;`.
  **L2892 CN**: 执行一条独立语句或声明：`[SDNPMemOperand]>;`。
- **L2893 EN**: Blank line separating nearby declarations or logic blocks.
  **L2893 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2894 EN**: Comment explains nearby logic, invariants, or intent: `uint4 llvm.amdgcn.image.bvh.intersect.ray <node_ptr>, <ray_extent>, <ray_origin>,`.
  **L2894 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`uint4 llvm.amdgcn.image.bvh.intersect.ray <node_ptr>, <ray_extent>, <ray_origin>,`。
- **L2895 EN**: Comment explains nearby logic, invariants, or intent: `<ray_dir>, <ray_inv_dir>, <texture_descr>`.
  **L2895 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<ray_dir>, <ray_inv_dir>, <texture_descr>`。
- **L2896 EN**: Comment explains nearby logic, invariants, or intent: `<node_ptr> is i32 or i64.`.
  **L2896 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<node_ptr> is i32 or i64.`。
- **L2897 EN**: Comment explains nearby logic, invariants, or intent: `<ray_dir> and <ray_inv_dir> are both v3f16 or both v3f32.`.
  **L2897 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<ray_dir> and <ray_inv_dir> are both v3f16 or both v3f32.`。
- **L2898 EN**: Declares TableGen def `int_amdgcn_image_bvh_intersect_ray`.
  **L2898 CN**: 声明 TableGen def `int_amdgcn_image_bvh_intersect_ray`。
- **L2899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty],`.
  **L2899 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty],`。
- **L2900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyint_ty, llvm_float_ty, llvm_v3f32_ty, llvm_anyvector_ty,`.
  **L2900 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyint_ty, llvm_float_ty, llvm_v3f32_ty, llvm_anyvector_ty,`。
- **L2901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, llvm_v4i32_ty],`.
  **L2901 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, llvm_v4i32_ty],`。
- **L2902 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L2902 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L2903 EN**: Blank line separating nearby declarations or logic blocks.
  **L2903 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2904 EN**: Banner comment marking a file or section boundary.
  **L2904 CN**: 横幅注释，用于标记文件或章节边界。
- **L2905 EN**: Comment explains nearby logic, invariants, or intent: `GFX11 Intrinsics`.
  **L2905 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GFX11 Intrinsics`。
- **L2906 EN**: Banner comment marking a file or section boundary.
  **L2906 CN**: 横幅注释，用于标记文件或章节边界。
- **L2907 EN**: Blank line separating nearby declarations or logic blocks.
  **L2907 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2908 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlane64 <src0>`.
  **L2908 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlane64 <src0>`。
- **L2909 EN**: Declares TableGen def `int_amdgcn_permlane64`.
  **L2909 CN**: 声明 TableGen def `int_amdgcn_permlane64`。
- **L2910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty], [LLVMMatchType<0>],`.
  **L2910 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty], [LLVMMatchType<0>],`。
- **L2911 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L2911 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L2912 EN**: Blank line separating nearby declarations or logic blocks.
  **L2912 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2913-2944

````tablegen
def int_amdgcn_ds_add_gs_reg_rtn :
  ClangBuiltin<"__builtin_amdgcn_ds_add_gs_reg_rtn">,
  Intrinsic<[llvm_anyint_ty], [llvm_i32_ty, llvm_i32_ty],
            [ImmArg<ArgIndex<1>>, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree],
            "", [SDNPMemOperand]>;

def int_amdgcn_ds_sub_gs_reg_rtn :
  ClangBuiltin<"__builtin_amdgcn_ds_sub_gs_reg_rtn">,
  Intrinsic<[llvm_anyint_ty], [llvm_i32_ty, llvm_i32_ty],
            [ImmArg<ArgIndex<1>>, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree],
            "", [SDNPMemOperand]>;

class IntDSBVHStackRtn<LLVMType vdst, LLVMType data1> :
  Intrinsic<
    [vdst, llvm_i32_ty], // %vdst, %addr
    [
      llvm_i32_ty,   // %addr
      llvm_i32_ty,   // %data0
      data1,         // %data1
      llvm_i32_ty,   // %offset
    ],
    [ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree]
  >;

def int_amdgcn_ds_bvh_stack_rtn : IntDSBVHStackRtn<vdst  = llvm_i32_ty,
                                                   data1 = llvm_v4i32_ty>;

// Emit s_wait_event instruction. Note that between gfx11 and gfx12,
// the bit for the export_ready event changed. gfx11 expects bit 0 to
// be 0, and gfx12 expects bit 1 to be 0. Thus, an immediate value of
// 2 can be used as the universal value for export_ready.
def int_amdgcn_s_wait_event :
````
- **L2913 EN**: Declares TableGen def `int_amdgcn_ds_add_gs_reg_rtn`.
  **L2913 CN**: 声明 TableGen def `int_amdgcn_ds_add_gs_reg_rtn`。
- **L2914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_add_gs_reg_rtn">,`.
  **L2914 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_add_gs_reg_rtn">,`。
- **L2915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_anyint_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L2915 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_anyint_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L2916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<1>>, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree],`.
  **L2916 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<1>>, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree],`。
- **L2917 EN**: Executes a standalone statement or declaration: `"", [SDNPMemOperand]>;`.
  **L2917 CN**: 执行一条独立语句或声明：`"", [SDNPMemOperand]>;`。
- **L2918 EN**: Blank line separating nearby declarations or logic blocks.
  **L2918 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2919 EN**: Declares TableGen def `int_amdgcn_ds_sub_gs_reg_rtn`.
  **L2919 CN**: 声明 TableGen def `int_amdgcn_ds_sub_gs_reg_rtn`。
- **L2920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_sub_gs_reg_rtn">,`.
  **L2920 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_sub_gs_reg_rtn">,`。
- **L2921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_anyint_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L2921 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_anyint_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L2922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<1>>, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree],`.
  **L2922 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<1>>, IntrHasSideEffects, IntrWillReturn, IntrNoCallback, IntrNoFree],`。
- **L2923 EN**: Executes a standalone statement or declaration: `"", [SDNPMemOperand]>;`.
  **L2923 CN**: 执行一条独立语句或声明：`"", [SDNPMemOperand]>;`。
- **L2924 EN**: Blank line separating nearby declarations or logic blocks.
  **L2924 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2925 EN**: Declares class `IntDSBVHStackRtn<LLVMType`.
  **L2925 CN**: 声明 class `IntDSBVHStackRtn<LLVMType`。
- **L2926 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L2926 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L2927 EN**: Continues the surrounding expression or declaration: `[vdst, llvm_i32_ty], // %vdst, %addr`.
  **L2927 CN**: 继续构造周围的表达式或声明：`[vdst, llvm_i32_ty], // %vdst, %addr`。
- **L2928 EN**: Continues the surrounding expression or declaration: `[`.
  **L2928 CN**: 继续构造周围的表达式或声明：`[`。
- **L2929 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // %addr`.
  **L2929 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // %addr`。
- **L2930 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // %data0`.
  **L2930 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // %data0`。
- **L2931 EN**: Continues the surrounding expression or declaration: `data1,         // %data1`.
  **L2931 CN**: 继续构造周围的表达式或声明：`data1,         // %data1`。
- **L2932 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // %offset`.
  **L2932 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // %offset`。
- **L2933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L2933 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L2934 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L2934 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L2935 EN**: Executes a standalone statement or declaration: `>;`.
  **L2935 CN**: 执行一条独立语句或声明：`>;`。
- **L2936 EN**: Blank line separating nearby declarations or logic blocks.
  **L2936 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2937 EN**: Declares TableGen def `int_amdgcn_ds_bvh_stack_rtn`.
  **L2937 CN**: 声明 TableGen def `int_amdgcn_ds_bvh_stack_rtn`。
- **L2938 EN**: Executes a standalone statement or declaration: `data1 = llvm_v4i32_ty>;`.
  **L2938 CN**: 执行一条独立语句或声明：`data1 = llvm_v4i32_ty>;`。
- **L2939 EN**: Blank line separating nearby declarations or logic blocks.
  **L2939 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2940 EN**: Comment explains nearby logic, invariants, or intent: `Emit s_wait_event instruction. Note that between gfx11 and gfx12,`.
  **L2940 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Emit s_wait_event instruction. Note that between gfx11 and gfx12,`。
- **L2941 EN**: Comment explains nearby logic, invariants, or intent: `the bit for the export_ready event changed. gfx11 expects bit 0 to`.
  **L2941 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the bit for the export_ready event changed. gfx11 expects bit 0 to`。
- **L2942 EN**: Comment explains nearby logic, invariants, or intent: `be 0, and gfx12 expects bit 1 to be 0. Thus, an immediate value of`.
  **L2942 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`be 0, and gfx12 expects bit 1 to be 0. Thus, an immediate value of`。
- **L2943 EN**: Comment explains nearby logic, invariants, or intent: `2 can be used as the universal value for export_ready.`.
  **L2943 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2 can be used as the universal value for export_ready.`。
- **L2944 EN**: Declares TableGen def `int_amdgcn_s_wait_event`.
  **L2944 CN**: 声明 TableGen def `int_amdgcn_s_wait_event`。

### Lines 2945-2976

````tablegen
  ClangBuiltin<"__builtin_amdgcn_s_wait_event">,
  Intrinsic<[], [llvm_i16_ty],
  [ImmArg<ArgIndex<0>>, IntrNoMem, IntrNoCallback, IntrNoFree,
   IntrHasSideEffects, IntrWillReturn]
>;

// Emits same instruction as s_wait_event, with a hardcoded immediate
// value. FIXME: This should be removed
def int_amdgcn_s_wait_event_export_ready :
  ClangBuiltin<"__builtin_amdgcn_s_wait_event_export_ready">,
  Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn,
                     IntrNoCallback, IntrNoFree]
>;

// WMMA (Wave Matrix Multiply-Accumulate) intrinsics
//
// These operations perform a matrix multiplication and accumulation of
// the form: D = A * B + C .

class AMDGPUWmmaIntrinsic<LLVMType AB, LLVMType CD> :
  Intrinsic<
    [CD], // %D
    [
      AB,               // %A
      LLVMMatchType<1>, // %B
      LLVMMatchType<0>, // %C
    ],
    [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree,
     IntrNoCreateUndefOrPoison]
>;

class AMDGPUWmmaIntrinsicOPSEL<LLVMType AB, LLVMType CD> :
````
- **L2945 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_wait_event">,`.
  **L2945 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_wait_event">,`。
- **L2946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i16_ty],`.
  **L2946 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i16_ty],`。
- **L2947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<0>>, IntrNoMem, IntrNoCallback, IntrNoFree,`.
  **L2947 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<0>>, IntrNoMem, IntrNoCallback, IntrNoFree,`。
- **L2948 EN**: Continues the surrounding expression or declaration: `IntrHasSideEffects, IntrWillReturn]`.
  **L2948 CN**: 继续构造周围的表达式或声明：`IntrHasSideEffects, IntrWillReturn]`。
- **L2949 EN**: Executes a standalone statement or declaration: `>;`.
  **L2949 CN**: 执行一条独立语句或声明：`>;`。
- **L2950 EN**: Blank line separating nearby declarations or logic blocks.
  **L2950 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2951 EN**: Comment explains nearby logic, invariants, or intent: `Emits same instruction as s_wait_event, with a hardcoded immediate`.
  **L2951 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Emits same instruction as s_wait_event, with a hardcoded immediate`。
- **L2952 EN**: Comment records a pending task or caution: `value. FIXME: This should be removed`.
  **L2952 CN**: 注释记录了待办事项或注意点：`value. FIXME: This should be removed`。
- **L2953 EN**: Declares TableGen def `int_amdgcn_s_wait_event_export_ready`.
  **L2953 CN**: 声明 TableGen def `int_amdgcn_s_wait_event_export_ready`。
- **L2954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_wait_event_export_ready">,`.
  **L2954 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_wait_event_export_ready">,`。
- **L2955 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn,`.
  **L2955 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrWillReturn,`。
- **L2956 EN**: Continues the surrounding expression or declaration: `IntrNoCallback, IntrNoFree]`.
  **L2956 CN**: 继续构造周围的表达式或声明：`IntrNoCallback, IntrNoFree]`。
- **L2957 EN**: Executes a standalone statement or declaration: `>;`.
  **L2957 CN**: 执行一条独立语句或声明：`>;`。
- **L2958 EN**: Blank line separating nearby declarations or logic blocks.
  **L2958 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2959 EN**: Comment explains nearby logic, invariants, or intent: `WMMA (Wave Matrix Multiply-Accumulate) intrinsics`.
  **L2959 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA (Wave Matrix Multiply-Accumulate) intrinsics`。
- **L2960 EN**: Separator comment used for visual grouping.
  **L2960 CN**: 用于视觉分组的分隔注释。
- **L2961 EN**: Comment explains nearby logic, invariants, or intent: `These operations perform a matrix multiplication and accumulation of`.
  **L2961 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These operations perform a matrix multiplication and accumulation of`。
- **L2962 EN**: Comment explains nearby logic, invariants, or intent: `the form: D = A * B + C .`.
  **L2962 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the form: D = A * B + C .`。
- **L2963 EN**: Blank line separating nearby declarations or logic blocks.
  **L2963 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2964 EN**: Declares class `AMDGPUWmmaIntrinsic<LLVMType`.
  **L2964 CN**: 声明 class `AMDGPUWmmaIntrinsic<LLVMType`。
- **L2965 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L2965 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L2966 EN**: Continues the surrounding expression or declaration: `[CD], // %D`.
  **L2966 CN**: 继续构造周围的表达式或声明：`[CD], // %D`。
- **L2967 EN**: Continues the surrounding expression or declaration: `[`.
  **L2967 CN**: 继续构造周围的表达式或声明：`[`。
- **L2968 EN**: Continues the surrounding expression or declaration: `AB,               // %A`.
  **L2968 CN**: 继续构造周围的表达式或声明：`AB,               // %A`。
- **L2969 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<1>, // %B`.
  **L2969 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<1>, // %B`。
- **L2970 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L2970 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L2971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L2971 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L2972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree,`.
  **L2972 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree,`。
- **L2973 EN**: Continues the surrounding expression or declaration: `IntrNoCreateUndefOrPoison]`.
  **L2973 CN**: 继续构造周围的表达式或声明：`IntrNoCreateUndefOrPoison]`。
- **L2974 EN**: Executes a standalone statement or declaration: `>;`.
  **L2974 CN**: 执行一条独立语句或声明：`>;`。
- **L2975 EN**: Blank line separating nearby declarations or logic blocks.
  **L2975 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2976 EN**: Declares class `AMDGPUWmmaIntrinsicOPSEL<LLVMType`.
  **L2976 CN**: 声明 class `AMDGPUWmmaIntrinsicOPSEL<LLVMType`。

### Lines 2977-3008

````tablegen
  Intrinsic<
    [CD], // %D
    [
      AB,               // %A
      LLVMMatchType<1>, // %B
      LLVMMatchType<0>, // %C
      llvm_i1_ty,       // %high (op_sel) for GFX11, 0 for GFX12
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree,
     IntrNoCreateUndefOrPoison]
>;

class AMDGPUWmmaIntrinsicIU<LLVMType AB, LLVMType CD> :
  Intrinsic<
    [CD], // %D
    [
      llvm_i1_ty,       // %A_sign
      AB,               // %A
      llvm_i1_ty,       // %B_sign
      LLVMMatchType<1>, // %B
      LLVMMatchType<0>, // %C
      llvm_i1_ty,       // %clamp
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree,
     IntrNoCreateUndefOrPoison]
>;

// WMMA GFX11Only

// The OPSEL intrinsics read from and write to one half of the registers, selected by the op_sel bit.
// The tied versions of the f16/bf16 wmma intrinsics tie the destination matrix registers to the input accumulator registers.
// The content of the other 16-bit half is preserved from the input.
````
- **L2977 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L2977 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L2978 EN**: Continues the surrounding expression or declaration: `[CD], // %D`.
  **L2978 CN**: 继续构造周围的表达式或声明：`[CD], // %D`。
- **L2979 EN**: Continues the surrounding expression or declaration: `[`.
  **L2979 CN**: 继续构造周围的表达式或声明：`[`。
- **L2980 EN**: Continues the surrounding expression or declaration: `AB,               // %A`.
  **L2980 CN**: 继续构造周围的表达式或声明：`AB,               // %A`。
- **L2981 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<1>, // %B`.
  **L2981 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<1>, // %B`。
- **L2982 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L2982 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L2983 EN**: Continues logic associated with callable symbol `high`.
  **L2983 CN**: 继续与可调用符号 `high` 相关的逻辑。
- **L2984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L2984 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L2985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree,`.
  **L2985 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree,`。
- **L2986 EN**: Continues the surrounding expression or declaration: `IntrNoCreateUndefOrPoison]`.
  **L2986 CN**: 继续构造周围的表达式或声明：`IntrNoCreateUndefOrPoison]`。
- **L2987 EN**: Executes a standalone statement or declaration: `>;`.
  **L2987 CN**: 执行一条独立语句或声明：`>;`。
- **L2988 EN**: Blank line separating nearby declarations or logic blocks.
  **L2988 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2989 EN**: Declares class `AMDGPUWmmaIntrinsicIU<LLVMType`.
  **L2989 CN**: 声明 class `AMDGPUWmmaIntrinsicIU<LLVMType`。
- **L2990 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L2990 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L2991 EN**: Continues the surrounding expression or declaration: `[CD], // %D`.
  **L2991 CN**: 继续构造周围的表达式或声明：`[CD], // %D`。
- **L2992 EN**: Continues the surrounding expression or declaration: `[`.
  **L2992 CN**: 继续构造周围的表达式或声明：`[`。
- **L2993 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %A_sign`.
  **L2993 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %A_sign`。
- **L2994 EN**: Continues the surrounding expression or declaration: `AB,               // %A`.
  **L2994 CN**: 继续构造周围的表达式或声明：`AB,               // %A`。
- **L2995 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %B_sign`.
  **L2995 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %B_sign`。
- **L2996 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<1>, // %B`.
  **L2996 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<1>, // %B`。
- **L2997 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L2997 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L2998 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %clamp`.
  **L2998 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %clamp`。
- **L2999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L2999 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree,`.
  **L3000 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>, IntrWillReturn, IntrNoCallback, IntrNoFree,`。
- **L3001 EN**: Continues the surrounding expression or declaration: `IntrNoCreateUndefOrPoison]`.
  **L3001 CN**: 继续构造周围的表达式或声明：`IntrNoCreateUndefOrPoison]`。
- **L3002 EN**: Executes a standalone statement or declaration: `>;`.
  **L3002 CN**: 执行一条独立语句或声明：`>;`。
- **L3003 EN**: Blank line separating nearby declarations or logic blocks.
  **L3003 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3004 EN**: Comment explains nearby logic, invariants, or intent: `WMMA GFX11Only`.
  **L3004 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA GFX11Only`。
- **L3005 EN**: Blank line separating nearby declarations or logic blocks.
  **L3005 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3006 EN**: Comment explains nearby logic, invariants, or intent: `The OPSEL intrinsics read from and write to one half of the registers, selected by the op_sel bit.`.
  **L3006 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The OPSEL intrinsics read from and write to one half of the registers, selected by the op_sel bit.`。
- **L3007 EN**: Comment explains nearby logic, invariants, or intent: `The tied versions of the f16/bf16 wmma intrinsics tie the destination matrix registers to the input accumulator registers.`.
  **L3007 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The tied versions of the f16/bf16 wmma intrinsics tie the destination matrix registers to the input accumulator registers.`。
- **L3008 EN**: Comment explains nearby logic, invariants, or intent: `The content of the other 16-bit half is preserved from the input.`.
  **L3008 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The content of the other 16-bit half is preserved from the input.`。

### Lines 3009-3040

````tablegen

defset list<Intrinsic> AMDGPUWMMAIntrinsicsGFX11 = {
def int_amdgcn_wmma_f16_16x16x16_f16_tied   : AMDGPUWmmaIntrinsicOPSEL<llvm_anyfloat_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_bf16_16x16x16_bf16_tied : AMDGPUWmmaIntrinsicOPSEL<llvm_anyint_ty, llvm_anyint_ty>;

// WMMA GFX11Plus

def int_amdgcn_wmma_f32_16x16x16_f16   : AMDGPUWmmaIntrinsic<llvm_anyfloat_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x16_bf16  : AMDGPUWmmaIntrinsic<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_i32_16x16x16_iu8   : AMDGPUWmmaIntrinsicIU<llvm_anyint_ty, llvm_anyint_ty>;
def int_amdgcn_wmma_i32_16x16x16_iu4   : AMDGPUWmmaIntrinsicIU<llvm_anyint_ty, llvm_anyint_ty>;

// GFX11: The OPSEL intrinsics read from and write to one half of the registers, selected by the op_sel bit.
//        The content of the other 16-bit half is undefined.
// GFX12: The op_sel bit must be 0.
def int_amdgcn_wmma_f16_16x16x16_f16   : AMDGPUWmmaIntrinsicOPSEL<llvm_anyfloat_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_bf16_16x16x16_bf16 : AMDGPUWmmaIntrinsicOPSEL<llvm_anyint_ty, llvm_anyint_ty>;
}

//===----------------------------------------------------------------------===//
// GFX12 Intrinsics
//===----------------------------------------------------------------------===//

def int_amdgcn_ds_bvh_stack_push4_pop1_rtn : IntDSBVHStackRtn<vdst  = llvm_i32_ty,
                                                              data1 = llvm_v4i32_ty>;

def int_amdgcn_ds_bvh_stack_push8_pop1_rtn : IntDSBVHStackRtn<vdst  = llvm_i32_ty,
                                                              data1 = llvm_v8i32_ty>;

def int_amdgcn_ds_bvh_stack_push8_pop2_rtn : IntDSBVHStackRtn<vdst  = llvm_i64_ty,
                                                              data1 = llvm_v8i32_ty>;

````
- **L3009 EN**: Blank line separating nearby declarations or logic blocks.
  **L3009 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3010 EN**: Continues the surrounding expression or declaration: `defset list<Intrinsic> AMDGPUWMMAIntrinsicsGFX11 = {`.
  **L3010 CN**: 继续构造周围的表达式或声明：`defset list<Intrinsic> AMDGPUWMMAIntrinsicsGFX11 = {`。
- **L3011 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x16_f16_tied`.
  **L3011 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x16_f16_tied`。
- **L3012 EN**: Declares TableGen def `int_amdgcn_wmma_bf16_16x16x16_bf16_tied`.
  **L3012 CN**: 声明 TableGen def `int_amdgcn_wmma_bf16_16x16x16_bf16_tied`。
- **L3013 EN**: Blank line separating nearby declarations or logic blocks.
  **L3013 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3014 EN**: Comment explains nearby logic, invariants, or intent: `WMMA GFX11Plus`.
  **L3014 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA GFX11Plus`。
- **L3015 EN**: Blank line separating nearby declarations or logic blocks.
  **L3015 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3016 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x16_f16`.
  **L3016 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x16_f16`。
- **L3017 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x16_bf16`.
  **L3017 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x16_bf16`。
- **L3018 EN**: Declares TableGen def `int_amdgcn_wmma_i32_16x16x16_iu8`.
  **L3018 CN**: 声明 TableGen def `int_amdgcn_wmma_i32_16x16x16_iu8`。
- **L3019 EN**: Declares TableGen def `int_amdgcn_wmma_i32_16x16x16_iu4`.
  **L3019 CN**: 声明 TableGen def `int_amdgcn_wmma_i32_16x16x16_iu4`。
- **L3020 EN**: Blank line separating nearby declarations or logic blocks.
  **L3020 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3021 EN**: Comment explains nearby logic, invariants, or intent: `GFX11: The OPSEL intrinsics read from and write to one half of the registers, selected by the op_sel bit.`.
  **L3021 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GFX11: The OPSEL intrinsics read from and write to one half of the registers, selected by the op_sel bit.`。
- **L3022 EN**: Comment explains nearby logic, invariants, or intent: `The content of the other 16-bit half is undefined.`.
  **L3022 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The content of the other 16-bit half is undefined.`。
- **L3023 EN**: Comment explains nearby logic, invariants, or intent: `GFX12: The op_sel bit must be 0.`.
  **L3023 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GFX12: The op_sel bit must be 0.`。
- **L3024 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x16_f16`.
  **L3024 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x16_f16`。
- **L3025 EN**: Declares TableGen def `int_amdgcn_wmma_bf16_16x16x16_bf16`.
  **L3025 CN**: 声明 TableGen def `int_amdgcn_wmma_bf16_16x16x16_bf16`。
- **L3026 EN**: Closes the current lexical scope or compound statement.
  **L3026 CN**: 结束当前词法作用域或复合语句块。
- **L3027 EN**: Blank line separating nearby declarations or logic blocks.
  **L3027 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3028 EN**: Banner comment marking a file or section boundary.
  **L3028 CN**: 横幅注释，用于标记文件或章节边界。
- **L3029 EN**: Comment explains nearby logic, invariants, or intent: `GFX12 Intrinsics`.
  **L3029 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GFX12 Intrinsics`。
- **L3030 EN**: Banner comment marking a file or section boundary.
  **L3030 CN**: 横幅注释，用于标记文件或章节边界。
- **L3031 EN**: Blank line separating nearby declarations or logic blocks.
  **L3031 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3032 EN**: Declares TableGen def `int_amdgcn_ds_bvh_stack_push4_pop1_rtn`.
  **L3032 CN**: 声明 TableGen def `int_amdgcn_ds_bvh_stack_push4_pop1_rtn`。
- **L3033 EN**: Executes a standalone statement or declaration: `data1 = llvm_v4i32_ty>;`.
  **L3033 CN**: 执行一条独立语句或声明：`data1 = llvm_v4i32_ty>;`。
- **L3034 EN**: Blank line separating nearby declarations or logic blocks.
  **L3034 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3035 EN**: Declares TableGen def `int_amdgcn_ds_bvh_stack_push8_pop1_rtn`.
  **L3035 CN**: 声明 TableGen def `int_amdgcn_ds_bvh_stack_push8_pop1_rtn`。
- **L3036 EN**: Executes a standalone statement or declaration: `data1 = llvm_v8i32_ty>;`.
  **L3036 CN**: 执行一条独立语句或声明：`data1 = llvm_v8i32_ty>;`。
- **L3037 EN**: Blank line separating nearby declarations or logic blocks.
  **L3037 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3038 EN**: Declares TableGen def `int_amdgcn_ds_bvh_stack_push8_pop2_rtn`.
  **L3038 CN**: 声明 TableGen def `int_amdgcn_ds_bvh_stack_push8_pop2_rtn`。
- **L3039 EN**: Executes a standalone statement or declaration: `data1 = llvm_v8i32_ty>;`.
  **L3039 CN**: 执行一条独立语句或声明：`data1 = llvm_v8i32_ty>;`。
- **L3040 EN**: Blank line separating nearby declarations or logic blocks.
  **L3040 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3041-3072

````tablegen
// <vdata>, <ray_origin>, <ray_dir>
//   llvm.amdgcn.image.bvh.dual.intersect.ray <node_ptr>, <ray_extent>,
//                                            <instance_mask>, <ray_origin>,
//                                            <ray_dir>, <offsets>,
//                                            <texture_descr>
def int_amdgcn_image_bvh_dual_intersect_ray :
  Intrinsic<[llvm_v10i32_ty, llvm_v3f32_ty, llvm_v3f32_ty],
            [llvm_i64_ty, llvm_float_ty, llvm_i8_ty, llvm_v3f32_ty,
             llvm_v3f32_ty, llvm_v2i32_ty, llvm_v4i32_ty],
            [IntrReadMem, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// <vdata>, <ray_origin>, <ray_dir>
//   llvm.amdgcn.image.bvh8.intersect.ray <node_ptr>, <ray_extent>,
//                                        <instance_mask>, <ray_origin>,
//                                        <ray_dir>, <offset>,
//                                        <texture_descr>
def int_amdgcn_image_bvh8_intersect_ray :
  Intrinsic<[llvm_v10i32_ty, llvm_v3f32_ty, llvm_v3f32_ty],
            [llvm_i64_ty, llvm_float_ty, llvm_i8_ty, llvm_v3f32_ty,
             llvm_v3f32_ty, llvm_i32_ty, llvm_v4i32_ty],
            [IntrReadMem, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.permlane16.var <old> <src0> <src1> <fi> <bound_control>
def int_amdgcn_permlane16_var : ClangBuiltin<"__builtin_amdgcn_permlane16_var">,
  Intrinsic<[llvm_i32_ty],
            [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn,
             ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree,]>;

// llvm.amdgcn.permlanex16.var <old> <src0> <src1> <fi> <bound_control>
def int_amdgcn_permlanex16_var : ClangBuiltin<"__builtin_amdgcn_permlanex16_var">,
  Intrinsic<[llvm_i32_ty],
````
- **L3041 EN**: Comment explains nearby logic, invariants, or intent: `<vdata>, <ray_origin>, <ray_dir>`.
  **L3041 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<vdata>, <ray_origin>, <ray_dir>`。
- **L3042 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.image.bvh.dual.intersect.ray <node_ptr>, <ray_extent>,`.
  **L3042 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.image.bvh.dual.intersect.ray <node_ptr>, <ray_extent>,`。
- **L3043 EN**: Comment explains nearby logic, invariants, or intent: `<instance_mask>, <ray_origin>,`.
  **L3043 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<instance_mask>, <ray_origin>,`。
- **L3044 EN**: Comment explains nearby logic, invariants, or intent: `<ray_dir>, <offsets>,`.
  **L3044 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<ray_dir>, <offsets>,`。
- **L3045 EN**: Comment explains nearby logic, invariants, or intent: `<texture_descr>`.
  **L3045 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<texture_descr>`。
- **L3046 EN**: Declares TableGen def `int_amdgcn_image_bvh_dual_intersect_ray`.
  **L3046 CN**: 声明 TableGen def `int_amdgcn_image_bvh_dual_intersect_ray`。
- **L3047 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v10i32_ty, llvm_v3f32_ty, llvm_v3f32_ty],`.
  **L3047 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v10i32_ty, llvm_v3f32_ty, llvm_v3f32_ty],`。
- **L3048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i64_ty, llvm_float_ty, llvm_i8_ty, llvm_v3f32_ty,`.
  **L3048 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i64_ty, llvm_float_ty, llvm_i8_ty, llvm_v3f32_ty,`。
- **L3049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v3f32_ty, llvm_v2i32_ty, llvm_v4i32_ty],`.
  **L3049 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v3f32_ty, llvm_v2i32_ty, llvm_v4i32_ty],`。
- **L3050 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3050 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3051 EN**: Blank line separating nearby declarations or logic blocks.
  **L3051 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3052 EN**: Comment explains nearby logic, invariants, or intent: `<vdata>, <ray_origin>, <ray_dir>`.
  **L3052 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<vdata>, <ray_origin>, <ray_dir>`。
- **L3053 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.image.bvh8.intersect.ray <node_ptr>, <ray_extent>,`.
  **L3053 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.image.bvh8.intersect.ray <node_ptr>, <ray_extent>,`。
- **L3054 EN**: Comment explains nearby logic, invariants, or intent: `<instance_mask>, <ray_origin>,`.
  **L3054 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<instance_mask>, <ray_origin>,`。
- **L3055 EN**: Comment explains nearby logic, invariants, or intent: `<ray_dir>, <offset>,`.
  **L3055 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<ray_dir>, <offset>,`。
- **L3056 EN**: Comment explains nearby logic, invariants, or intent: `<texture_descr>`.
  **L3056 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<texture_descr>`。
- **L3057 EN**: Declares TableGen def `int_amdgcn_image_bvh8_intersect_ray`.
  **L3057 CN**: 声明 TableGen def `int_amdgcn_image_bvh8_intersect_ray`。
- **L3058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v10i32_ty, llvm_v3f32_ty, llvm_v3f32_ty],`.
  **L3058 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v10i32_ty, llvm_v3f32_ty, llvm_v3f32_ty],`。
- **L3059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i64_ty, llvm_float_ty, llvm_i8_ty, llvm_v3f32_ty,`.
  **L3059 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i64_ty, llvm_float_ty, llvm_i8_ty, llvm_v3f32_ty,`。
- **L3060 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v3f32_ty, llvm_i32_ty, llvm_v4i32_ty],`.
  **L3060 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v3f32_ty, llvm_i32_ty, llvm_v4i32_ty],`。
- **L3061 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3061 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3062 EN**: Blank line separating nearby declarations or logic blocks.
  **L3062 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3063 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlane16.var <old> <src0> <src1> <fi> <bound_control>`.
  **L3063 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlane16.var <old> <src0> <src1> <fi> <bound_control>`。
- **L3064 EN**: Declares TableGen def `int_amdgcn_permlane16_var`.
  **L3064 CN**: 声明 TableGen def `int_amdgcn_permlane16_var`。
- **L3065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty],`.
  **L3065 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty],`。
- **L3066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],`.
  **L3066 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],`。
- **L3067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,`.
  **L3067 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,`。
- **L3068 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree,]>;`.
  **L3068 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree,]>;`。
- **L3069 EN**: Blank line separating nearby declarations or logic blocks.
  **L3069 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3070 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlanex16.var <old> <src0> <src1> <fi> <bound_control>`.
  **L3070 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlanex16.var <old> <src0> <src1> <fi> <bound_control>`。
- **L3071 EN**: Declares TableGen def `int_amdgcn_permlanex16_var`.
  **L3071 CN**: 声明 TableGen def `int_amdgcn_permlanex16_var`。
- **L3072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty],`.
  **L3072 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty],`。

### Lines 3073-3104

````tablegen
            [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn,
             ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree]>;

// SWMMAC (Wave Matrix(sparse) Multiply-Accumulate) intrinsics
//
// These operations perform a sparse matrix multiplication and accumulation of
// the form: D = A * B + C.
// A is sparse matrix, half the size of B, and is expanded using sparsity index.

class AMDGPUSWmmacIntrinsicIdxReuse<LLVMType A, LLVMType B, LLVMType CD, LLVMType Index> :
  Intrinsic<
    [CD],               // %D
    [
      A,                // %A
      B,                // %B
      LLVMMatchType<0>, // %C
      Index,            // %Sparsity index for A
      llvm_i1_ty,       // matrix_a_reuse
      llvm_i1_ty,       // matrix_b_reuse
    ],
    [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison,
     ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]
>;

class AMDGPUSWmmacIntrinsicIdx<LLVMType A, LLVMType B, LLVMType CD, LLVMType Index> :
  Intrinsic<
    [CD],               // %D
    [
      A,                // %A
      B,                // %B
      LLVMMatchType<0>, // %C
````
- **L3073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],`.
  **L3073 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i1_ty],`。
- **L3074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,`.
  **L3074 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,`。
- **L3075 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree]>;`.
  **L3075 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, IntrNoCallback, IntrNoFree]>;`。
- **L3076 EN**: Blank line separating nearby declarations or logic blocks.
  **L3076 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3077 EN**: Comment explains nearby logic, invariants, or intent: `SWMMAC (Wave Matrix(sparse) Multiply-Accumulate) intrinsics`.
  **L3077 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SWMMAC (Wave Matrix(sparse) Multiply-Accumulate) intrinsics`。
- **L3078 EN**: Separator comment used for visual grouping.
  **L3078 CN**: 用于视觉分组的分隔注释。
- **L3079 EN**: Comment explains nearby logic, invariants, or intent: `These operations perform a sparse matrix multiplication and accumulation of`.
  **L3079 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These operations perform a sparse matrix multiplication and accumulation of`。
- **L3080 EN**: Comment explains nearby logic, invariants, or intent: `the form: D = A * B + C.`.
  **L3080 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the form: D = A * B + C.`。
- **L3081 EN**: Comment explains nearby logic, invariants, or intent: `A is sparse matrix, half the size of B, and is expanded using sparsity index.`.
  **L3081 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A is sparse matrix, half the size of B, and is expanded using sparsity index.`。
- **L3082 EN**: Blank line separating nearby declarations or logic blocks.
  **L3082 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3083 EN**: Declares class `AMDGPUSWmmacIntrinsicIdxReuse<LLVMType`.
  **L3083 CN**: 声明 class `AMDGPUSWmmacIntrinsicIdxReuse<LLVMType`。
- **L3084 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L3084 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L3085 EN**: Continues the surrounding expression or declaration: `[CD],               // %D`.
  **L3085 CN**: 继续构造周围的表达式或声明：`[CD],               // %D`。
- **L3086 EN**: Continues the surrounding expression or declaration: `[`.
  **L3086 CN**: 继续构造周围的表达式或声明：`[`。
- **L3087 EN**: Continues the surrounding expression or declaration: `A,                // %A`.
  **L3087 CN**: 继续构造周围的表达式或声明：`A,                // %A`。
- **L3088 EN**: Continues the surrounding expression or declaration: `B,                // %B`.
  **L3088 CN**: 继续构造周围的表达式或声明：`B,                // %B`。
- **L3089 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L3089 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L3090 EN**: Continues the surrounding expression or declaration: `Index,            // %Sparsity index for A`.
  **L3090 CN**: 继续构造周围的表达式或声明：`Index,            // %Sparsity index for A`。
- **L3091 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_a_reuse`.
  **L3091 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_a_reuse`。
- **L3092 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_b_reuse`.
  **L3092 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_b_reuse`。
- **L3093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3093 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison,`.
  **L3094 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison,`。
- **L3095 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]`.
  **L3095 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]`。
- **L3096 EN**: Executes a standalone statement or declaration: `>;`.
  **L3096 CN**: 执行一条独立语句或声明：`>;`。
- **L3097 EN**: Blank line separating nearby declarations or logic blocks.
  **L3097 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3098 EN**: Declares class `AMDGPUSWmmacIntrinsicIdx<LLVMType`.
  **L3098 CN**: 声明 class `AMDGPUSWmmacIntrinsicIdx<LLVMType`。
- **L3099 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L3099 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L3100 EN**: Continues the surrounding expression or declaration: `[CD],               // %D`.
  **L3100 CN**: 继续构造周围的表达式或声明：`[CD],               // %D`。
- **L3101 EN**: Continues the surrounding expression or declaration: `[`.
  **L3101 CN**: 继续构造周围的表达式或声明：`[`。
- **L3102 EN**: Continues the surrounding expression or declaration: `A,                // %A`.
  **L3102 CN**: 继续构造周围的表达式或声明：`A,                // %A`。
- **L3103 EN**: Continues the surrounding expression or declaration: `B,                // %B`.
  **L3103 CN**: 继续构造周围的表达式或声明：`B,                // %B`。
- **L3104 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L3104 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。

### Lines 3105-3136

````tablegen
      Index             // %Sparsity index for A
    ],
    [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison]
>;

class AMDGPUSWmmacIntrinsicIUIdx<LLVMType A, LLVMType B, LLVMType CD, LLVMType Index> :
  Intrinsic<
    [CD],               // %D
    [
      llvm_i1_ty,       // %A_sign
      A,                // %A
      llvm_i1_ty,       // %B_sign
      B,                // %B
      LLVMMatchType<0>, // %C
      Index,            // %Sparsity index for A
      llvm_i1_ty,       // %clamp
    ],
    [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>]
>;

defset list<Intrinsic> AMDGPUWMMAIntrinsicsGFX12 = {
// WMMA (Wave Matrix Multiply-Accumulate) intrinsics
//
// These operations perform a matrix multiplication and accumulation of
// the form: D = A * B + C .

// A and B are <8 x fp8> or <8 x bf8>, but since fp8 and bf8 are not supported by llvm we use <2 x i32>.
def int_amdgcn_wmma_f32_16x16x16_fp8_fp8 : AMDGPUWmmaIntrinsic<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x16_fp8_bf8 : AMDGPUWmmaIntrinsic<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x16_bf8_fp8 : AMDGPUWmmaIntrinsic<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x16_bf8_bf8 : AMDGPUWmmaIntrinsic<llvm_anyint_ty, llvm_anyfloat_ty>;
// A and B are <16 x iu4>.
````
- **L3105 EN**: Continues the surrounding expression or declaration: `Index             // %Sparsity index for A`.
  **L3105 CN**: 继续构造周围的表达式或声明：`Index             // %Sparsity index for A`。
- **L3106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3106 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3107 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison]`.
  **L3107 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison]`。
- **L3108 EN**: Executes a standalone statement or declaration: `>;`.
  **L3108 CN**: 执行一条独立语句或声明：`>;`。
- **L3109 EN**: Blank line separating nearby declarations or logic blocks.
  **L3109 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3110 EN**: Declares class `AMDGPUSWmmacIntrinsicIUIdx<LLVMType`.
  **L3110 CN**: 声明 class `AMDGPUSWmmacIntrinsicIUIdx<LLVMType`。
- **L3111 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L3111 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L3112 EN**: Continues the surrounding expression or declaration: `[CD],               // %D`.
  **L3112 CN**: 继续构造周围的表达式或声明：`[CD],               // %D`。
- **L3113 EN**: Continues the surrounding expression or declaration: `[`.
  **L3113 CN**: 继续构造周围的表达式或声明：`[`。
- **L3114 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %A_sign`.
  **L3114 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %A_sign`。
- **L3115 EN**: Continues the surrounding expression or declaration: `A,                // %A`.
  **L3115 CN**: 继续构造周围的表达式或声明：`A,                // %A`。
- **L3116 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %B_sign`.
  **L3116 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %B_sign`。
- **L3117 EN**: Continues the surrounding expression or declaration: `B,                // %B`.
  **L3117 CN**: 继续构造周围的表达式或声明：`B,                // %B`。
- **L3118 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L3118 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L3119 EN**: Continues the surrounding expression or declaration: `Index,            // %Sparsity index for A`.
  **L3119 CN**: 继续构造周围的表达式或声明：`Index,            // %Sparsity index for A`。
- **L3120 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %clamp`.
  **L3120 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %clamp`。
- **L3121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3121 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3122 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>]`.
  **L3122 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>]`。
- **L3123 EN**: Executes a standalone statement or declaration: `>;`.
  **L3123 CN**: 执行一条独立语句或声明：`>;`。
- **L3124 EN**: Blank line separating nearby declarations or logic blocks.
  **L3124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3125 EN**: Continues the surrounding expression or declaration: `defset list<Intrinsic> AMDGPUWMMAIntrinsicsGFX12 = {`.
  **L3125 CN**: 继续构造周围的表达式或声明：`defset list<Intrinsic> AMDGPUWMMAIntrinsicsGFX12 = {`。
- **L3126 EN**: Comment explains nearby logic, invariants, or intent: `WMMA (Wave Matrix Multiply-Accumulate) intrinsics`.
  **L3126 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA (Wave Matrix Multiply-Accumulate) intrinsics`。
- **L3127 EN**: Separator comment used for visual grouping.
  **L3127 CN**: 用于视觉分组的分隔注释。
- **L3128 EN**: Comment explains nearby logic, invariants, or intent: `These operations perform a matrix multiplication and accumulation of`.
  **L3128 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These operations perform a matrix multiplication and accumulation of`。
- **L3129 EN**: Comment explains nearby logic, invariants, or intent: `the form: D = A * B + C .`.
  **L3129 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the form: D = A * B + C .`。
- **L3130 EN**: Blank line separating nearby declarations or logic blocks.
  **L3130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3131 EN**: Comment explains nearby logic, invariants, or intent: `A and B are <8 x fp8> or <8 x bf8>, but since fp8 and bf8 are not supported by llvm we use <2 x i32>.`.
  **L3131 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A and B are <8 x fp8> or <8 x bf8>, but since fp8 and bf8 are not supported by llvm we use <2 x i32>.`。
- **L3132 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x16_fp8_fp8`.
  **L3132 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x16_fp8_fp8`。
- **L3133 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x16_fp8_bf8`.
  **L3133 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x16_fp8_bf8`。
- **L3134 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x16_bf8_fp8`.
  **L3134 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x16_bf8_fp8`。
- **L3135 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x16_bf8_bf8`.
  **L3135 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x16_bf8_bf8`。
- **L3136 EN**: Comment explains nearby logic, invariants, or intent: `A and B are <16 x iu4>.`.
  **L3136 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A and B are <16 x iu4>.`。

### Lines 3137-3168

````tablegen
def int_amdgcn_wmma_i32_16x16x32_iu4     : AMDGPUWmmaIntrinsicIU<llvm_anyint_ty, llvm_anyint_ty>;

def int_amdgcn_swmmac_f32_16x16x32_f16     : AMDGPUSWmmacIntrinsicIdx<llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x32_bf16    : AMDGPUSWmmacIntrinsicIdx<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f16_16x16x32_f16     : AMDGPUSWmmacIntrinsicIdx<llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_bf16_16x16x32_bf16   : AMDGPUSWmmacIntrinsicIdx<llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_i32_16x16x32_iu8     : AMDGPUSWmmacIntrinsicIUIdx<llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_i32_16x16x32_iu4     : AMDGPUSWmmacIntrinsicIUIdx<llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_i32_16x16x64_iu4     : AMDGPUSWmmacIntrinsicIUIdx<llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x32_fp8_fp8 : AMDGPUSWmmacIntrinsicIdx<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x32_fp8_bf8 : AMDGPUSWmmacIntrinsicIdx<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x32_bf8_fp8 : AMDGPUSWmmacIntrinsicIdx<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x32_bf8_bf8 : AMDGPUSWmmacIntrinsicIdx<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
}

def int_amdgcn_global_atomic_ordered_add_b64 : AMDGPUAtomicRtn<llvm_i64_ty, global_ptr_ty>;

def int_amdgcn_flat_atomic_fmin_num   : AMDGPUAtomicRtn<llvm_anyfloat_ty>;
def int_amdgcn_flat_atomic_fmax_num   : AMDGPUAtomicRtn<llvm_anyfloat_ty>;
def int_amdgcn_global_atomic_fmin_num : AMDGPUAtomicRtn<llvm_anyfloat_ty>;
def int_amdgcn_global_atomic_fmax_num : AMDGPUAtomicRtn<llvm_anyfloat_ty>;

class AMDGPULoadIntrinsic<LLVMType ptr_ty>:
  Intrinsic<
    [llvm_any_ty],
    [ptr_ty],
    [IntrReadMem, IntrArgMemOnly, IntrWillReturn, IntrConvergent, NoCapture<ArgIndex<0>>, IntrNoCallback, IntrNoFree],
    "",
    [SDNPMemOperand]
  >;

// Wave32
````
- **L3137 EN**: Declares TableGen def `int_amdgcn_wmma_i32_16x16x32_iu4`.
  **L3137 CN**: 声明 TableGen def `int_amdgcn_wmma_i32_16x16x32_iu4`。
- **L3138 EN**: Blank line separating nearby declarations or logic blocks.
  **L3138 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3139 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x32_f16`.
  **L3139 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x32_f16`。
- **L3140 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x32_bf16`.
  **L3140 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x32_bf16`。
- **L3141 EN**: Declares TableGen def `int_amdgcn_swmmac_f16_16x16x32_f16`.
  **L3141 CN**: 声明 TableGen def `int_amdgcn_swmmac_f16_16x16x32_f16`。
- **L3142 EN**: Declares TableGen def `int_amdgcn_swmmac_bf16_16x16x32_bf16`.
  **L3142 CN**: 声明 TableGen def `int_amdgcn_swmmac_bf16_16x16x32_bf16`。
- **L3143 EN**: Declares TableGen def `int_amdgcn_swmmac_i32_16x16x32_iu8`.
  **L3143 CN**: 声明 TableGen def `int_amdgcn_swmmac_i32_16x16x32_iu8`。
- **L3144 EN**: Declares TableGen def `int_amdgcn_swmmac_i32_16x16x32_iu4`.
  **L3144 CN**: 声明 TableGen def `int_amdgcn_swmmac_i32_16x16x32_iu4`。
- **L3145 EN**: Declares TableGen def `int_amdgcn_swmmac_i32_16x16x64_iu4`.
  **L3145 CN**: 声明 TableGen def `int_amdgcn_swmmac_i32_16x16x64_iu4`。
- **L3146 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x32_fp8_fp8`.
  **L3146 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x32_fp8_fp8`。
- **L3147 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x32_fp8_bf8`.
  **L3147 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x32_fp8_bf8`。
- **L3148 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x32_bf8_fp8`.
  **L3148 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x32_bf8_fp8`。
- **L3149 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x32_bf8_bf8`.
  **L3149 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x32_bf8_bf8`。
- **L3150 EN**: Closes the current lexical scope or compound statement.
  **L3150 CN**: 结束当前词法作用域或复合语句块。
- **L3151 EN**: Blank line separating nearby declarations or logic blocks.
  **L3151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3152 EN**: Declares TableGen def `int_amdgcn_global_atomic_ordered_add_b64`.
  **L3152 CN**: 声明 TableGen def `int_amdgcn_global_atomic_ordered_add_b64`。
- **L3153 EN**: Blank line separating nearby declarations or logic blocks.
  **L3153 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3154 EN**: Declares TableGen def `int_amdgcn_flat_atomic_fmin_num`.
  **L3154 CN**: 声明 TableGen def `int_amdgcn_flat_atomic_fmin_num`。
- **L3155 EN**: Declares TableGen def `int_amdgcn_flat_atomic_fmax_num`.
  **L3155 CN**: 声明 TableGen def `int_amdgcn_flat_atomic_fmax_num`。
- **L3156 EN**: Declares TableGen def `int_amdgcn_global_atomic_fmin_num`.
  **L3156 CN**: 声明 TableGen def `int_amdgcn_global_atomic_fmin_num`。
- **L3157 EN**: Declares TableGen def `int_amdgcn_global_atomic_fmax_num`.
  **L3157 CN**: 声明 TableGen def `int_amdgcn_global_atomic_fmax_num`。
- **L3158 EN**: Blank line separating nearby declarations or logic blocks.
  **L3158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3159 EN**: Declares class `AMDGPULoadIntrinsic<LLVMType`.
  **L3159 CN**: 声明 class `AMDGPULoadIntrinsic<LLVMType`。
- **L3160 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L3160 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L3161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty],`.
  **L3161 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty],`。
- **L3162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ptr_ty],`.
  **L3162 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ptr_ty],`。
- **L3163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem, IntrArgMemOnly, IntrWillReturn, IntrConvergent, NoCapture<ArgIndex<0>>, IntrNoCallback, IntrNoFree],`.
  **L3163 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem, IntrArgMemOnly, IntrWillReturn, IntrConvergent, NoCapture<ArgIndex<0>>, IntrNoCallback, IntrNoFree],`。
- **L3164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"",`.
  **L3164 CN**: 继续一个多行参数列表、初始化器或聚合项：`"",`。
- **L3165 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand]`.
  **L3165 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand]`。
- **L3166 EN**: Executes a standalone statement or declaration: `>;`.
  **L3166 CN**: 执行一条独立语句或声明：`>;`。
- **L3167 EN**: Blank line separating nearby declarations or logic blocks.
  **L3167 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3168 EN**: Comment explains nearby logic, invariants, or intent: `Wave32`.
  **L3168 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Wave32`。

### Lines 3169-3200

````tablegen
// <2 x i32>    @llvm.amdgcn.global.load.tr.b64.v2i32(ptr addrspace(1))  -> global_load_tr_b64
// <8 x i16>    @llvm.amdgcn.global.load.tr.b128.v8i16(ptr addrspace(1))  -> global_load_tr_b128
// Wave64
// i32          @llvm.amdgcn.global.load.tr.b64.i32(ptr addrspace(1))    -> global_load_tr_b64
// <4 x i16>    @llvm.amdgcn.global.load.tr.b128.v4i16(ptr addrspace(1))  -> global_load_tr_b128

def int_amdgcn_global_load_tr_b64  : AMDGPULoadIntrinsic<global_ptr_ty>;
def int_amdgcn_global_load_tr_b128 : AMDGPULoadIntrinsic<global_ptr_ty>;
def int_amdgcn_global_load_tr4_b64 : AMDGPULoadIntrinsic<global_ptr_ty>;
def int_amdgcn_global_load_tr6_b96 : AMDGPULoadIntrinsic<global_ptr_ty>;
def int_amdgcn_ds_load_tr8_b64     : AMDGPULoadIntrinsic<local_ptr_ty>;
def int_amdgcn_ds_load_tr16_b128   : AMDGPULoadIntrinsic<local_ptr_ty>;
def int_amdgcn_ds_load_tr4_b64     : AMDGPULoadIntrinsic<local_ptr_ty>;
def int_amdgcn_ds_load_tr6_b96     : AMDGPULoadIntrinsic<local_ptr_ty>;
def int_amdgcn_ds_read_tr4_b64     : AMDGPULoadIntrinsic<local_ptr_ty>;
def int_amdgcn_ds_read_tr6_b96     : AMDGPULoadIntrinsic<local_ptr_ty>;
def int_amdgcn_ds_read_tr8_b64     : AMDGPULoadIntrinsic<local_ptr_ty>;
def int_amdgcn_ds_read_tr16_b64    : AMDGPULoadIntrinsic<local_ptr_ty>;

// i32 @llvm.amdgcn.wave.id()
def int_amdgcn_wave_id :
  DefaultAttrsIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;

def int_amdgcn_s_prefetch_data :
  Intrinsic<[],
  [llvm_anyptr_ty, // Pointer to a constant/global memory
   llvm_i32_ty],   // Length to prefetch 0-31 (1-32 chaunks, units of 128 bytes)
    [IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>, IntrNoCallback, IntrNoFree],
    "", [SDNPMemOperand]
  >;

// llvm.amdgcn.ds.bpermute.fi.b32 <index> <src>
````
- **L3169 EN**: Comment explains nearby logic, invariants, or intent: `<2 x i32>    @llvm.amdgcn.global.load.tr.b64.v2i32(ptr addrspace(1))  -> global_load_tr_b64`.
  **L3169 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<2 x i32>    @llvm.amdgcn.global.load.tr.b64.v2i32(ptr addrspace(1))  -> global_load_tr_b64`。
- **L3170 EN**: Comment explains nearby logic, invariants, or intent: `<8 x i16>    @llvm.amdgcn.global.load.tr.b128.v8i16(ptr addrspace(1))  -> global_load_tr_b128`.
  **L3170 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<8 x i16>    @llvm.amdgcn.global.load.tr.b128.v8i16(ptr addrspace(1))  -> global_load_tr_b128`。
- **L3171 EN**: Comment explains nearby logic, invariants, or intent: `Wave64`.
  **L3171 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Wave64`。
- **L3172 EN**: Comment explains nearby logic, invariants, or intent: `i32          @llvm.amdgcn.global.load.tr.b64.i32(ptr addrspace(1))    -> global_load_tr_b64`.
  **L3172 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i32          @llvm.amdgcn.global.load.tr.b64.i32(ptr addrspace(1))    -> global_load_tr_b64`。
- **L3173 EN**: Comment explains nearby logic, invariants, or intent: `<4 x i16>    @llvm.amdgcn.global.load.tr.b128.v4i16(ptr addrspace(1))  -> global_load_tr_b128`.
  **L3173 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<4 x i16>    @llvm.amdgcn.global.load.tr.b128.v4i16(ptr addrspace(1))  -> global_load_tr_b128`。
- **L3174 EN**: Blank line separating nearby declarations or logic blocks.
  **L3174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3175 EN**: Declares TableGen def `int_amdgcn_global_load_tr_b64`.
  **L3175 CN**: 声明 TableGen def `int_amdgcn_global_load_tr_b64`。
- **L3176 EN**: Declares TableGen def `int_amdgcn_global_load_tr_b128`.
  **L3176 CN**: 声明 TableGen def `int_amdgcn_global_load_tr_b128`。
- **L3177 EN**: Declares TableGen def `int_amdgcn_global_load_tr4_b64`.
  **L3177 CN**: 声明 TableGen def `int_amdgcn_global_load_tr4_b64`。
- **L3178 EN**: Declares TableGen def `int_amdgcn_global_load_tr6_b96`.
  **L3178 CN**: 声明 TableGen def `int_amdgcn_global_load_tr6_b96`。
- **L3179 EN**: Declares TableGen def `int_amdgcn_ds_load_tr8_b64`.
  **L3179 CN**: 声明 TableGen def `int_amdgcn_ds_load_tr8_b64`。
- **L3180 EN**: Declares TableGen def `int_amdgcn_ds_load_tr16_b128`.
  **L3180 CN**: 声明 TableGen def `int_amdgcn_ds_load_tr16_b128`。
- **L3181 EN**: Declares TableGen def `int_amdgcn_ds_load_tr4_b64`.
  **L3181 CN**: 声明 TableGen def `int_amdgcn_ds_load_tr4_b64`。
- **L3182 EN**: Declares TableGen def `int_amdgcn_ds_load_tr6_b96`.
  **L3182 CN**: 声明 TableGen def `int_amdgcn_ds_load_tr6_b96`。
- **L3183 EN**: Declares TableGen def `int_amdgcn_ds_read_tr4_b64`.
  **L3183 CN**: 声明 TableGen def `int_amdgcn_ds_read_tr4_b64`。
- **L3184 EN**: Declares TableGen def `int_amdgcn_ds_read_tr6_b96`.
  **L3184 CN**: 声明 TableGen def `int_amdgcn_ds_read_tr6_b96`。
- **L3185 EN**: Declares TableGen def `int_amdgcn_ds_read_tr8_b64`.
  **L3185 CN**: 声明 TableGen def `int_amdgcn_ds_read_tr8_b64`。
- **L3186 EN**: Declares TableGen def `int_amdgcn_ds_read_tr16_b64`.
  **L3186 CN**: 声明 TableGen def `int_amdgcn_ds_read_tr16_b64`。
- **L3187 EN**: Blank line separating nearby declarations or logic blocks.
  **L3187 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3188 EN**: Comment explains nearby logic, invariants, or intent: `i32 @llvm.amdgcn.wave.id()`.
  **L3188 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i32 @llvm.amdgcn.wave.id()`。
- **L3189 EN**: Declares TableGen def `int_amdgcn_wave_id`.
  **L3189 CN**: 声明 TableGen def `int_amdgcn_wave_id`。
- **L3190 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`.
  **L3190 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>, IntrNoMem, IntrSpeculatable]>;`。
- **L3191 EN**: Blank line separating nearby declarations or logic blocks.
  **L3191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3192 EN**: Declares TableGen def `int_amdgcn_s_prefetch_data`.
  **L3192 CN**: 声明 TableGen def `int_amdgcn_s_prefetch_data`。
- **L3193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L3193 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L3194 EN**: Continues the surrounding expression or declaration: `[llvm_anyptr_ty, // Pointer to a constant/global memory`.
  **L3194 CN**: 继续构造周围的表达式或声明：`[llvm_anyptr_ty, // Pointer to a constant/global memory`。
- **L3195 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],   // Length to prefetch 0-31 (1-32 chaunks, units of 128 bytes)`.
  **L3195 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],   // Length to prefetch 0-31 (1-32 chaunks, units of 128 bytes)`。
- **L3196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>, IntrNoCallback, IntrNoFree],`.
  **L3196 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>, IntrNoCallback, IntrNoFree],`。
- **L3197 EN**: Continues the surrounding expression or declaration: `"", [SDNPMemOperand]`.
  **L3197 CN**: 继续构造周围的表达式或声明：`"", [SDNPMemOperand]`。
- **L3198 EN**: Executes a standalone statement or declaration: `>;`.
  **L3198 CN**: 执行一条独立语句或声明：`>;`。
- **L3199 EN**: Blank line separating nearby declarations or logic blocks.
  **L3199 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3200 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ds.bpermute.fi.b32 <index> <src>`.
  **L3200 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ds.bpermute.fi.b32 <index> <src>`。

### Lines 3201-3232

````tablegen
def int_amdgcn_ds_bpermute_fi_b32 :
  ClangBuiltin<"__builtin_amdgcn_ds_bpermute_fi_b32">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
    [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

def int_amdgcn_flat_prefetch : ClangBuiltin<"__builtin_amdgcn_flat_prefetch">,
  Intrinsic<[],
  [llvm_ptr_ty,    // Pointer
   llvm_i32_ty],   // cachepolicy(imm), bits [0-2] = th, bits [3-4] = scope
    [IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>,
     IntrNoCallback, IntrNoFree, ImmArg<ArgIndex<1>>],
    "", [SDNPMemOperand]
  >;

def int_amdgcn_global_prefetch : ClangBuiltin<"__builtin_amdgcn_global_prefetch">,
  Intrinsic<[],
  [LLVMQualPointerType<1>,    // Pointer
   llvm_i32_ty],              // cachepolicy(imm), bits [0-2] = th, bits [3-4] = scope
    [IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>,
     IntrNoCallback, IntrNoFree, ImmArg<ArgIndex<1>>],
    "", [SDNPMemOperand]
  >;

//===----------------------------------------------------------------------===//
// Deep learning intrinsics.
//===----------------------------------------------------------------------===//

// f32 %r = llvm.amdgcn.fdot2(v2f16 %a, v2f16 %b, f32 %c, i1 %clamp)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %c
def int_amdgcn_fdot2 :
  ClangBuiltin<"__builtin_amdgcn_fdot2">,
  PureIntrinsic<
````
- **L3201 EN**: Declares TableGen def `int_amdgcn_ds_bpermute_fi_b32`.
  **L3201 CN**: 声明 TableGen def `int_amdgcn_ds_bpermute_fi_b32`。
- **L3202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_bpermute_fi_b32">,`.
  **L3202 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_bpermute_fi_b32">,`。
- **L3203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L3203 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L3204 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3204 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3205 EN**: Blank line separating nearby declarations or logic blocks.
  **L3205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3206 EN**: Declares TableGen def `int_amdgcn_flat_prefetch`.
  **L3206 CN**: 声明 TableGen def `int_amdgcn_flat_prefetch`。
- **L3207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L3207 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L3208 EN**: Continues the surrounding expression or declaration: `[llvm_ptr_ty,    // Pointer`.
  **L3208 CN**: 继续构造周围的表达式或声明：`[llvm_ptr_ty,    // Pointer`。
- **L3209 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L3209 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L3210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>,`.
  **L3210 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>,`。
- **L3211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrNoCallback, IntrNoFree, ImmArg<ArgIndex<1>>],`.
  **L3211 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrNoCallback, IntrNoFree, ImmArg<ArgIndex<1>>],`。
- **L3212 EN**: Continues the surrounding expression or declaration: `"", [SDNPMemOperand]`.
  **L3212 CN**: 继续构造周围的表达式或声明：`"", [SDNPMemOperand]`。
- **L3213 EN**: Executes a standalone statement or declaration: `>;`.
  **L3213 CN**: 执行一条独立语句或声明：`>;`。
- **L3214 EN**: Blank line separating nearby declarations or logic blocks.
  **L3214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3215 EN**: Declares TableGen def `int_amdgcn_global_prefetch`.
  **L3215 CN**: 声明 TableGen def `int_amdgcn_global_prefetch`。
- **L3216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L3216 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L3217 EN**: Continues the surrounding expression or declaration: `[LLVMQualPointerType<1>,    // Pointer`.
  **L3217 CN**: 继续构造周围的表达式或声明：`[LLVMQualPointerType<1>,    // Pointer`。
- **L3218 EN**: Continues logic associated with callable symbol `cachepolicy`.
  **L3218 CN**: 继续与可调用符号 `cachepolicy` 相关的逻辑。
- **L3219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>,`.
  **L3219 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, NoCapture<ArgIndex<0>>,`。
- **L3220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrNoCallback, IntrNoFree, ImmArg<ArgIndex<1>>],`.
  **L3220 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrNoCallback, IntrNoFree, ImmArg<ArgIndex<1>>],`。
- **L3221 EN**: Continues the surrounding expression or declaration: `"", [SDNPMemOperand]`.
  **L3221 CN**: 继续构造周围的表达式或声明：`"", [SDNPMemOperand]`。
- **L3222 EN**: Executes a standalone statement or declaration: `>;`.
  **L3222 CN**: 执行一条独立语句或声明：`>;`。
- **L3223 EN**: Blank line separating nearby declarations or logic blocks.
  **L3223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3224 EN**: Banner comment marking a file or section boundary.
  **L3224 CN**: 横幅注释，用于标记文件或章节边界。
- **L3225 EN**: Comment explains nearby logic, invariants, or intent: `Deep learning intrinsics.`.
  **L3225 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Deep learning intrinsics.`。
- **L3226 EN**: Banner comment marking a file or section boundary.
  **L3226 CN**: 横幅注释，用于标记文件或章节边界。
- **L3227 EN**: Blank line separating nearby declarations or logic blocks.
  **L3227 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3228 EN**: Comment explains nearby logic, invariants, or intent: `f32 %r = llvm.amdgcn.fdot2(v2f16 %a, v2f16 %b, f32 %c, i1 %clamp)`.
  **L3228 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`f32 %r = llvm.amdgcn.fdot2(v2f16 %a, v2f16 %b, f32 %c, i1 %clamp)`。
- **L3229 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`.
  **L3229 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`。
- **L3230 EN**: Declares TableGen def `int_amdgcn_fdot2`.
  **L3230 CN**: 声明 TableGen def `int_amdgcn_fdot2`。
- **L3231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_fdot2">,`.
  **L3231 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_fdot2">,`。
- **L3232 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3232 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。

### Lines 3233-3264

````tablegen
    [llvm_float_ty], // %r
    [
      llvm_v2f16_ty, // %a
      llvm_v2f16_ty, // %b
      llvm_float_ty, // %c
      llvm_i1_ty     // %clamp
    ],
    [ImmArg<ArgIndex<3>>]
  >;

// f16 %r = llvm.amdgcn.fdot2.f16.f16(v2f16 %a, v2f16 %b, f16 %c)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %c
def int_amdgcn_fdot2_f16_f16 :
  ClangBuiltin<"__builtin_amdgcn_fdot2_f16_f16">,
  PureIntrinsic<
    [llvm_half_ty],  // %r
    [
      llvm_v2f16_ty, // %a
      llvm_v2f16_ty, // %b
      llvm_half_ty   // %c
    ]
  >;

// bf16 %r = llvm.amdgcn.fdot2.bf16.bf16(v2bf16 %a, v2bf16 %b, bf16 %c)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %c
def int_amdgcn_fdot2_bf16_bf16 :
  ClangBuiltin<"__builtin_amdgcn_fdot2_bf16_bf16">,
  PureIntrinsic<
    [llvm_bfloat_ty],   // %r
    [
      llvm_v2bf16_ty, // %a
      llvm_v2bf16_ty, // %b
````
- **L3233 EN**: Continues the surrounding expression or declaration: `[llvm_float_ty], // %r`.
  **L3233 CN**: 继续构造周围的表达式或声明：`[llvm_float_ty], // %r`。
- **L3234 EN**: Continues the surrounding expression or declaration: `[`.
  **L3234 CN**: 继续构造周围的表达式或声明：`[`。
- **L3235 EN**: Continues the surrounding expression or declaration: `llvm_v2f16_ty, // %a`.
  **L3235 CN**: 继续构造周围的表达式或声明：`llvm_v2f16_ty, // %a`。
- **L3236 EN**: Continues the surrounding expression or declaration: `llvm_v2f16_ty, // %b`.
  **L3236 CN**: 继续构造周围的表达式或声明：`llvm_v2f16_ty, // %b`。
- **L3237 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // %c`.
  **L3237 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // %c`。
- **L3238 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty     // %clamp`.
  **L3238 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty     // %clamp`。
- **L3239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3239 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3240 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L3240 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L3241 EN**: Executes a standalone statement or declaration: `>;`.
  **L3241 CN**: 执行一条独立语句或声明：`>;`。
- **L3242 EN**: Blank line separating nearby declarations or logic blocks.
  **L3242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3243 EN**: Comment explains nearby logic, invariants, or intent: `f16 %r = llvm.amdgcn.fdot2.f16.f16(v2f16 %a, v2f16 %b, f16 %c)`.
  **L3243 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`f16 %r = llvm.amdgcn.fdot2.f16.f16(v2f16 %a, v2f16 %b, f16 %c)`。
- **L3244 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`.
  **L3244 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`。
- **L3245 EN**: Declares TableGen def `int_amdgcn_fdot2_f16_f16`.
  **L3245 CN**: 声明 TableGen def `int_amdgcn_fdot2_f16_f16`。
- **L3246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_fdot2_f16_f16">,`.
  **L3246 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_fdot2_f16_f16">,`。
- **L3247 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3247 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3248 EN**: Continues the surrounding expression or declaration: `[llvm_half_ty],  // %r`.
  **L3248 CN**: 继续构造周围的表达式或声明：`[llvm_half_ty],  // %r`。
- **L3249 EN**: Continues the surrounding expression or declaration: `[`.
  **L3249 CN**: 继续构造周围的表达式或声明：`[`。
- **L3250 EN**: Continues the surrounding expression or declaration: `llvm_v2f16_ty, // %a`.
  **L3250 CN**: 继续构造周围的表达式或声明：`llvm_v2f16_ty, // %a`。
- **L3251 EN**: Continues the surrounding expression or declaration: `llvm_v2f16_ty, // %b`.
  **L3251 CN**: 继续构造周围的表达式或声明：`llvm_v2f16_ty, // %b`。
- **L3252 EN**: Continues the surrounding expression or declaration: `llvm_half_ty   // %c`.
  **L3252 CN**: 继续构造周围的表达式或声明：`llvm_half_ty   // %c`。
- **L3253 EN**: Continues the surrounding expression or declaration: `]`.
  **L3253 CN**: 继续构造周围的表达式或声明：`]`。
- **L3254 EN**: Executes a standalone statement or declaration: `>;`.
  **L3254 CN**: 执行一条独立语句或声明：`>;`。
- **L3255 EN**: Blank line separating nearby declarations or logic blocks.
  **L3255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3256 EN**: Comment explains nearby logic, invariants, or intent: `bf16 %r = llvm.amdgcn.fdot2.bf16.bf16(v2bf16 %a, v2bf16 %b, bf16 %c)`.
  **L3256 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bf16 %r = llvm.amdgcn.fdot2.bf16.bf16(v2bf16 %a, v2bf16 %b, bf16 %c)`。
- **L3257 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`.
  **L3257 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`。
- **L3258 EN**: Declares TableGen def `int_amdgcn_fdot2_bf16_bf16`.
  **L3258 CN**: 声明 TableGen def `int_amdgcn_fdot2_bf16_bf16`。
- **L3259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_fdot2_bf16_bf16">,`.
  **L3259 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_fdot2_bf16_bf16">,`。
- **L3260 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3260 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3261 EN**: Continues the surrounding expression or declaration: `[llvm_bfloat_ty],   // %r`.
  **L3261 CN**: 继续构造周围的表达式或声明：`[llvm_bfloat_ty],   // %r`。
- **L3262 EN**: Continues the surrounding expression or declaration: `[`.
  **L3262 CN**: 继续构造周围的表达式或声明：`[`。
- **L3263 EN**: Continues the surrounding expression or declaration: `llvm_v2bf16_ty, // %a`.
  **L3263 CN**: 继续构造周围的表达式或声明：`llvm_v2bf16_ty, // %a`。
- **L3264 EN**: Continues the surrounding expression or declaration: `llvm_v2bf16_ty, // %b`.
  **L3264 CN**: 继续构造周围的表达式或声明：`llvm_v2bf16_ty, // %b`。

### Lines 3265-3296

````tablegen
      llvm_bfloat_ty    // %c
    ]
  >;

// f32 %r = llvm.amdgcn.fdot2.f32.bf16(v2bf16 %a, v2bf16 %b, f32 %c, i1 %clamp)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %c
def int_amdgcn_fdot2_f32_bf16 :
  ClangBuiltin<"__builtin_amdgcn_fdot2_f32_bf16">,
  PureIntrinsic<
    [llvm_float_ty], // %r
    [
      llvm_v2bf16_ty, // %a
      llvm_v2bf16_ty, // %b
      llvm_float_ty, // %c
      llvm_i1_ty     // %clamp
    ],
    [ImmArg<ArgIndex<3>>]
  >;

// f32 %r = llvm.amdgcn.fdot2c.f32.bf16(v2bf16 %a, v2bf16 %b, f32 %c, i1 %clamp)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + c
// TODO: This actually is similar to llvm.amdgcn.fdot2 intrinsics which produces
// v_dot2c_f32_f16 on gfx942. Maybe we can consolidate these.

def int_amdgcn_fdot2c_f32_bf16 :
  ClangBuiltin<"__builtin_amdgcn_fdot2c_f32_bf16">,
  PureIntrinsic<
    [llvm_float_ty], // %r
    [
      llvm_v2bf16_ty, // %a
      llvm_v2bf16_ty, // %b
      llvm_float_ty, // %c
````
- **L3265 EN**: Continues the surrounding expression or declaration: `llvm_bfloat_ty    // %c`.
  **L3265 CN**: 继续构造周围的表达式或声明：`llvm_bfloat_ty    // %c`。
- **L3266 EN**: Continues the surrounding expression or declaration: `]`.
  **L3266 CN**: 继续构造周围的表达式或声明：`]`。
- **L3267 EN**: Executes a standalone statement or declaration: `>;`.
  **L3267 CN**: 执行一条独立语句或声明：`>;`。
- **L3268 EN**: Blank line separating nearby declarations or logic blocks.
  **L3268 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3269 EN**: Comment explains nearby logic, invariants, or intent: `f32 %r = llvm.amdgcn.fdot2.f32.bf16(v2bf16 %a, v2bf16 %b, f32 %c, i1 %clamp)`.
  **L3269 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`f32 %r = llvm.amdgcn.fdot2.f32.bf16(v2bf16 %a, v2bf16 %b, f32 %c, i1 %clamp)`。
- **L3270 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`.
  **L3270 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`。
- **L3271 EN**: Declares TableGen def `int_amdgcn_fdot2_f32_bf16`.
  **L3271 CN**: 声明 TableGen def `int_amdgcn_fdot2_f32_bf16`。
- **L3272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_fdot2_f32_bf16">,`.
  **L3272 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_fdot2_f32_bf16">,`。
- **L3273 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3273 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3274 EN**: Continues the surrounding expression or declaration: `[llvm_float_ty], // %r`.
  **L3274 CN**: 继续构造周围的表达式或声明：`[llvm_float_ty], // %r`。
- **L3275 EN**: Continues the surrounding expression or declaration: `[`.
  **L3275 CN**: 继续构造周围的表达式或声明：`[`。
- **L3276 EN**: Continues the surrounding expression or declaration: `llvm_v2bf16_ty, // %a`.
  **L3276 CN**: 继续构造周围的表达式或声明：`llvm_v2bf16_ty, // %a`。
- **L3277 EN**: Continues the surrounding expression or declaration: `llvm_v2bf16_ty, // %b`.
  **L3277 CN**: 继续构造周围的表达式或声明：`llvm_v2bf16_ty, // %b`。
- **L3278 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // %c`.
  **L3278 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // %c`。
- **L3279 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty     // %clamp`.
  **L3279 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty     // %clamp`。
- **L3280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3280 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3281 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L3281 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L3282 EN**: Executes a standalone statement or declaration: `>;`.
  **L3282 CN**: 执行一条独立语句或声明：`>;`。
- **L3283 EN**: Blank line separating nearby declarations or logic blocks.
  **L3283 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3284 EN**: Comment explains nearby logic, invariants, or intent: `f32 %r = llvm.amdgcn.fdot2c.f32.bf16(v2bf16 %a, v2bf16 %b, f32 %c, i1 %clamp)`.
  **L3284 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`f32 %r = llvm.amdgcn.fdot2c.f32.bf16(v2bf16 %a, v2bf16 %b, f32 %c, i1 %clamp)`。
- **L3285 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + c`.
  **L3285 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + c`。
- **L3286 EN**: Comment records a pending task or caution: `TODO: This actually is similar to llvm.amdgcn.fdot2 intrinsics which produces`.
  **L3286 CN**: 注释记录了待办事项或注意点：`TODO: This actually is similar to llvm.amdgcn.fdot2 intrinsics which produces`。
- **L3287 EN**: Comment explains nearby logic, invariants, or intent: `v_dot2c_f32_f16 on gfx942. Maybe we can consolidate these.`.
  **L3287 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v_dot2c_f32_f16 on gfx942. Maybe we can consolidate these.`。
- **L3288 EN**: Blank line separating nearby declarations or logic blocks.
  **L3288 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3289 EN**: Declares TableGen def `int_amdgcn_fdot2c_f32_bf16`.
  **L3289 CN**: 声明 TableGen def `int_amdgcn_fdot2c_f32_bf16`。
- **L3290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_fdot2c_f32_bf16">,`.
  **L3290 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_fdot2c_f32_bf16">,`。
- **L3291 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3291 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3292 EN**: Continues the surrounding expression or declaration: `[llvm_float_ty], // %r`.
  **L3292 CN**: 继续构造周围的表达式或声明：`[llvm_float_ty], // %r`。
- **L3293 EN**: Continues the surrounding expression or declaration: `[`.
  **L3293 CN**: 继续构造周围的表达式或声明：`[`。
- **L3294 EN**: Continues the surrounding expression or declaration: `llvm_v2bf16_ty, // %a`.
  **L3294 CN**: 继续构造周围的表达式或声明：`llvm_v2bf16_ty, // %a`。
- **L3295 EN**: Continues the surrounding expression or declaration: `llvm_v2bf16_ty, // %b`.
  **L3295 CN**: 继续构造周围的表达式或声明：`llvm_v2bf16_ty, // %b`。
- **L3296 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // %c`.
  **L3296 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // %c`。

### Lines 3297-3328

````tablegen
      llvm_i1_ty     // %clamp
    ],
    [ImmArg<ArgIndex<3>>]
  >;

// i32 %r = llvm.amdgcn.sdot2(v2i16 %a, v2i16 %b, i32 %c, i1 %clamp)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %c
def int_amdgcn_sdot2 :
  ClangBuiltin<"__builtin_amdgcn_sdot2">,
  PureIntrinsic<
    [llvm_i32_ty], // %r
    [
      llvm_v2i16_ty, // %a
      llvm_v2i16_ty, // %b
      llvm_i32_ty,   // %c
      llvm_i1_ty     // %clamp
    ],
    [ImmArg<ArgIndex<3>>]
  >;

// u32 %r = llvm.amdgcn.udot2(v2u16 %a, v2u16 %b, u32 %c, i1 %clamp)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %c
def int_amdgcn_udot2 :
  ClangBuiltin<"__builtin_amdgcn_udot2">,
  PureIntrinsic<
    [llvm_i32_ty], // %r
    [
      llvm_v2i16_ty, // %a
      llvm_v2i16_ty, // %b
      llvm_i32_ty,   // %c
      llvm_i1_ty     // %clamp
    ],
````
- **L3297 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty     // %clamp`.
  **L3297 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty     // %clamp`。
- **L3298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3298 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3299 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L3299 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L3300 EN**: Executes a standalone statement or declaration: `>;`.
  **L3300 CN**: 执行一条独立语句或声明：`>;`。
- **L3301 EN**: Blank line separating nearby declarations or logic blocks.
  **L3301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3302 EN**: Comment explains nearby logic, invariants, or intent: `i32 %r = llvm.amdgcn.sdot2(v2i16 %a, v2i16 %b, i32 %c, i1 %clamp)`.
  **L3302 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i32 %r = llvm.amdgcn.sdot2(v2i16 %a, v2i16 %b, i32 %c, i1 %clamp)`。
- **L3303 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`.
  **L3303 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`。
- **L3304 EN**: Declares TableGen def `int_amdgcn_sdot2`.
  **L3304 CN**: 声明 TableGen def `int_amdgcn_sdot2`。
- **L3305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_sdot2">,`.
  **L3305 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_sdot2">,`。
- **L3306 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3306 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3307 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // %r`.
  **L3307 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // %r`。
- **L3308 EN**: Continues the surrounding expression or declaration: `[`.
  **L3308 CN**: 继续构造周围的表达式或声明：`[`。
- **L3309 EN**: Continues the surrounding expression or declaration: `llvm_v2i16_ty, // %a`.
  **L3309 CN**: 继续构造周围的表达式或声明：`llvm_v2i16_ty, // %a`。
- **L3310 EN**: Continues the surrounding expression or declaration: `llvm_v2i16_ty, // %b`.
  **L3310 CN**: 继续构造周围的表达式或声明：`llvm_v2i16_ty, // %b`。
- **L3311 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // %c`.
  **L3311 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // %c`。
- **L3312 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty     // %clamp`.
  **L3312 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty     // %clamp`。
- **L3313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3313 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3314 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L3314 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L3315 EN**: Executes a standalone statement or declaration: `>;`.
  **L3315 CN**: 执行一条独立语句或声明：`>;`。
- **L3316 EN**: Blank line separating nearby declarations or logic blocks.
  **L3316 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3317 EN**: Comment explains nearby logic, invariants, or intent: `u32 %r = llvm.amdgcn.udot2(v2u16 %a, v2u16 %b, u32 %c, i1 %clamp)`.
  **L3317 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`u32 %r = llvm.amdgcn.udot2(v2u16 %a, v2u16 %b, u32 %c, i1 %clamp)`。
- **L3318 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`.
  **L3318 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %c`。
- **L3319 EN**: Declares TableGen def `int_amdgcn_udot2`.
  **L3319 CN**: 声明 TableGen def `int_amdgcn_udot2`。
- **L3320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_udot2">,`.
  **L3320 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_udot2">,`。
- **L3321 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3321 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3322 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // %r`.
  **L3322 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // %r`。
- **L3323 EN**: Continues the surrounding expression or declaration: `[`.
  **L3323 CN**: 继续构造周围的表达式或声明：`[`。
- **L3324 EN**: Continues the surrounding expression or declaration: `llvm_v2i16_ty, // %a`.
  **L3324 CN**: 继续构造周围的表达式或声明：`llvm_v2i16_ty, // %a`。
- **L3325 EN**: Continues the surrounding expression or declaration: `llvm_v2i16_ty, // %b`.
  **L3325 CN**: 继续构造周围的表达式或声明：`llvm_v2i16_ty, // %b`。
- **L3326 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // %c`.
  **L3326 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // %c`。
- **L3327 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty     // %clamp`.
  **L3327 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty     // %clamp`。
- **L3328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3328 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。

### Lines 3329-3360

````tablegen
    [ImmArg<ArgIndex<3>>]
  >;

// i32 %r = llvm.amdgcn.sdot4(v4i8 (as i32) %a, v4i8 (as i32) %b, i32 %c, i1 %clamp)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c
def int_amdgcn_sdot4 :
  ClangBuiltin<"__builtin_amdgcn_sdot4">,
  PureIntrinsic<
    [llvm_i32_ty], // %r
    [
      llvm_i32_ty, // %a
      llvm_i32_ty, // %b
      llvm_i32_ty, // %c
      llvm_i1_ty   // %clamp
    ],
    [ImmArg<ArgIndex<3>>]
  >;

// u32 %r = llvm.amdgcn.udot4(v4u8 (as u32) %a, v4u8 (as u32) %b, u32 %c, i1 %clamp)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c
def int_amdgcn_udot4 :
  ClangBuiltin<"__builtin_amdgcn_udot4">,
  PureIntrinsic<
    [llvm_i32_ty], // %r
    [
      llvm_i32_ty, // %a
      llvm_i32_ty, // %b
      llvm_i32_ty, // %c
      llvm_i1_ty   // %clamp
    ],
    [ImmArg<ArgIndex<3>>]
  >;
````
- **L3329 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L3329 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L3330 EN**: Executes a standalone statement or declaration: `>;`.
  **L3330 CN**: 执行一条独立语句或声明：`>;`。
- **L3331 EN**: Blank line separating nearby declarations or logic blocks.
  **L3331 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3332 EN**: Comment explains nearby logic, invariants, or intent: `i32 %r = llvm.amdgcn.sdot4(v4i8 (as i32) %a, v4i8 (as i32) %b, i32 %c, i1 %clamp)`.
  **L3332 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i32 %r = llvm.amdgcn.sdot4(v4i8 (as i32) %a, v4i8 (as i32) %b, i32 %c, i1 %clamp)`。
- **L3333 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c`.
  **L3333 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c`。
- **L3334 EN**: Declares TableGen def `int_amdgcn_sdot4`.
  **L3334 CN**: 声明 TableGen def `int_amdgcn_sdot4`。
- **L3335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_sdot4">,`.
  **L3335 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_sdot4">,`。
- **L3336 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3336 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3337 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // %r`.
  **L3337 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // %r`。
- **L3338 EN**: Continues the surrounding expression or declaration: `[`.
  **L3338 CN**: 继续构造周围的表达式或声明：`[`。
- **L3339 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %a`.
  **L3339 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %a`。
- **L3340 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %b`.
  **L3340 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %b`。
- **L3341 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %c`.
  **L3341 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %c`。
- **L3342 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty   // %clamp`.
  **L3342 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty   // %clamp`。
- **L3343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3343 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3344 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L3344 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L3345 EN**: Executes a standalone statement or declaration: `>;`.
  **L3345 CN**: 执行一条独立语句或声明：`>;`。
- **L3346 EN**: Blank line separating nearby declarations or logic blocks.
  **L3346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3347 EN**: Comment explains nearby logic, invariants, or intent: `u32 %r = llvm.amdgcn.udot4(v4u8 (as u32) %a, v4u8 (as u32) %b, u32 %c, i1 %clamp)`.
  **L3347 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`u32 %r = llvm.amdgcn.udot4(v4u8 (as u32) %a, v4u8 (as u32) %b, u32 %c, i1 %clamp)`。
- **L3348 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c`.
  **L3348 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c`。
- **L3349 EN**: Declares TableGen def `int_amdgcn_udot4`.
  **L3349 CN**: 声明 TableGen def `int_amdgcn_udot4`。
- **L3350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_udot4">,`.
  **L3350 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_udot4">,`。
- **L3351 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3351 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3352 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // %r`.
  **L3352 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // %r`。
- **L3353 EN**: Continues the surrounding expression or declaration: `[`.
  **L3353 CN**: 继续构造周围的表达式或声明：`[`。
- **L3354 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %a`.
  **L3354 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %a`。
- **L3355 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %b`.
  **L3355 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %b`。
- **L3356 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %c`.
  **L3356 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %c`。
- **L3357 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty   // %clamp`.
  **L3357 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty   // %clamp`。
- **L3358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3358 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3359 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L3359 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L3360 EN**: Executes a standalone statement or declaration: `>;`.
  **L3360 CN**: 执行一条独立语句或声明：`>;`。

### Lines 3361-3392

````tablegen

// i32 %r = llvm.amdgcn.sudot4(i1 %a_sign, v4i8 (as i32) %a, i1 %b_sign, v4i8 (as i32) %b, i32 %c, i1 %clamp)
// Treat input as signed (_sign = 1) or unsigned (_sign = 0).
// a[i in 0. . . 3] = (%a_sign ? a.i8[i] : promoteToSigned(a.u8[i]));
// b[i in 0. . . 3] = (%b_sign ? b.i8[i] : promoteToSigned(b.u8[i]));
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c
def int_amdgcn_sudot4 :
  ClangBuiltin<"__builtin_amdgcn_sudot4">,
  PureIntrinsic<
    [llvm_i32_ty], // %r
    [
      llvm_i1_ty,  // %a_sign
      llvm_i32_ty, // %a
      llvm_i1_ty,  // %b_sign
      llvm_i32_ty, // %b
      llvm_i32_ty, // %c
      llvm_i1_ty   // %clamp
    ],
    [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>]
  >;

// i32 %r = llvm.amdgcn.sdot8(v8i4 (as i32) %a, v8i4 (as i32) %b, i32 %c, i1 %clamp)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] +
//        %a[4] * %b[4] + %a[5] * %b[5] + %a[6] * %b[6] + %a[7] * %b[7] + %c
def int_amdgcn_sdot8 :
  ClangBuiltin<"__builtin_amdgcn_sdot8">,
  PureIntrinsic<
    [llvm_i32_ty], // %r
    [
      llvm_i32_ty, // %a
      llvm_i32_ty, // %b
      llvm_i32_ty, // %c
````
- **L3361 EN**: Blank line separating nearby declarations or logic blocks.
  **L3361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3362 EN**: Comment explains nearby logic, invariants, or intent: `i32 %r = llvm.amdgcn.sudot4(i1 %a_sign, v4i8 (as i32) %a, i1 %b_sign, v4i8 (as i32) %b, i32 %c, i1 %clamp)`.
  **L3362 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i32 %r = llvm.amdgcn.sudot4(i1 %a_sign, v4i8 (as i32) %a, i1 %b_sign, v4i8 (as i32) %b, i32 %c, i1 %clamp)`。
- **L3363 EN**: Comment explains nearby logic, invariants, or intent: `Treat input as signed (_sign = 1) or unsigned (_sign = 0).`.
  **L3363 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Treat input as signed (_sign = 1) or unsigned (_sign = 0).`。
- **L3364 EN**: Comment explains nearby logic, invariants, or intent: `a[i in 0. . . 3] = (%a_sign ? a.i8[i] : promoteToSigned(a.u8[i]));`.
  **L3364 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a[i in 0. . . 3] = (%a_sign ? a.i8[i] : promoteToSigned(a.u8[i]));`。
- **L3365 EN**: Comment explains nearby logic, invariants, or intent: `b[i in 0. . . 3] = (%b_sign ? b.i8[i] : promoteToSigned(b.u8[i]));`.
  **L3365 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`b[i in 0. . . 3] = (%b_sign ? b.i8[i] : promoteToSigned(b.u8[i]));`。
- **L3366 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c`.
  **L3366 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c`。
- **L3367 EN**: Declares TableGen def `int_amdgcn_sudot4`.
  **L3367 CN**: 声明 TableGen def `int_amdgcn_sudot4`。
- **L3368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_sudot4">,`.
  **L3368 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_sudot4">,`。
- **L3369 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3369 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3370 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // %r`.
  **L3370 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // %r`。
- **L3371 EN**: Continues the surrounding expression or declaration: `[`.
  **L3371 CN**: 继续构造周围的表达式或声明：`[`。
- **L3372 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,  // %a_sign`.
  **L3372 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,  // %a_sign`。
- **L3373 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %a`.
  **L3373 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %a`。
- **L3374 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,  // %b_sign`.
  **L3374 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,  // %b_sign`。
- **L3375 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %b`.
  **L3375 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %b`。
- **L3376 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %c`.
  **L3376 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %c`。
- **L3377 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty   // %clamp`.
  **L3377 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty   // %clamp`。
- **L3378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3378 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3379 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>]`.
  **L3379 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>]`。
- **L3380 EN**: Executes a standalone statement or declaration: `>;`.
  **L3380 CN**: 执行一条独立语句或声明：`>;`。
- **L3381 EN**: Blank line separating nearby declarations or logic blocks.
  **L3381 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3382 EN**: Comment explains nearby logic, invariants, or intent: `i32 %r = llvm.amdgcn.sdot8(v8i4 (as i32) %a, v8i4 (as i32) %b, i32 %c, i1 %clamp)`.
  **L3382 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i32 %r = llvm.amdgcn.sdot8(v8i4 (as i32) %a, v8i4 (as i32) %b, i32 %c, i1 %clamp)`。
- **L3383 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] +`.
  **L3383 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] +`。
- **L3384 EN**: Comment explains nearby logic, invariants, or intent: `%a[4] * %b[4] + %a[5] * %b[5] + %a[6] * %b[6] + %a[7] * %b[7] + %c`.
  **L3384 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%a[4] * %b[4] + %a[5] * %b[5] + %a[6] * %b[6] + %a[7] * %b[7] + %c`。
- **L3385 EN**: Declares TableGen def `int_amdgcn_sdot8`.
  **L3385 CN**: 声明 TableGen def `int_amdgcn_sdot8`。
- **L3386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_sdot8">,`.
  **L3386 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_sdot8">,`。
- **L3387 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3387 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3388 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // %r`.
  **L3388 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // %r`。
- **L3389 EN**: Continues the surrounding expression or declaration: `[`.
  **L3389 CN**: 继续构造周围的表达式或声明：`[`。
- **L3390 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %a`.
  **L3390 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %a`。
- **L3391 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %b`.
  **L3391 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %b`。
- **L3392 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %c`.
  **L3392 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %c`。

### Lines 3393-3424

````tablegen
      llvm_i1_ty   // %clamp
    ],
    [ImmArg<ArgIndex<3>>]
  >;

// u32 %r = llvm.amdgcn.udot8(v8u4 (as u32) %a, v8u4 (as u32) %b, u32 %c, i1 %clamp)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] +
//        %a[4] * %b[4] + %a[5] * %b[5] + %a[6] * %b[6] + %a[7] * %b[7] + %c
def int_amdgcn_udot8 :
  ClangBuiltin<"__builtin_amdgcn_udot8">,
  PureIntrinsic<
    [llvm_i32_ty], // %r
    [
      llvm_i32_ty, // %a
      llvm_i32_ty, // %b
      llvm_i32_ty, // %c
      llvm_i1_ty   // %clamp
    ],
    [ImmArg<ArgIndex<3>>]
  >;

// i32 %r = llvm.amdgcn.sudot8(i1 %a_sign, v8i4 (as i32) %a, i1 %b_sign, v8i4 (as i32) %b, i32 %c, i1 %clamp)
// Treat input as signed (_sign = 1) or unsigned (_sign = 0).
// a[i in 0. . . 7] = (%a_sign ? a.i4[i] : promoteToSigned(a.u4[i]));
// b[i in 0. . . 7] = (%b_sign ? b.i4[i] : promoteToSigned(b.u4[i]));
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] +
//        %a[4] * %b[4] + %a[5] * %b[5] + %a[6] * %b[6] + %a[7] * %b[7] + %c
  def int_amdgcn_sudot8 :
  ClangBuiltin<"__builtin_amdgcn_sudot8">,
  PureIntrinsic<
    [llvm_i32_ty], // %r
    [
````
- **L3393 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty   // %clamp`.
  **L3393 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty   // %clamp`。
- **L3394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3394 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3395 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L3395 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L3396 EN**: Executes a standalone statement or declaration: `>;`.
  **L3396 CN**: 执行一条独立语句或声明：`>;`。
- **L3397 EN**: Blank line separating nearby declarations or logic blocks.
  **L3397 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3398 EN**: Comment explains nearby logic, invariants, or intent: `u32 %r = llvm.amdgcn.udot8(v8u4 (as u32) %a, v8u4 (as u32) %b, u32 %c, i1 %clamp)`.
  **L3398 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`u32 %r = llvm.amdgcn.udot8(v8u4 (as u32) %a, v8u4 (as u32) %b, u32 %c, i1 %clamp)`。
- **L3399 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] +`.
  **L3399 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] +`。
- **L3400 EN**: Comment explains nearby logic, invariants, or intent: `%a[4] * %b[4] + %a[5] * %b[5] + %a[6] * %b[6] + %a[7] * %b[7] + %c`.
  **L3400 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%a[4] * %b[4] + %a[5] * %b[5] + %a[6] * %b[6] + %a[7] * %b[7] + %c`。
- **L3401 EN**: Declares TableGen def `int_amdgcn_udot8`.
  **L3401 CN**: 声明 TableGen def `int_amdgcn_udot8`。
- **L3402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_udot8">,`.
  **L3402 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_udot8">,`。
- **L3403 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3403 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3404 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // %r`.
  **L3404 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // %r`。
- **L3405 EN**: Continues the surrounding expression or declaration: `[`.
  **L3405 CN**: 继续构造周围的表达式或声明：`[`。
- **L3406 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %a`.
  **L3406 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %a`。
- **L3407 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %b`.
  **L3407 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %b`。
- **L3408 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %c`.
  **L3408 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %c`。
- **L3409 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty   // %clamp`.
  **L3409 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty   // %clamp`。
- **L3410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3410 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3411 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<3>>]`.
  **L3411 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<3>>]`。
- **L3412 EN**: Executes a standalone statement or declaration: `>;`.
  **L3412 CN**: 执行一条独立语句或声明：`>;`。
- **L3413 EN**: Blank line separating nearby declarations or logic blocks.
  **L3413 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3414 EN**: Comment explains nearby logic, invariants, or intent: `i32 %r = llvm.amdgcn.sudot8(i1 %a_sign, v8i4 (as i32) %a, i1 %b_sign, v8i4 (as i32) %b, i32 %c, i1 %clamp)`.
  **L3414 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i32 %r = llvm.amdgcn.sudot8(i1 %a_sign, v8i4 (as i32) %a, i1 %b_sign, v8i4 (as i32) %b, i32 %c, i1 %clamp)`。
- **L3415 EN**: Comment explains nearby logic, invariants, or intent: `Treat input as signed (_sign = 1) or unsigned (_sign = 0).`.
  **L3415 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Treat input as signed (_sign = 1) or unsigned (_sign = 0).`。
- **L3416 EN**: Comment explains nearby logic, invariants, or intent: `a[i in 0. . . 7] = (%a_sign ? a.i4[i] : promoteToSigned(a.u4[i]));`.
  **L3416 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a[i in 0. . . 7] = (%a_sign ? a.i4[i] : promoteToSigned(a.u4[i]));`。
- **L3417 EN**: Comment explains nearby logic, invariants, or intent: `b[i in 0. . . 7] = (%b_sign ? b.i4[i] : promoteToSigned(b.u4[i]));`.
  **L3417 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`b[i in 0. . . 7] = (%b_sign ? b.i4[i] : promoteToSigned(b.u4[i]));`。
- **L3418 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] +`.
  **L3418 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] +`。
- **L3419 EN**: Comment explains nearby logic, invariants, or intent: `%a[4] * %b[4] + %a[5] * %b[5] + %a[6] * %b[6] + %a[7] * %b[7] + %c`.
  **L3419 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%a[4] * %b[4] + %a[5] * %b[5] + %a[6] * %b[6] + %a[7] * %b[7] + %c`。
- **L3420 EN**: Declares TableGen def `int_amdgcn_sudot8`.
  **L3420 CN**: 声明 TableGen def `int_amdgcn_sudot8`。
- **L3421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_sudot8">,`.
  **L3421 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_sudot8">,`。
- **L3422 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3422 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3423 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // %r`.
  **L3423 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // %r`。
- **L3424 EN**: Continues the surrounding expression or declaration: `[`.
  **L3424 CN**: 继续构造周围的表达式或声明：`[`。

### Lines 3425-3456

````tablegen
      llvm_i1_ty,  // %a_sign
      llvm_i32_ty, // %a
      llvm_i1_ty,  // %b_sign
      llvm_i32_ty, // %b
      llvm_i32_ty, // %c
      llvm_i1_ty   // %clamp
    ],
    [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>]
  >;

// f32 %r = llvm.amdgcn.dot4.f32.type_a.type_b (v4type_a (as i32) %a, v4type_b (as i32) %b, f32 %c)
//   %r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c
class AMDGPU8bitFloatDot4Intrinsic :
  ClangBuiltin<!subst("int", "__builtin", NAME)>,
  PureIntrinsic<
    [llvm_float_ty], // %r
    [
      llvm_i32_ty,   // %a
      llvm_i32_ty,   // %b
      llvm_float_ty, // %c
    ]
  >;

def int_amdgcn_dot4_f32_fp8_bf8 : AMDGPU8bitFloatDot4Intrinsic;
def int_amdgcn_dot4_f32_bf8_fp8 : AMDGPU8bitFloatDot4Intrinsic;
def int_amdgcn_dot4_f32_fp8_fp8 : AMDGPU8bitFloatDot4Intrinsic;
def int_amdgcn_dot4_f32_bf8_bf8 : AMDGPU8bitFloatDot4Intrinsic;

//===----------------------------------------------------------------------===//
// gfx908 intrinsics
// ===----------------------------------------------------------------------===//

````
- **L3425 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,  // %a_sign`.
  **L3425 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,  // %a_sign`。
- **L3426 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %a`.
  **L3426 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %a`。
- **L3427 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,  // %b_sign`.
  **L3427 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,  // %b_sign`。
- **L3428 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %b`.
  **L3428 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %b`。
- **L3429 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // %c`.
  **L3429 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // %c`。
- **L3430 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty   // %clamp`.
  **L3430 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty   // %clamp`。
- **L3431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3431 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3432 EN**: Continues the surrounding expression or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>]`.
  **L3432 CN**: 继续构造周围的表达式或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>]`。
- **L3433 EN**: Executes a standalone statement or declaration: `>;`.
  **L3433 CN**: 执行一条独立语句或声明：`>;`。
- **L3434 EN**: Blank line separating nearby declarations or logic blocks.
  **L3434 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3435 EN**: Comment explains nearby logic, invariants, or intent: `f32 %r = llvm.amdgcn.dot4.f32.type_a.type_b (v4type_a (as i32) %a, v4type_b (as i32) %b, f32 %c)`.
  **L3435 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`f32 %r = llvm.amdgcn.dot4.f32.type_a.type_b (v4type_a (as i32) %a, v4type_b (as i32) %b, f32 %c)`。
- **L3436 EN**: Comment explains nearby logic, invariants, or intent: `%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c`.
  **L3436 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%r = %a[0] * %b[0] + %a[1] * %b[1] + %a[2] * %b[2] + %a[3] * %b[3] + %c`。
- **L3437 EN**: Declares class `AMDGPU8bitFloatDot4Intrinsic`.
  **L3437 CN**: 声明 class `AMDGPU8bitFloatDot4Intrinsic`。
- **L3438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<!subst("int", "__builtin", NAME)>,`.
  **L3438 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<!subst("int", "__builtin", NAME)>,`。
- **L3439 EN**: Continues the surrounding expression or declaration: `PureIntrinsic<`.
  **L3439 CN**: 继续构造周围的表达式或声明：`PureIntrinsic<`。
- **L3440 EN**: Continues the surrounding expression or declaration: `[llvm_float_ty], // %r`.
  **L3440 CN**: 继续构造周围的表达式或声明：`[llvm_float_ty], // %r`。
- **L3441 EN**: Continues the surrounding expression or declaration: `[`.
  **L3441 CN**: 继续构造周围的表达式或声明：`[`。
- **L3442 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // %a`.
  **L3442 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // %a`。
- **L3443 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // %b`.
  **L3443 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // %b`。
- **L3444 EN**: Continues the surrounding expression or declaration: `llvm_float_ty, // %c`.
  **L3444 CN**: 继续构造周围的表达式或声明：`llvm_float_ty, // %c`。
- **L3445 EN**: Continues the surrounding expression or declaration: `]`.
  **L3445 CN**: 继续构造周围的表达式或声明：`]`。
- **L3446 EN**: Executes a standalone statement or declaration: `>;`.
  **L3446 CN**: 执行一条独立语句或声明：`>;`。
- **L3447 EN**: Blank line separating nearby declarations or logic blocks.
  **L3447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3448 EN**: Declares TableGen def `int_amdgcn_dot4_f32_fp8_bf8`.
  **L3448 CN**: 声明 TableGen def `int_amdgcn_dot4_f32_fp8_bf8`。
- **L3449 EN**: Declares TableGen def `int_amdgcn_dot4_f32_bf8_fp8`.
  **L3449 CN**: 声明 TableGen def `int_amdgcn_dot4_f32_bf8_fp8`。
- **L3450 EN**: Declares TableGen def `int_amdgcn_dot4_f32_fp8_fp8`.
  **L3450 CN**: 声明 TableGen def `int_amdgcn_dot4_f32_fp8_fp8`。
- **L3451 EN**: Declares TableGen def `int_amdgcn_dot4_f32_bf8_bf8`.
  **L3451 CN**: 声明 TableGen def `int_amdgcn_dot4_f32_bf8_bf8`。
- **L3452 EN**: Blank line separating nearby declarations or logic blocks.
  **L3452 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3453 EN**: Banner comment marking a file or section boundary.
  **L3453 CN**: 横幅注释，用于标记文件或章节边界。
- **L3454 EN**: Comment explains nearby logic, invariants, or intent: `gfx908 intrinsics`.
  **L3454 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx908 intrinsics`。
- **L3455 EN**: Comment explains nearby logic, invariants, or intent: `===----------------------------------------------------------------------===//`.
  **L3455 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`===----------------------------------------------------------------------===//`。
- **L3456 EN**: Blank line separating nearby declarations or logic blocks.
  **L3456 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3457-3488

````tablegen
// llvm.amdgcn.mfma.*.* vdst, srcA, srcB, srcC, cbsz, abid, blgp
class AMDGPUMfmaIntrinsic<LLVMType DestTy, LLVMType SrcABTy> :
  ClangBuiltin<!subst("int", "__builtin", NAME)>,
  DefaultAttrsIntrinsic<[DestTy],
            [SrcABTy, SrcABTy, DestTy,
             llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrConvergent, IntrNoMem, IntrNoCreateUndefOrPoison,
             ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;


// srcA's format is determined by cbsz. srcB's format is determined by
// blgp.
//
// These should be <8 x i32> for f8 formats, <6 x i32> for f6 formats,
// and <4 x i32> for f4 formats. It is invalid to use a format that
// requires more registers than the corresponding vector type (e.g. it
// is illegal to use <6 x i32> in operand 0 if cbsz specifies an f8
// format that requires 8 registers).
class AMDGPUMfmaScaleIntrinsic<LLVMType DestTy> :
  DefaultAttrsIntrinsic<[DestTy],
            [llvm_anyvector_ty, llvm_anyvector_ty, DestTy,
             llvm_i32_ty, // cbsz
             llvm_i32_ty, // blgp
             // llvm_i1_ty, // TODO: neg_src2
             // llvm_i1_ty, // TODO: abs_src2
             // llvm_i1_ty, // TODO: clamp
             llvm_i32_ty, // op_sel (A matrix scale, 2-bits) // TODO: Make i2?
             llvm_i32_ty, // v_mfma_ld_scale_b32 src0 (A matrix scale)
             llvm_i32_ty, // op_sel (B matrix scale, 2-bits) // TODO: Make i2?
             llvm_i32_ty  // v_mfma_ld_scale_b32 src1 (B matrix scale)
            ],
            [IntrConvergent, IntrNoMem, IntrNoCreateUndefOrPoison,
````
- **L3457 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.mfma.*.* vdst, srcA, srcB, srcC, cbsz, abid, blgp`.
  **L3457 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.mfma.*.* vdst, srcA, srcB, srcC, cbsz, abid, blgp`。
- **L3458 EN**: Declares class `AMDGPUMfmaIntrinsic<LLVMType`.
  **L3458 CN**: 声明 class `AMDGPUMfmaIntrinsic<LLVMType`。
- **L3459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<!subst("int", "__builtin", NAME)>,`.
  **L3459 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<!subst("int", "__builtin", NAME)>,`。
- **L3460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[DestTy],`.
  **L3460 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[DestTy],`。
- **L3461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[SrcABTy, SrcABTy, DestTy,`.
  **L3461 CN**: 继续一个多行参数列表、初始化器或聚合项：`[SrcABTy, SrcABTy, DestTy,`。
- **L3462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L3462 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L3463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrNoMem, IntrNoCreateUndefOrPoison,`.
  **L3463 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrNoMem, IntrNoCreateUndefOrPoison,`。
- **L3464 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`.
  **L3464 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`。
- **L3465 EN**: Blank line separating nearby declarations or logic blocks.
  **L3465 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3466 EN**: Blank line separating nearby declarations or logic blocks.
  **L3466 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3467 EN**: Comment explains nearby logic, invariants, or intent: `srcA's format is determined by cbsz. srcB's format is determined by`.
  **L3467 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`srcA's format is determined by cbsz. srcB's format is determined by`。
- **L3468 EN**: Comment explains nearby logic, invariants, or intent: `blgp.`.
  **L3468 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`blgp.`。
- **L3469 EN**: Separator comment used for visual grouping.
  **L3469 CN**: 用于视觉分组的分隔注释。
- **L3470 EN**: Comment explains nearby logic, invariants, or intent: `These should be <8 x i32> for f8 formats, <6 x i32> for f6 formats,`.
  **L3470 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These should be <8 x i32> for f8 formats, <6 x i32> for f6 formats,`。
- **L3471 EN**: Comment explains nearby logic, invariants, or intent: `and <4 x i32> for f4 formats. It is invalid to use a format that`.
  **L3471 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and <4 x i32> for f4 formats. It is invalid to use a format that`。
- **L3472 EN**: Comment explains nearby logic, invariants, or intent: `requires more registers than the corresponding vector type (e.g. it`.
  **L3472 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`requires more registers than the corresponding vector type (e.g. it`。
- **L3473 EN**: Comment explains nearby logic, invariants, or intent: `is illegal to use <6 x i32> in operand 0 if cbsz specifies an f8`.
  **L3473 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is illegal to use <6 x i32> in operand 0 if cbsz specifies an f8`。
- **L3474 EN**: Comment explains nearby logic, invariants, or intent: `format that requires 8 registers).`.
  **L3474 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`format that requires 8 registers).`。
- **L3475 EN**: Declares class `AMDGPUMfmaScaleIntrinsic<LLVMType`.
  **L3475 CN**: 声明 class `AMDGPUMfmaScaleIntrinsic<LLVMType`。
- **L3476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[DestTy],`.
  **L3476 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[DestTy],`。
- **L3477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_anyvector_ty, DestTy,`.
  **L3477 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_anyvector_ty, DestTy,`。
- **L3478 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // cbsz`.
  **L3478 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // cbsz`。
- **L3479 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // blgp`.
  **L3479 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // blgp`。
- **L3480 EN**: Comment records a pending task or caution: `llvm_i1_ty, // TODO: neg_src2`.
  **L3480 CN**: 注释记录了待办事项或注意点：`llvm_i1_ty, // TODO: neg_src2`。
- **L3481 EN**: Comment records a pending task or caution: `llvm_i1_ty, // TODO: abs_src2`.
  **L3481 CN**: 注释记录了待办事项或注意点：`llvm_i1_ty, // TODO: abs_src2`。
- **L3482 EN**: Comment records a pending task or caution: `llvm_i1_ty, // TODO: clamp`.
  **L3482 CN**: 注释记录了待办事项或注意点：`llvm_i1_ty, // TODO: clamp`。
- **L3483 EN**: Continues logic associated with callable symbol `op_sel`.
  **L3483 CN**: 继续与可调用符号 `op_sel` 相关的逻辑。
- **L3484 EN**: Continues logic associated with callable symbol `src0`.
  **L3484 CN**: 继续与可调用符号 `src0` 相关的逻辑。
- **L3485 EN**: Continues logic associated with callable symbol `op_sel`.
  **L3485 CN**: 继续与可调用符号 `op_sel` 相关的逻辑。
- **L3486 EN**: Continues logic associated with callable symbol `src1`.
  **L3486 CN**: 继续与可调用符号 `src1` 相关的逻辑。
- **L3487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3487 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrNoMem, IntrNoCreateUndefOrPoison,`.
  **L3488 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrNoMem, IntrNoCreateUndefOrPoison,`。

### Lines 3489-3520

````tablegen
             ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>,
             ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<7>>
             ]>;

defset list<Intrinsic> AMDGPUMFMAIntrinsics908 = {
def int_amdgcn_mfma_f32_32x32x1f32  : AMDGPUMfmaIntrinsic<llvm_v32f32_ty, llvm_float_ty>;
def int_amdgcn_mfma_f32_16x16x1f32  : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_float_ty>;
def int_amdgcn_mfma_f32_4x4x1f32    : AMDGPUMfmaIntrinsic<llvm_v4f32_ty,  llvm_float_ty>;
def int_amdgcn_mfma_f32_32x32x2f32  : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_float_ty>;
def int_amdgcn_mfma_f32_16x16x4f32  : AMDGPUMfmaIntrinsic<llvm_v4f32_ty,  llvm_float_ty>;
def int_amdgcn_mfma_f32_32x32x4f16  : AMDGPUMfmaIntrinsic<llvm_v32f32_ty, llvm_v4f16_ty>;
def int_amdgcn_mfma_f32_16x16x4f16  : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_v4f16_ty>;
def int_amdgcn_mfma_f32_4x4x4f16    : AMDGPUMfmaIntrinsic<llvm_v4f32_ty,  llvm_v4f16_ty>;
def int_amdgcn_mfma_f32_32x32x8f16  : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_v4f16_ty>;
def int_amdgcn_mfma_f32_16x16x16f16 : AMDGPUMfmaIntrinsic<llvm_v4f32_ty,  llvm_v4f16_ty>;
def int_amdgcn_mfma_i32_32x32x4i8   : AMDGPUMfmaIntrinsic<llvm_v32i32_ty, llvm_i32_ty>;
def int_amdgcn_mfma_i32_16x16x4i8   : AMDGPUMfmaIntrinsic<llvm_v16i32_ty, llvm_i32_ty>;
def int_amdgcn_mfma_i32_4x4x4i8     : AMDGPUMfmaIntrinsic<llvm_v4i32_ty,  llvm_i32_ty>;
def int_amdgcn_mfma_i32_32x32x8i8   : AMDGPUMfmaIntrinsic<llvm_v16i32_ty, llvm_i32_ty>;
def int_amdgcn_mfma_i32_16x16x16i8  : AMDGPUMfmaIntrinsic<llvm_v4i32_ty,  llvm_i32_ty>;
def int_amdgcn_mfma_f32_32x32x2bf16 : AMDGPUMfmaIntrinsic<llvm_v32f32_ty, llvm_v2i16_ty>;
def int_amdgcn_mfma_f32_16x16x2bf16 : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_v2i16_ty>;
def int_amdgcn_mfma_f32_4x4x2bf16   : AMDGPUMfmaIntrinsic<llvm_v4f32_ty,  llvm_v2i16_ty>;
def int_amdgcn_mfma_f32_32x32x4bf16 : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_v2i16_ty>;
def int_amdgcn_mfma_f32_16x16x8bf16 : AMDGPUMfmaIntrinsic<llvm_v4f32_ty,  llvm_v2i16_ty>;
}

//===----------------------------------------------------------------------===//
// gfx90a intrinsics
// ===----------------------------------------------------------------------===//

defset list<Intrinsic> AMDGPUMFMAIntrinsics90A = {
````
- **L3489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>,`.
  **L3489 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>,`。
- **L3490 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<7>>`.
  **L3490 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<7>>`。
- **L3491 EN**: Executes a standalone statement or declaration: `]>;`.
  **L3491 CN**: 执行一条独立语句或声明：`]>;`。
- **L3492 EN**: Blank line separating nearby declarations or logic blocks.
  **L3492 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3493 EN**: Continues the surrounding expression or declaration: `defset list<Intrinsic> AMDGPUMFMAIntrinsics908 = {`.
  **L3493 CN**: 继续构造周围的表达式或声明：`defset list<Intrinsic> AMDGPUMFMAIntrinsics908 = {`。
- **L3494 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x1f32`.
  **L3494 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x1f32`。
- **L3495 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x1f32`.
  **L3495 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x1f32`。
- **L3496 EN**: Declares TableGen def `int_amdgcn_mfma_f32_4x4x1f32`.
  **L3496 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_4x4x1f32`。
- **L3497 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x2f32`.
  **L3497 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x2f32`。
- **L3498 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x4f32`.
  **L3498 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x4f32`。
- **L3499 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x4f16`.
  **L3499 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x4f16`。
- **L3500 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x4f16`.
  **L3500 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x4f16`。
- **L3501 EN**: Declares TableGen def `int_amdgcn_mfma_f32_4x4x4f16`.
  **L3501 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_4x4x4f16`。
- **L3502 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x8f16`.
  **L3502 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x8f16`。
- **L3503 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x16f16`.
  **L3503 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x16f16`。
- **L3504 EN**: Declares TableGen def `int_amdgcn_mfma_i32_32x32x4i8`.
  **L3504 CN**: 声明 TableGen def `int_amdgcn_mfma_i32_32x32x4i8`。
- **L3505 EN**: Declares TableGen def `int_amdgcn_mfma_i32_16x16x4i8`.
  **L3505 CN**: 声明 TableGen def `int_amdgcn_mfma_i32_16x16x4i8`。
- **L3506 EN**: Declares TableGen def `int_amdgcn_mfma_i32_4x4x4i8`.
  **L3506 CN**: 声明 TableGen def `int_amdgcn_mfma_i32_4x4x4i8`。
- **L3507 EN**: Declares TableGen def `int_amdgcn_mfma_i32_32x32x8i8`.
  **L3507 CN**: 声明 TableGen def `int_amdgcn_mfma_i32_32x32x8i8`。
- **L3508 EN**: Declares TableGen def `int_amdgcn_mfma_i32_16x16x16i8`.
  **L3508 CN**: 声明 TableGen def `int_amdgcn_mfma_i32_16x16x16i8`。
- **L3509 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x2bf16`.
  **L3509 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x2bf16`。
- **L3510 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x2bf16`.
  **L3510 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x2bf16`。
- **L3511 EN**: Declares TableGen def `int_amdgcn_mfma_f32_4x4x2bf16`.
  **L3511 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_4x4x2bf16`。
- **L3512 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x4bf16`.
  **L3512 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x4bf16`。
- **L3513 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x8bf16`.
  **L3513 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x8bf16`。
- **L3514 EN**: Closes the current lexical scope or compound statement.
  **L3514 CN**: 结束当前词法作用域或复合语句块。
- **L3515 EN**: Blank line separating nearby declarations or logic blocks.
  **L3515 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3516 EN**: Banner comment marking a file or section boundary.
  **L3516 CN**: 横幅注释，用于标记文件或章节边界。
- **L3517 EN**: Comment explains nearby logic, invariants, or intent: `gfx90a intrinsics`.
  **L3517 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx90a intrinsics`。
- **L3518 EN**: Comment explains nearby logic, invariants, or intent: `===----------------------------------------------------------------------===//`.
  **L3518 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`===----------------------------------------------------------------------===//`。
- **L3519 EN**: Blank line separating nearby declarations or logic blocks.
  **L3519 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3520 EN**: Continues the surrounding expression or declaration: `defset list<Intrinsic> AMDGPUMFMAIntrinsics90A = {`.
  **L3520 CN**: 继续构造周围的表达式或声明：`defset list<Intrinsic> AMDGPUMFMAIntrinsics90A = {`。

### Lines 3521-3552

````tablegen
def int_amdgcn_mfma_f32_32x32x4bf16_1k  : AMDGPUMfmaIntrinsic<llvm_v32f32_ty, llvm_v4i16_ty>;
def int_amdgcn_mfma_f32_16x16x4bf16_1k  : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_v4i16_ty>;
def int_amdgcn_mfma_f32_4x4x4bf16_1k    : AMDGPUMfmaIntrinsic<llvm_v4f32_ty,  llvm_v4i16_ty>;
def int_amdgcn_mfma_f32_32x32x8bf16_1k  : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_v4i16_ty>;
def int_amdgcn_mfma_f32_16x16x16bf16_1k : AMDGPUMfmaIntrinsic<llvm_v4f32_ty,  llvm_v4i16_ty>;

// Note: in gfx942 BLGP argument is replaced by NEG bitfield in the DGEMM MFMA.
//       Three bits corresponding to the neg modifier applied to the respective
//       source operand.
def int_amdgcn_mfma_f64_16x16x4f64      : AMDGPUMfmaIntrinsic<llvm_v4f64_ty,  llvm_double_ty>;
def int_amdgcn_mfma_f64_4x4x4f64        : AMDGPUMfmaIntrinsic<llvm_double_ty, llvm_double_ty>;
}

//===----------------------------------------------------------------------===//
// gfx942 intrinsics
// ===----------------------------------------------------------------------===//

class AMDGPUMFp8MfmaIntrinsic<LLVMType DestTy> :
  AMDGPUMfmaIntrinsic<DestTy, llvm_i64_ty>;

multiclass AMDGPUMFp8MfmaIntrinsic<LLVMType DestTy> {
  foreach kind = ["bf8_bf8", "bf8_fp8", "fp8_bf8", "fp8_fp8"] in
    def NAME#"_"#kind : AMDGPUMFp8MfmaIntrinsic<DestTy>;
}

// llvm.amdgcn.smfmac.?32.* vdst, srcA, srcB, srcC, index, cbsz, abid
class AMDGPUMSmfmacIntrinsic<LLVMType DestTy, LLVMType SrcA, LLVMType SrcB> :
  ClangBuiltin<!subst("int", "__builtin", NAME)>,
  DefaultAttrsIntrinsic<[DestTy],
            [SrcA, SrcB, DestTy, llvm_i32_ty,
             llvm_i32_ty, llvm_i32_ty],
            [IntrConvergent, IntrNoMem, IntrNoCreateUndefOrPoison,
````
- **L3521 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x4bf16_1k`.
  **L3521 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x4bf16_1k`。
- **L3522 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x4bf16_1k`.
  **L3522 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x4bf16_1k`。
- **L3523 EN**: Declares TableGen def `int_amdgcn_mfma_f32_4x4x4bf16_1k`.
  **L3523 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_4x4x4bf16_1k`。
- **L3524 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x8bf16_1k`.
  **L3524 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x8bf16_1k`。
- **L3525 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x16bf16_1k`.
  **L3525 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x16bf16_1k`。
- **L3526 EN**: Blank line separating nearby declarations or logic blocks.
  **L3526 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3527 EN**: Comment explains nearby logic, invariants, or intent: `Note: in gfx942 BLGP argument is replaced by NEG bitfield in the DGEMM MFMA.`.
  **L3527 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: in gfx942 BLGP argument is replaced by NEG bitfield in the DGEMM MFMA.`。
- **L3528 EN**: Comment explains nearby logic, invariants, or intent: `Three bits corresponding to the neg modifier applied to the respective`.
  **L3528 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Three bits corresponding to the neg modifier applied to the respective`。
- **L3529 EN**: Comment explains nearby logic, invariants, or intent: `source operand.`.
  **L3529 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`source operand.`。
- **L3530 EN**: Declares TableGen def `int_amdgcn_mfma_f64_16x16x4f64`.
  **L3530 CN**: 声明 TableGen def `int_amdgcn_mfma_f64_16x16x4f64`。
- **L3531 EN**: Declares TableGen def `int_amdgcn_mfma_f64_4x4x4f64`.
  **L3531 CN**: 声明 TableGen def `int_amdgcn_mfma_f64_4x4x4f64`。
- **L3532 EN**: Closes the current lexical scope or compound statement.
  **L3532 CN**: 结束当前词法作用域或复合语句块。
- **L3533 EN**: Blank line separating nearby declarations or logic blocks.
  **L3533 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3534 EN**: Banner comment marking a file or section boundary.
  **L3534 CN**: 横幅注释，用于标记文件或章节边界。
- **L3535 EN**: Comment explains nearby logic, invariants, or intent: `gfx942 intrinsics`.
  **L3535 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx942 intrinsics`。
- **L3536 EN**: Comment explains nearby logic, invariants, or intent: `===----------------------------------------------------------------------===//`.
  **L3536 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`===----------------------------------------------------------------------===//`。
- **L3537 EN**: Blank line separating nearby declarations or logic blocks.
  **L3537 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3538 EN**: Declares class `AMDGPUMFp8MfmaIntrinsic<LLVMType`.
  **L3538 CN**: 声明 class `AMDGPUMFp8MfmaIntrinsic<LLVMType`。
- **L3539 EN**: Executes a standalone statement or declaration: `AMDGPUMfmaIntrinsic<DestTy, llvm_i64_ty>;`.
  **L3539 CN**: 执行一条独立语句或声明：`AMDGPUMfmaIntrinsic<DestTy, llvm_i64_ty>;`。
- **L3540 EN**: Blank line separating nearby declarations or logic blocks.
  **L3540 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3541 EN**: Declares TableGen multiclass `AMDGPUMFp8MfmaIntrinsic`.
  **L3541 CN**: 声明 TableGen multiclass `AMDGPUMFp8MfmaIntrinsic`。
- **L3542 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3542 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3543 EN**: Declares TableGen def `NAME#"_"#kind`.
  **L3543 CN**: 声明 TableGen def `NAME#"_"#kind`。
- **L3544 EN**: Closes the current lexical scope or compound statement.
  **L3544 CN**: 结束当前词法作用域或复合语句块。
- **L3545 EN**: Blank line separating nearby declarations or logic blocks.
  **L3545 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3546 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.smfmac.?32.* vdst, srcA, srcB, srcC, index, cbsz, abid`.
  **L3546 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.smfmac.?32.* vdst, srcA, srcB, srcC, index, cbsz, abid`。
- **L3547 EN**: Declares class `AMDGPUMSmfmacIntrinsic<LLVMType`.
  **L3547 CN**: 声明 class `AMDGPUMSmfmacIntrinsic<LLVMType`。
- **L3548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<!subst("int", "__builtin", NAME)>,`.
  **L3548 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<!subst("int", "__builtin", NAME)>,`。
- **L3549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[DestTy],`.
  **L3549 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[DestTy],`。
- **L3550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[SrcA, SrcB, DestTy, llvm_i32_ty,`.
  **L3550 CN**: 继续一个多行参数列表、初始化器或聚合项：`[SrcA, SrcB, DestTy, llvm_i32_ty,`。
- **L3551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L3551 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。
- **L3552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrNoMem, IntrNoCreateUndefOrPoison,`.
  **L3552 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrNoMem, IntrNoCreateUndefOrPoison,`。

### Lines 3553-3584

````tablegen
             ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;

class AMDGPUMFp8SmfmacIntrinsic<LLVMType DestTy> :
  AMDGPUMSmfmacIntrinsic<DestTy, llvm_v2i32_ty, llvm_v4i32_ty>;

multiclass AMDGPUMFp8SmfmacIntrinsic<LLVMType DestTy> {
  foreach kind = ["bf8_bf8", "bf8_fp8", "fp8_bf8", "fp8_fp8"] in
    def NAME#"_"#kind : AMDGPUMFp8SmfmacIntrinsic<DestTy>;
}

defset list<Intrinsic> AMDGPUMFMAIntrinsics940 = {
def int_amdgcn_mfma_i32_16x16x32_i8     : AMDGPUMfmaIntrinsic<llvm_v4i32_ty,  llvm_i64_ty>;
def int_amdgcn_mfma_i32_32x32x16_i8     : AMDGPUMfmaIntrinsic<llvm_v16i32_ty, llvm_i64_ty>;
def int_amdgcn_mfma_f32_16x16x8_xf32    : AMDGPUMfmaIntrinsic<llvm_v4f32_ty,  llvm_v2f32_ty>;
def int_amdgcn_mfma_f32_32x32x4_xf32    : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_v2f32_ty>;

defm int_amdgcn_mfma_f32_16x16x32 : AMDGPUMFp8MfmaIntrinsic<llvm_v4f32_ty>;
defm int_amdgcn_mfma_f32_32x32x16 : AMDGPUMFp8MfmaIntrinsic<llvm_v16f32_ty>;

def int_amdgcn_smfmac_f32_16x16x32_f16  : AMDGPUMSmfmacIntrinsic<llvm_v4f32_ty,  llvm_v4f16_ty, llvm_v8f16_ty>;
def int_amdgcn_smfmac_f32_32x32x16_f16  : AMDGPUMSmfmacIntrinsic<llvm_v16f32_ty, llvm_v4f16_ty, llvm_v8f16_ty>;
def int_amdgcn_smfmac_f32_16x16x32_bf16 : AMDGPUMSmfmacIntrinsic<llvm_v4f32_ty,  llvm_v4i16_ty, llvm_v8i16_ty>;
def int_amdgcn_smfmac_f32_32x32x16_bf16 : AMDGPUMSmfmacIntrinsic<llvm_v16f32_ty, llvm_v4i16_ty, llvm_v8i16_ty>;
def int_amdgcn_smfmac_i32_16x16x64_i8   : AMDGPUMSmfmacIntrinsic<llvm_v4i32_ty,  llvm_v2i32_ty, llvm_v4i32_ty>;
def int_amdgcn_smfmac_i32_32x32x32_i8   : AMDGPUMSmfmacIntrinsic<llvm_v16i32_ty, llvm_v2i32_ty, llvm_v4i32_ty>;

defm int_amdgcn_smfmac_f32_16x16x64 : AMDGPUMFp8SmfmacIntrinsic<llvm_v4f32_ty>;
defm int_amdgcn_smfmac_f32_32x32x32 : AMDGPUMFp8SmfmacIntrinsic<llvm_v16f32_ty>;
}

// llvm.amdgcn.cvt.f32.bf8 float vdst, int srcA, imm byte_sel [0..3]
// byte_sel selects byte from srcA.
````
- **L3553 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`.
  **L3553 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`。
- **L3554 EN**: Blank line separating nearby declarations or logic blocks.
  **L3554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3555 EN**: Declares class `AMDGPUMFp8SmfmacIntrinsic<LLVMType`.
  **L3555 CN**: 声明 class `AMDGPUMFp8SmfmacIntrinsic<LLVMType`。
- **L3556 EN**: Executes a standalone statement or declaration: `AMDGPUMSmfmacIntrinsic<DestTy, llvm_v2i32_ty, llvm_v4i32_ty>;`.
  **L3556 CN**: 执行一条独立语句或声明：`AMDGPUMSmfmacIntrinsic<DestTy, llvm_v2i32_ty, llvm_v4i32_ty>;`。
- **L3557 EN**: Blank line separating nearby declarations or logic blocks.
  **L3557 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3558 EN**: Declares TableGen multiclass `AMDGPUMFp8SmfmacIntrinsic`.
  **L3558 CN**: 声明 TableGen multiclass `AMDGPUMFp8SmfmacIntrinsic`。
- **L3559 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3559 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3560 EN**: Declares TableGen def `NAME#"_"#kind`.
  **L3560 CN**: 声明 TableGen def `NAME#"_"#kind`。
- **L3561 EN**: Closes the current lexical scope or compound statement.
  **L3561 CN**: 结束当前词法作用域或复合语句块。
- **L3562 EN**: Blank line separating nearby declarations or logic blocks.
  **L3562 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3563 EN**: Continues the surrounding expression or declaration: `defset list<Intrinsic> AMDGPUMFMAIntrinsics940 = {`.
  **L3563 CN**: 继续构造周围的表达式或声明：`defset list<Intrinsic> AMDGPUMFMAIntrinsics940 = {`。
- **L3564 EN**: Declares TableGen def `int_amdgcn_mfma_i32_16x16x32_i8`.
  **L3564 CN**: 声明 TableGen def `int_amdgcn_mfma_i32_16x16x32_i8`。
- **L3565 EN**: Declares TableGen def `int_amdgcn_mfma_i32_32x32x16_i8`.
  **L3565 CN**: 声明 TableGen def `int_amdgcn_mfma_i32_32x32x16_i8`。
- **L3566 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x8_xf32`.
  **L3566 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x8_xf32`。
- **L3567 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x4_xf32`.
  **L3567 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x4_xf32`。
- **L3568 EN**: Blank line separating nearby declarations or logic blocks.
  **L3568 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3569 EN**: Declares TableGen defm `int_amdgcn_mfma_f32_16x16x32`.
  **L3569 CN**: 声明 TableGen defm `int_amdgcn_mfma_f32_16x16x32`。
- **L3570 EN**: Declares TableGen defm `int_amdgcn_mfma_f32_32x32x16`.
  **L3570 CN**: 声明 TableGen defm `int_amdgcn_mfma_f32_32x32x16`。
- **L3571 EN**: Blank line separating nearby declarations or logic blocks.
  **L3571 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3572 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_16x16x32_f16`.
  **L3572 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_16x16x32_f16`。
- **L3573 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_32x32x16_f16`.
  **L3573 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_32x32x16_f16`。
- **L3574 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_16x16x32_bf16`.
  **L3574 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_16x16x32_bf16`。
- **L3575 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_32x32x16_bf16`.
  **L3575 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_32x32x16_bf16`。
- **L3576 EN**: Declares TableGen def `int_amdgcn_smfmac_i32_16x16x64_i8`.
  **L3576 CN**: 声明 TableGen def `int_amdgcn_smfmac_i32_16x16x64_i8`。
- **L3577 EN**: Declares TableGen def `int_amdgcn_smfmac_i32_32x32x32_i8`.
  **L3577 CN**: 声明 TableGen def `int_amdgcn_smfmac_i32_32x32x32_i8`。
- **L3578 EN**: Blank line separating nearby declarations or logic blocks.
  **L3578 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3579 EN**: Declares TableGen defm `int_amdgcn_smfmac_f32_16x16x64`.
  **L3579 CN**: 声明 TableGen defm `int_amdgcn_smfmac_f32_16x16x64`。
- **L3580 EN**: Declares TableGen defm `int_amdgcn_smfmac_f32_32x32x32`.
  **L3580 CN**: 声明 TableGen defm `int_amdgcn_smfmac_f32_32x32x32`。
- **L3581 EN**: Closes the current lexical scope or compound statement.
  **L3581 CN**: 结束当前词法作用域或复合语句块。
- **L3582 EN**: Blank line separating nearby declarations or logic blocks.
  **L3582 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3583 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.f32.bf8 float vdst, int srcA, imm byte_sel [0..3]`.
  **L3583 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.f32.bf8 float vdst, int srcA, imm byte_sel [0..3]`。
- **L3584 EN**: Comment explains nearby logic, invariants, or intent: `byte_sel selects byte from srcA.`.
  **L3584 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`byte_sel selects byte from srcA.`。

### Lines 3585-3616

````tablegen
def int_amdgcn_cvt_f32_bf8 : ClangBuiltin<"__builtin_amdgcn_cvt_f32_bf8">,
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<1>>]>;

// llvm.amdgcn.cvt.f32.fp8 float vdst, int srcA, imm byte_sel [0..3]
def int_amdgcn_cvt_f32_fp8 : ClangBuiltin<"__builtin_amdgcn_cvt_f32_fp8">,
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<1>>]>;

// llvm.amdgcn.cvt.f32.fp8.e5m3 float vdst, int srcA, imm byte_sel [0..3]
def int_amdgcn_cvt_f32_fp8_e5m3 : ClangBuiltin<"__builtin_amdgcn_cvt_f32_fp8_e5m3">,
  DefaultAttrsIntrinsic<[llvm_float_ty],
            [llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<1>>]>;

// llvm.amdgcn.cvt.pk.f32.bf8 float2 vdst, int srcA, imm word_sel
// word_sel = 1 selects 2 high bytes, 0 selects 2 low bytes.
def int_amdgcn_cvt_pk_f32_bf8 : ClangBuiltin<"__builtin_amdgcn_cvt_pk_f32_bf8">,
  PureIntrinsic<[llvm_v2f32_ty],
            [llvm_i32_ty, llvm_i1_ty],
            [ImmArg<ArgIndex<1>>]>;

// llvm.amdgcn.cvt.pk.f32.fp8 float2 vdst, int srcA, imm word_sel.
def int_amdgcn_cvt_pk_f32_fp8 : ClangBuiltin<"__builtin_amdgcn_cvt_pk_f32_fp8">,
  PureIntrinsic<[llvm_v2f32_ty],
            [llvm_i32_ty, llvm_i1_ty],
            [ImmArg<ArgIndex<1>>]>;

// llvm.amdgcn.cvt.pk.bf8.f32 int vdst, float srcA, float srcB, int old, imm word_sel
// word_sel = 1 selects 2 high bytes in the vdst, 0 selects 2 low bytes.
````
- **L3585 EN**: Declares TableGen def `int_amdgcn_cvt_f32_bf8`.
  **L3585 CN**: 声明 TableGen def `int_amdgcn_cvt_f32_bf8`。
- **L3586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L3586 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L3587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3587 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3588 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3588 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L3589 EN**: Blank line separating nearby declarations or logic blocks.
  **L3589 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3590 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.f32.fp8 float vdst, int srcA, imm byte_sel [0..3]`.
  **L3590 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.f32.fp8 float vdst, int srcA, imm byte_sel [0..3]`。
- **L3591 EN**: Declares TableGen def `int_amdgcn_cvt_f32_fp8`.
  **L3591 CN**: 声明 TableGen def `int_amdgcn_cvt_f32_fp8`。
- **L3592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L3592 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L3593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3593 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3594 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3594 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L3595 EN**: Blank line separating nearby declarations or logic blocks.
  **L3595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3596 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.f32.fp8.e5m3 float vdst, int srcA, imm byte_sel [0..3]`.
  **L3596 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.f32.fp8.e5m3 float vdst, int srcA, imm byte_sel [0..3]`。
- **L3597 EN**: Declares TableGen def `int_amdgcn_cvt_f32_fp8_e5m3`.
  **L3597 CN**: 声明 TableGen def `int_amdgcn_cvt_f32_fp8_e5m3`。
- **L3598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L3598 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L3599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3599 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3600 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3600 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L3601 EN**: Blank line separating nearby declarations or logic blocks.
  **L3601 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3602 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.pk.f32.bf8 float2 vdst, int srcA, imm word_sel`.
  **L3602 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.pk.f32.bf8 float2 vdst, int srcA, imm word_sel`。
- **L3603 EN**: Comment explains nearby logic, invariants, or intent: `word_sel = 1 selects 2 high bytes, 0 selects 2 low bytes.`.
  **L3603 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`word_sel = 1 selects 2 high bytes, 0 selects 2 low bytes.`。
- **L3604 EN**: Declares TableGen def `int_amdgcn_cvt_pk_f32_bf8`.
  **L3604 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_f32_bf8`。
- **L3605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_v2f32_ty],`.
  **L3605 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_v2f32_ty],`。
- **L3606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i1_ty],`.
  **L3606 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i1_ty],`。
- **L3607 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<1>>]>;`.
  **L3607 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<1>>]>;`。
- **L3608 EN**: Blank line separating nearby declarations or logic blocks.
  **L3608 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3609 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.pk.f32.fp8 float2 vdst, int srcA, imm word_sel.`.
  **L3609 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.pk.f32.fp8 float2 vdst, int srcA, imm word_sel.`。
- **L3610 EN**: Declares TableGen def `int_amdgcn_cvt_pk_f32_fp8`.
  **L3610 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_f32_fp8`。
- **L3611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_v2f32_ty],`.
  **L3611 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_v2f32_ty],`。
- **L3612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i1_ty],`.
  **L3612 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i1_ty],`。
- **L3613 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<1>>]>;`.
  **L3613 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<1>>]>;`。
- **L3614 EN**: Blank line separating nearby declarations or logic blocks.
  **L3614 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3615 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.pk.bf8.f32 int vdst, float srcA, float srcB, int old, imm word_sel`.
  **L3615 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.pk.bf8.f32 int vdst, float srcA, float srcB, int old, imm word_sel`。
- **L3616 EN**: Comment explains nearby logic, invariants, or intent: `word_sel = 1 selects 2 high bytes in the vdst, 0 selects 2 low bytes.`.
  **L3616 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`word_sel = 1 selects 2 high bytes in the vdst, 0 selects 2 low bytes.`。

### Lines 3617-3648

````tablegen
def int_amdgcn_cvt_pk_bf8_f32 : ClangBuiltin<"__builtin_amdgcn_cvt_pk_bf8_f32">,
  PureIntrinsic<[llvm_i32_ty],
            [llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i1_ty],
            [ImmArg<ArgIndex<3>>]>;

// llvm.amdgcn.cvt.pk.fp8.f32 int vdst, float srcA, float srcB, int old, imm word_sel
def int_amdgcn_cvt_pk_fp8_f32 : ClangBuiltin<"__builtin_amdgcn_cvt_pk_fp8_f32">,
  PureIntrinsic<[llvm_i32_ty],
            [llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i1_ty],
            [ImmArg<ArgIndex<3>>]>;

// llvm.amdgcn.cvt.pk.fp8.f32.e5m3 int vdst, float srcA, float srcB, int old, imm word_sel
def int_amdgcn_cvt_pk_fp8_f32_e5m3 : ClangBuiltin<"__builtin_amdgcn_cvt_pk_fp8_f32_e5m3">,
  PureIntrinsic<[llvm_i32_ty],
            [llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i1_ty],
            [ImmArg<ArgIndex<3>>]>;

// llvm.amdgcn.cvt.sr.bf8.f32 int vdst, float srcA, int srcB, int old, imm byte_sel [0..3]
// byte_sel selects byte to write into vdst.
def int_amdgcn_cvt_sr_bf8_f32 : ClangBuiltin<"__builtin_amdgcn_cvt_sr_bf8_f32">,
  PureIntrinsic<[llvm_i32_ty],
            [llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [ImmArg<ArgIndex<3>>]>;

// llvm.amdgcn.cvt.sr.fp8.f32 int vdst, float srcA, int srcB, int old, imm byte_sel [0..3]
def int_amdgcn_cvt_sr_fp8_f32 : ClangBuiltin<"__builtin_amdgcn_cvt_sr_fp8_f32">,
  PureIntrinsic<[llvm_i32_ty],
            [llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [ImmArg<ArgIndex<3>>]>;

// llvm.amdgcn.cvt.sr.fp8.f32.e5m3 int vdst, float srcA, int srcB, int old, imm byte_sel [0..3]
def int_amdgcn_cvt_sr_fp8_f32_e5m3 : ClangBuiltin<"__builtin_amdgcn_cvt_sr_fp8_f32_e5m3">,
````
- **L3617 EN**: Declares TableGen def `int_amdgcn_cvt_pk_bf8_f32`.
  **L3617 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_bf8_f32`。
- **L3618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_i32_ty],`.
  **L3618 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_i32_ty],`。
- **L3619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i1_ty],`.
  **L3619 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i1_ty],`。
- **L3620 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<3>>]>;`.
  **L3620 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<3>>]>;`。
- **L3621 EN**: Blank line separating nearby declarations or logic blocks.
  **L3621 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3622 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.pk.fp8.f32 int vdst, float srcA, float srcB, int old, imm word_sel`.
  **L3622 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.pk.fp8.f32 int vdst, float srcA, float srcB, int old, imm word_sel`。
- **L3623 EN**: Declares TableGen def `int_amdgcn_cvt_pk_fp8_f32`.
  **L3623 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_fp8_f32`。
- **L3624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_i32_ty],`.
  **L3624 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_i32_ty],`。
- **L3625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i1_ty],`.
  **L3625 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i1_ty],`。
- **L3626 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<3>>]>;`.
  **L3626 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<3>>]>;`。
- **L3627 EN**: Blank line separating nearby declarations or logic blocks.
  **L3627 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3628 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.pk.fp8.f32.e5m3 int vdst, float srcA, float srcB, int old, imm word_sel`.
  **L3628 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.pk.fp8.f32.e5m3 int vdst, float srcA, float srcB, int old, imm word_sel`。
- **L3629 EN**: Declares TableGen def `int_amdgcn_cvt_pk_fp8_f32_e5m3`.
  **L3629 CN**: 声明 TableGen def `int_amdgcn_cvt_pk_fp8_f32_e5m3`。
- **L3630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_i32_ty],`.
  **L3630 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_i32_ty],`。
- **L3631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i1_ty],`.
  **L3631 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_i32_ty, llvm_i1_ty],`。
- **L3632 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<3>>]>;`.
  **L3632 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<3>>]>;`。
- **L3633 EN**: Blank line separating nearby declarations or logic blocks.
  **L3633 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3634 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.sr.bf8.f32 int vdst, float srcA, int srcB, int old, imm byte_sel [0..3]`.
  **L3634 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.sr.bf8.f32 int vdst, float srcA, int srcB, int old, imm byte_sel [0..3]`。
- **L3635 EN**: Comment explains nearby logic, invariants, or intent: `byte_sel selects byte to write into vdst.`.
  **L3635 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`byte_sel selects byte to write into vdst.`。
- **L3636 EN**: Declares TableGen def `int_amdgcn_cvt_sr_bf8_f32`.
  **L3636 CN**: 声明 TableGen def `int_amdgcn_cvt_sr_bf8_f32`。
- **L3637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_i32_ty],`.
  **L3637 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_i32_ty],`。
- **L3638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L3638 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L3639 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<3>>]>;`.
  **L3639 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<3>>]>;`。
- **L3640 EN**: Blank line separating nearby declarations or logic blocks.
  **L3640 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3641 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.sr.fp8.f32 int vdst, float srcA, int srcB, int old, imm byte_sel [0..3]`.
  **L3641 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.sr.fp8.f32 int vdst, float srcA, int srcB, int old, imm byte_sel [0..3]`。
- **L3642 EN**: Declares TableGen def `int_amdgcn_cvt_sr_fp8_f32`.
  **L3642 CN**: 声明 TableGen def `int_amdgcn_cvt_sr_fp8_f32`。
- **L3643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_i32_ty],`.
  **L3643 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_i32_ty],`。
- **L3644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L3644 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L3645 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<3>>]>;`.
  **L3645 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<3>>]>;`。
- **L3646 EN**: Blank line separating nearby declarations or logic blocks.
  **L3646 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3647 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.sr.fp8.f32.e5m3 int vdst, float srcA, int srcB, int old, imm byte_sel [0..3]`.
  **L3647 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.sr.fp8.f32.e5m3 int vdst, float srcA, int srcB, int old, imm byte_sel [0..3]`。
- **L3648 EN**: Declares TableGen def `int_amdgcn_cvt_sr_fp8_f32_e5m3`.
  **L3648 CN**: 声明 TableGen def `int_amdgcn_cvt_sr_fp8_f32_e5m3`。

### Lines 3649-3680

````tablegen
  PureIntrinsic<[llvm_i32_ty],
            [llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [ImmArg<ArgIndex<3>>]>;

// llvm.amdgcn.cvt.off.fp32.i4 int srcA
def int_amdgcn_cvt_off_f32_i4: ClangBuiltin<"__builtin_amdgcn_cvt_off_f32_i4">,
  PureIntrinsic<[llvm_float_ty],
            [llvm_i32_ty]>;

//===----------------------------------------------------------------------===//
// gfx950 intrinsics
//===----------------------------------------------------------------------===//

defset list<Intrinsic> AMDGPUMFMAIntrinsics950 = {
def int_amdgcn_mfma_f32_16x16x32_f16 : AMDGPUMfmaIntrinsic<llvm_v4f32_ty, llvm_v8f16_ty>;
def int_amdgcn_mfma_f32_32x32x16_f16 : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_v8f16_ty>;
def int_amdgcn_mfma_i32_16x16x64_i8 : AMDGPUMfmaIntrinsic<llvm_v4i32_ty, llvm_v4i32_ty>;
def int_amdgcn_mfma_i32_32x32x32_i8 : AMDGPUMfmaIntrinsic<llvm_v16i32_ty, llvm_v4i32_ty>;
def int_amdgcn_mfma_f32_16x16x32_bf16 : AMDGPUMfmaIntrinsic<llvm_v4f32_ty, llvm_v8bf16_ty>;
def int_amdgcn_mfma_f32_32x32x16_bf16 : AMDGPUMfmaIntrinsic<llvm_v16f32_ty, llvm_v8bf16_ty>;
def int_amdgcn_mfma_scale_f32_16x16x128_f8f6f4 : AMDGPUMfmaScaleIntrinsic<llvm_v4f32_ty>;
def int_amdgcn_mfma_scale_f32_32x32x64_f8f6f4 : AMDGPUMfmaScaleIntrinsic<llvm_v16f32_ty>;
def int_amdgcn_smfmac_f32_16x16x64_f16 : AMDGPUMSmfmacIntrinsic<llvm_v4f32_ty, llvm_v8f16_ty, llvm_v16f16_ty>;
def int_amdgcn_smfmac_f32_32x32x32_f16 : AMDGPUMSmfmacIntrinsic<llvm_v16f32_ty, llvm_v8f16_ty, llvm_v16f16_ty>;
def int_amdgcn_smfmac_f32_16x16x64_bf16 : AMDGPUMSmfmacIntrinsic<llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v16bf16_ty>;
def int_amdgcn_smfmac_f32_32x32x32_bf16 : AMDGPUMSmfmacIntrinsic<llvm_v16f32_ty, llvm_v8bf16_ty, llvm_v16bf16_ty>;
def int_amdgcn_smfmac_i32_16x16x128_i8 : AMDGPUMSmfmacIntrinsic<llvm_v4i32_ty,  llvm_v4i32_ty, llvm_v8i32_ty>;
def int_amdgcn_smfmac_i32_32x32x64_i8 : AMDGPUMSmfmacIntrinsic<llvm_v16i32_ty, llvm_v4i32_ty, llvm_v8i32_ty>;
def int_amdgcn_smfmac_f32_16x16x128_bf8_bf8 : AMDGPUMSmfmacIntrinsic<llvm_v4f32_ty, llvm_v4i32_ty, llvm_v8i32_ty>;
def int_amdgcn_smfmac_f32_16x16x128_bf8_fp8 : AMDGPUMSmfmacIntrinsic<llvm_v4f32_ty, llvm_v4i32_ty, llvm_v8i32_ty>;
def int_amdgcn_smfmac_f32_16x16x128_fp8_bf8 : AMDGPUMSmfmacIntrinsic<llvm_v4f32_ty, llvm_v4i32_ty, llvm_v8i32_ty>;
def int_amdgcn_smfmac_f32_16x16x128_fp8_fp8 : AMDGPUMSmfmacIntrinsic<llvm_v4f32_ty, llvm_v4i32_ty, llvm_v8i32_ty>;
````
- **L3649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_i32_ty],`.
  **L3649 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_i32_ty],`。
- **L3650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L3650 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L3651 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<3>>]>;`.
  **L3651 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<3>>]>;`。
- **L3652 EN**: Blank line separating nearby declarations or logic blocks.
  **L3652 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3653 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.off.fp32.i4 int srcA`.
  **L3653 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.off.fp32.i4 int srcA`。
- **L3654 EN**: Declares TableGen def `int_amdgcn_cvt_off_f32_i4`.
  **L3654 CN**: 声明 TableGen def `int_amdgcn_cvt_off_f32_i4`。
- **L3655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_float_ty],`.
  **L3655 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_float_ty],`。
- **L3656 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty]>;`.
  **L3656 CN**: 执行一条独立语句或声明：`[llvm_i32_ty]>;`。
- **L3657 EN**: Blank line separating nearby declarations or logic blocks.
  **L3657 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3658 EN**: Banner comment marking a file or section boundary.
  **L3658 CN**: 横幅注释，用于标记文件或章节边界。
- **L3659 EN**: Comment explains nearby logic, invariants, or intent: `gfx950 intrinsics`.
  **L3659 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx950 intrinsics`。
- **L3660 EN**: Banner comment marking a file or section boundary.
  **L3660 CN**: 横幅注释，用于标记文件或章节边界。
- **L3661 EN**: Blank line separating nearby declarations or logic blocks.
  **L3661 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3662 EN**: Continues the surrounding expression or declaration: `defset list<Intrinsic> AMDGPUMFMAIntrinsics950 = {`.
  **L3662 CN**: 继续构造周围的表达式或声明：`defset list<Intrinsic> AMDGPUMFMAIntrinsics950 = {`。
- **L3663 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x32_f16`.
  **L3663 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x32_f16`。
- **L3664 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x16_f16`.
  **L3664 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x16_f16`。
- **L3665 EN**: Declares TableGen def `int_amdgcn_mfma_i32_16x16x64_i8`.
  **L3665 CN**: 声明 TableGen def `int_amdgcn_mfma_i32_16x16x64_i8`。
- **L3666 EN**: Declares TableGen def `int_amdgcn_mfma_i32_32x32x32_i8`.
  **L3666 CN**: 声明 TableGen def `int_amdgcn_mfma_i32_32x32x32_i8`。
- **L3667 EN**: Declares TableGen def `int_amdgcn_mfma_f32_16x16x32_bf16`.
  **L3667 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_16x16x32_bf16`。
- **L3668 EN**: Declares TableGen def `int_amdgcn_mfma_f32_32x32x16_bf16`.
  **L3668 CN**: 声明 TableGen def `int_amdgcn_mfma_f32_32x32x16_bf16`。
- **L3669 EN**: Declares TableGen def `int_amdgcn_mfma_scale_f32_16x16x128_f8f6f4`.
  **L3669 CN**: 声明 TableGen def `int_amdgcn_mfma_scale_f32_16x16x128_f8f6f4`。
- **L3670 EN**: Declares TableGen def `int_amdgcn_mfma_scale_f32_32x32x64_f8f6f4`.
  **L3670 CN**: 声明 TableGen def `int_amdgcn_mfma_scale_f32_32x32x64_f8f6f4`。
- **L3671 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_16x16x64_f16`.
  **L3671 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_16x16x64_f16`。
- **L3672 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_32x32x32_f16`.
  **L3672 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_32x32x32_f16`。
- **L3673 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_16x16x64_bf16`.
  **L3673 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_16x16x64_bf16`。
- **L3674 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_32x32x32_bf16`.
  **L3674 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_32x32x32_bf16`。
- **L3675 EN**: Declares TableGen def `int_amdgcn_smfmac_i32_16x16x128_i8`.
  **L3675 CN**: 声明 TableGen def `int_amdgcn_smfmac_i32_16x16x128_i8`。
- **L3676 EN**: Declares TableGen def `int_amdgcn_smfmac_i32_32x32x64_i8`.
  **L3676 CN**: 声明 TableGen def `int_amdgcn_smfmac_i32_32x32x64_i8`。
- **L3677 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_16x16x128_bf8_bf8`.
  **L3677 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_16x16x128_bf8_bf8`。
- **L3678 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_16x16x128_bf8_fp8`.
  **L3678 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_16x16x128_bf8_fp8`。
- **L3679 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_16x16x128_fp8_bf8`.
  **L3679 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_16x16x128_fp8_bf8`。
- **L3680 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_16x16x128_fp8_fp8`.
  **L3680 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_16x16x128_fp8_fp8`。

### Lines 3681-3712

````tablegen
def int_amdgcn_smfmac_f32_32x32x64_bf8_bf8 : AMDGPUMSmfmacIntrinsic<llvm_v16f32_ty, llvm_v4i32_ty, llvm_v8i32_ty>;
def int_amdgcn_smfmac_f32_32x32x64_bf8_fp8 : AMDGPUMSmfmacIntrinsic<llvm_v16f32_ty, llvm_v4i32_ty, llvm_v8i32_ty>;
def int_amdgcn_smfmac_f32_32x32x64_fp8_bf8 : AMDGPUMSmfmacIntrinsic<llvm_v16f32_ty, llvm_v4i32_ty, llvm_v8i32_ty>;
def int_amdgcn_smfmac_f32_32x32x64_fp8_fp8 : AMDGPUMSmfmacIntrinsic<llvm_v16f32_ty, llvm_v4i32_ty, llvm_v8i32_ty>;
}

// { vdst_new, vsrc_new } llvm.amdgcn.permlane16.swap <vdst_old> <vsrc_old> <fi> <bound_control>
def int_amdgcn_permlane16_swap :
  Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                                         llvm_i1_ty, llvm_i1_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn,
             ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrNoCallback, IntrNoFree]>;

// { vdst_new, vsrc_new } llvm.amdgcn.permlane32.swap <vdst_old> <vsrc_old> <fi> <bound_control>
def int_amdgcn_permlane32_swap :
  Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                                         llvm_i1_ty, llvm_i1_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn,
             ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.ashr_pk_i8_i32 int vdst, int src0, int src1 int src2
def int_amdgcn_ashr_pk_i8_i32 : ClangBuiltin<"__builtin_amdgcn_ashr_pk_i8_i32">,
  PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

// llvm.amdgcn.ashr_pk_u8_i32 int vdst, int src0, int src1 int src2
def int_amdgcn_ashr_pk_u8_i32 : ClangBuiltin<"__builtin_amdgcn_ashr_pk_u8_i32">,
  PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

//===----------------------------------------------------------------------===//
// gfx1250 intrinsics
// ===----------------------------------------------------------------------===//

````
- **L3681 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_32x32x64_bf8_bf8`.
  **L3681 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_32x32x64_bf8_bf8`。
- **L3682 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_32x32x64_bf8_fp8`.
  **L3682 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_32x32x64_bf8_fp8`。
- **L3683 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_32x32x64_fp8_bf8`.
  **L3683 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_32x32x64_fp8_bf8`。
- **L3684 EN**: Declares TableGen def `int_amdgcn_smfmac_f32_32x32x64_fp8_fp8`.
  **L3684 CN**: 声明 TableGen def `int_amdgcn_smfmac_f32_32x32x64_fp8_fp8`。
- **L3685 EN**: Closes the current lexical scope or compound statement.
  **L3685 CN**: 结束当前词法作用域或复合语句块。
- **L3686 EN**: Blank line separating nearby declarations or logic blocks.
  **L3686 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3687 EN**: Comment explains nearby logic, invariants, or intent: `{ vdst_new, vsrc_new } llvm.amdgcn.permlane16.swap <vdst_old> <vsrc_old> <fi> <bound_control>`.
  **L3687 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`{ vdst_new, vsrc_new } llvm.amdgcn.permlane16.swap <vdst_old> <vsrc_old> <fi> <bound_control>`。
- **L3688 EN**: Declares TableGen def `int_amdgcn_permlane16_swap`.
  **L3688 CN**: 声明 TableGen def `int_amdgcn_permlane16_swap`。
- **L3689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L3689 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L3690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i1_ty, llvm_i1_ty],`.
  **L3690 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i1_ty, llvm_i1_ty],`。
- **L3691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,`.
  **L3691 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,`。
- **L3692 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrNoCallback, IntrNoFree]>;`.
  **L3692 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrNoCallback, IntrNoFree]>;`。
- **L3693 EN**: Blank line separating nearby declarations or logic blocks.
  **L3693 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3694 EN**: Comment explains nearby logic, invariants, or intent: `{ vdst_new, vsrc_new } llvm.amdgcn.permlane32.swap <vdst_old> <vsrc_old> <fi> <bound_control>`.
  **L3694 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`{ vdst_new, vsrc_new } llvm.amdgcn.permlane32.swap <vdst_old> <vsrc_old> <fi> <bound_control>`。
- **L3695 EN**: Declares TableGen def `int_amdgcn_permlane32_swap`.
  **L3695 CN**: 声明 TableGen def `int_amdgcn_permlane32_swap`。
- **L3696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L3696 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L3697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i1_ty, llvm_i1_ty],`.
  **L3697 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i1_ty, llvm_i1_ty],`。
- **L3698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn,`.
  **L3698 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn,`。
- **L3699 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrNoCallback, IntrNoFree]>;`.
  **L3699 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrNoCallback, IntrNoFree]>;`。
- **L3700 EN**: Blank line separating nearby declarations or logic blocks.
  **L3700 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3701 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ashr_pk_i8_i32 int vdst, int src0, int src1 int src2`.
  **L3701 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ashr_pk_i8_i32 int vdst, int src0, int src1 int src2`。
- **L3702 EN**: Declares TableGen def `int_amdgcn_ashr_pk_i8_i32`.
  **L3702 CN**: 声明 TableGen def `int_amdgcn_ashr_pk_i8_i32`。
- **L3703 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L3703 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L3704 EN**: Blank line separating nearby declarations or logic blocks.
  **L3704 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3705 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.ashr_pk_u8_i32 int vdst, int src0, int src1 int src2`.
  **L3705 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.ashr_pk_u8_i32 int vdst, int src0, int src1 int src2`。
- **L3706 EN**: Declares TableGen def `int_amdgcn_ashr_pk_u8_i32`.
  **L3706 CN**: 声明 TableGen def `int_amdgcn_ashr_pk_u8_i32`。
- **L3707 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L3707 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L3708 EN**: Blank line separating nearby declarations or logic blocks.
  **L3708 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3709 EN**: Banner comment marking a file or section boundary.
  **L3709 CN**: 横幅注释，用于标记文件或章节边界。
- **L3710 EN**: Comment explains nearby logic, invariants, or intent: `gfx1250 intrinsics`.
  **L3710 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gfx1250 intrinsics`。
- **L3711 EN**: Comment explains nearby logic, invariants, or intent: `===----------------------------------------------------------------------===//`.
  **L3711 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`===----------------------------------------------------------------------===//`。
- **L3712 EN**: Blank line separating nearby declarations or logic blocks.
  **L3712 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3713-3744

````tablegen
// Vanilla cluster sync-barrier
def int_amdgcn_s_cluster_barrier : ClangBuiltin<"__builtin_amdgcn_s_cluster_barrier">,
  Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// Async waits decrement ASYNCcnt and tensor waits decrement TENSORcnt which is
// modeled as InaccessibleMem.
class AMDGPUWaitAsyncIntrinsic :
  Intrinsic<[], [llvm_i16_ty],
  [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>, IntrWillReturn, IntrNoCallback,
   IntrNoFree]>;

def int_amdgcn_s_wait_asynccnt :
    ClangBuiltin<"__builtin_amdgcn_s_wait_asynccnt">, AMDGPUWaitAsyncIntrinsic;
def int_amdgcn_s_wait_tensorcnt :
    ClangBuiltin<"__builtin_amdgcn_s_wait_tensorcnt">, AMDGPUWaitAsyncIntrinsic;

def int_amdgcn_ds_atomic_async_barrier_arrive_b64 :
  ClangBuiltin<"__builtin_amdgcn_ds_atomic_async_barrier_arrive_b64">,
  Intrinsic<[], [local_ptr_ty],
            // Atomically updates LDS and also ASYNC_CNT which is modeled as InaccessibleMem.
            [IntrConvergent, IntrWillReturn, IntrInaccessibleMemOrArgMemOnly],
            "", [SDNPMemOperand]>;

def int_amdgcn_ds_atomic_barrier_arrive_rtn_b64 :
  ClangBuiltin<"__builtin_amdgcn_ds_atomic_barrier_arrive_rtn_b64">,
  Intrinsic<[llvm_i64_ty], [local_ptr_ty, llvm_i64_ty],
            [IntrConvergent, IntrWillReturn, IntrArgMemOnly, NoCapture<ArgIndex<0>>],
            "", [SDNPMemOperand]>;

def int_amdgcn_s_monitor_sleep :
  ClangBuiltin<"__builtin_amdgcn_s_monitor_sleep">,
  DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,
````
- **L3713 EN**: Comment explains nearby logic, invariants, or intent: `Vanilla cluster sync-barrier`.
  **L3713 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vanilla cluster sync-barrier`。
- **L3714 EN**: Declares TableGen def `int_amdgcn_s_cluster_barrier`.
  **L3714 CN**: 声明 TableGen def `int_amdgcn_s_cluster_barrier`。
- **L3715 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3715 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3716 EN**: Blank line separating nearby declarations or logic blocks.
  **L3716 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3717 EN**: Comment explains nearby logic, invariants, or intent: `Async waits decrement ASYNCcnt and tensor waits decrement TENSORcnt which is`.
  **L3717 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Async waits decrement ASYNCcnt and tensor waits decrement TENSORcnt which is`。
- **L3718 EN**: Comment explains nearby logic, invariants, or intent: `modeled as InaccessibleMem.`.
  **L3718 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`modeled as InaccessibleMem.`。
- **L3719 EN**: Declares class `AMDGPUWaitAsyncIntrinsic`.
  **L3719 CN**: 声明 class `AMDGPUWaitAsyncIntrinsic`。
- **L3720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i16_ty],`.
  **L3720 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i16_ty],`。
- **L3721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>, IntrWillReturn, IntrNoCallback,`.
  **L3721 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>, IntrWillReturn, IntrNoCallback,`。
- **L3722 EN**: Executes a standalone statement or declaration: `IntrNoFree]>;`.
  **L3722 CN**: 执行一条独立语句或声明：`IntrNoFree]>;`。
- **L3723 EN**: Blank line separating nearby declarations or logic blocks.
  **L3723 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3724 EN**: Declares TableGen def `int_amdgcn_s_wait_asynccnt`.
  **L3724 CN**: 声明 TableGen def `int_amdgcn_s_wait_asynccnt`。
- **L3725 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_s_wait_asynccnt">, AMDGPUWaitAsyncIntrinsic;`.
  **L3725 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_s_wait_asynccnt">, AMDGPUWaitAsyncIntrinsic;`。
- **L3726 EN**: Declares TableGen def `int_amdgcn_s_wait_tensorcnt`.
  **L3726 CN**: 声明 TableGen def `int_amdgcn_s_wait_tensorcnt`。
- **L3727 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_s_wait_tensorcnt">, AMDGPUWaitAsyncIntrinsic;`.
  **L3727 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_s_wait_tensorcnt">, AMDGPUWaitAsyncIntrinsic;`。
- **L3728 EN**: Blank line separating nearby declarations or logic blocks.
  **L3728 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3729 EN**: Declares TableGen def `int_amdgcn_ds_atomic_async_barrier_arrive_b64`.
  **L3729 CN**: 声明 TableGen def `int_amdgcn_ds_atomic_async_barrier_arrive_b64`。
- **L3730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_atomic_async_barrier_arrive_b64">,`.
  **L3730 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_atomic_async_barrier_arrive_b64">,`。
- **L3731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [local_ptr_ty],`.
  **L3731 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [local_ptr_ty],`。
- **L3732 EN**: Comment explains nearby logic, invariants, or intent: `Atomically updates LDS and also ASYNC_CNT which is modeled as InaccessibleMem.`.
  **L3732 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Atomically updates LDS and also ASYNC_CNT which is modeled as InaccessibleMem.`。
- **L3733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrWillReturn, IntrInaccessibleMemOrArgMemOnly],`.
  **L3733 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrWillReturn, IntrInaccessibleMemOrArgMemOnly],`。
- **L3734 EN**: Executes a standalone statement or declaration: `"", [SDNPMemOperand]>;`.
  **L3734 CN**: 执行一条独立语句或声明：`"", [SDNPMemOperand]>;`。
- **L3735 EN**: Blank line separating nearby declarations or logic blocks.
  **L3735 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3736 EN**: Declares TableGen def `int_amdgcn_ds_atomic_barrier_arrive_rtn_b64`.
  **L3736 CN**: 声明 TableGen def `int_amdgcn_ds_atomic_barrier_arrive_rtn_b64`。
- **L3737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_ds_atomic_barrier_arrive_rtn_b64">,`.
  **L3737 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_ds_atomic_barrier_arrive_rtn_b64">,`。
- **L3738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [local_ptr_ty, llvm_i64_ty],`.
  **L3738 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [local_ptr_ty, llvm_i64_ty],`。
- **L3739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrWillReturn, IntrArgMemOnly, NoCapture<ArgIndex<0>>],`.
  **L3739 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrWillReturn, IntrArgMemOnly, NoCapture<ArgIndex<0>>],`。
- **L3740 EN**: Executes a standalone statement or declaration: `"", [SDNPMemOperand]>;`.
  **L3740 CN**: 执行一条独立语句或声明：`"", [SDNPMemOperand]>;`。
- **L3741 EN**: Blank line separating nearby declarations or logic blocks.
  **L3741 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3742 EN**: Declares TableGen def `int_amdgcn_s_monitor_sleep`.
  **L3742 CN**: 声明 TableGen def `int_amdgcn_s_monitor_sleep`。
- **L3743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_amdgcn_s_monitor_sleep">,`.
  **L3743 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_amdgcn_s_monitor_sleep">,`。
- **L3744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`.
  **L3744 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i16_ty], [ImmArg<ArgIndex<0>>, IntrNoMem,`。

### Lines 3745-3776

````tablegen
                                            IntrHasSideEffects]>;

// llvm.amdgcn.cvt.f16.fp8 half vdst, int srcA, imm byte_sel [0..3]
def int_amdgcn_cvt_f16_fp8 : ClangBuiltin<"__builtin_amdgcn_cvt_f16_fp8">,
  DefaultAttrsIntrinsic<[llvm_half_ty],
            [llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<1>>]>;

// llvm.amdgcn.cvt.f16.bf8 half vdst, int srcA, imm byte_sel [0..3]
def int_amdgcn_cvt_f16_bf8 : ClangBuiltin<"__builtin_amdgcn_cvt_f16_bf8">,
  PureIntrinsic<[llvm_half_ty],
            [llvm_i32_ty, llvm_i32_ty],
            [ImmArg<ArgIndex<1>>]>;

def int_amdgcn_sat_pk4_i4_i8 : ClangBuiltin<"__builtin_amdgcn_sat_pk4_i4_i8">,
  PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty]>;

def int_amdgcn_sat_pk4_u4_u8 : ClangBuiltin<"__builtin_amdgcn_sat_pk4_u4_u8">,
  PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty]>;

// llvm.amdgcn.permlane.bcast <src0> <src1> <src2>
def int_amdgcn_permlane_bcast :
  Intrinsic<[llvm_any_ty],
            [LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.permlane.up <src0> <src1> <src2>
def int_amdgcn_permlane_up :
  Intrinsic<[llvm_any_ty],
            [LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

````
- **L3745 EN**: Executes a standalone statement or declaration: `IntrHasSideEffects]>;`.
  **L3745 CN**: 执行一条独立语句或声明：`IntrHasSideEffects]>;`。
- **L3746 EN**: Blank line separating nearby declarations or logic blocks.
  **L3746 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3747 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.f16.fp8 half vdst, int srcA, imm byte_sel [0..3]`.
  **L3747 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.f16.fp8 half vdst, int srcA, imm byte_sel [0..3]`。
- **L3748 EN**: Declares TableGen def `int_amdgcn_cvt_f16_fp8`.
  **L3748 CN**: 声明 TableGen def `int_amdgcn_cvt_f16_fp8`。
- **L3749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_half_ty],`.
  **L3749 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_half_ty],`。
- **L3750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3750 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3751 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3751 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L3752 EN**: Blank line separating nearby declarations or logic blocks.
  **L3752 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3753 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.cvt.f16.bf8 half vdst, int srcA, imm byte_sel [0..3]`.
  **L3753 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.cvt.f16.bf8 half vdst, int srcA, imm byte_sel [0..3]`。
- **L3754 EN**: Declares TableGen def `int_amdgcn_cvt_f16_bf8`.
  **L3754 CN**: 声明 TableGen def `int_amdgcn_cvt_f16_bf8`。
- **L3755 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_half_ty],`.
  **L3755 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_half_ty],`。
- **L3756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3756 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3757 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<1>>]>;`.
  **L3757 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<1>>]>;`。
- **L3758 EN**: Blank line separating nearby declarations or logic blocks.
  **L3758 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3759 EN**: Declares TableGen def `int_amdgcn_sat_pk4_i4_i8`.
  **L3759 CN**: 声明 TableGen def `int_amdgcn_sat_pk4_i4_i8`。
- **L3760 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty]>;`.
  **L3760 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty]>;`。
- **L3761 EN**: Blank line separating nearby declarations or logic blocks.
  **L3761 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3762 EN**: Declares TableGen def `int_amdgcn_sat_pk4_u4_u8`.
  **L3762 CN**: 声明 TableGen def `int_amdgcn_sat_pk4_u4_u8`。
- **L3763 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty]>;`.
  **L3763 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_i32_ty]>;`。
- **L3764 EN**: Blank line separating nearby declarations or logic blocks.
  **L3764 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3765 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlane.bcast <src0> <src1> <src2>`.
  **L3765 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlane.bcast <src0> <src1> <src2>`。
- **L3766 EN**: Declares TableGen def `int_amdgcn_permlane_bcast`.
  **L3766 CN**: 声明 TableGen def `int_amdgcn_permlane_bcast`。
- **L3767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty],`.
  **L3767 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty],`。
- **L3768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`.
  **L3768 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`。
- **L3769 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3769 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3770 EN**: Blank line separating nearby declarations or logic blocks.
  **L3770 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3771 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlane.up <src0> <src1> <src2>`.
  **L3771 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlane.up <src0> <src1> <src2>`。
- **L3772 EN**: Declares TableGen def `int_amdgcn_permlane_up`.
  **L3772 CN**: 声明 TableGen def `int_amdgcn_permlane_up`。
- **L3773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty],`.
  **L3773 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty],`。
- **L3774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`.
  **L3774 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`。
- **L3775 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3775 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3776 EN**: Blank line separating nearby declarations or logic blocks.
  **L3776 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3777-3808

````tablegen
// llvm.amdgcn.permlane.down <src0> <src1> <src2>
def int_amdgcn_permlane_down :
  Intrinsic<[llvm_any_ty],
            [LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.permlane.xor <src0> <src1> <src2>
def int_amdgcn_permlane_xor :
  Intrinsic<[llvm_any_ty],
            [LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// llvm.amdgcn.permlane.idx.gen <src0> <src1>
def int_amdgcn_permlane_idx_gen : ClangBuiltin<"__builtin_amdgcn_permlane_idx_gen">,
  Intrinsic<[llvm_i32_ty],
            [llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;

def int_amdgcn_perm_pk16_b4_u4 : ClangBuiltin<"__builtin_amdgcn_perm_pk16_b4_u4">,
  PureIntrinsic<[llvm_v2i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_v2i32_ty]>;

def int_amdgcn_perm_pk16_b6_u4 : ClangBuiltin<"__builtin_amdgcn_perm_pk16_b6_u4">,
  PureIntrinsic<[llvm_v3i32_ty], [llvm_i32_ty, llvm_i64_ty, llvm_v2i32_ty]>;

def int_amdgcn_perm_pk16_b8_u4 : ClangBuiltin<"__builtin_amdgcn_perm_pk16_b8_u4">,
  PureIntrinsic<[llvm_v4i32_ty], [llvm_i64_ty, llvm_i64_ty, llvm_v2i32_ty]>;

class AMDGPUAddMinMax<LLVMType Ty, string Name> : ClangBuiltin<"__builtin_amdgcn_"#Name>,
  PureIntrinsic<[Ty], [Ty, Ty, Ty, llvm_i1_ty /* clamp */],
                        [ImmArg<ArgIndex<3>>]>;

def int_amdgcn_add_max_i32 : AMDGPUAddMinMax<llvm_i32_ty, "add_max_i32">;
````
- **L3777 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlane.down <src0> <src1> <src2>`.
  **L3777 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlane.down <src0> <src1> <src2>`。
- **L3778 EN**: Declares TableGen def `int_amdgcn_permlane_down`.
  **L3778 CN**: 声明 TableGen def `int_amdgcn_permlane_down`。
- **L3779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty],`.
  **L3779 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty],`。
- **L3780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`.
  **L3780 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`。
- **L3781 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3781 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3782 EN**: Blank line separating nearby declarations or logic blocks.
  **L3782 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3783 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlane.xor <src0> <src1> <src2>`.
  **L3783 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlane.xor <src0> <src1> <src2>`。
- **L3784 EN**: Declares TableGen def `int_amdgcn_permlane_xor`.
  **L3784 CN**: 声明 TableGen def `int_amdgcn_permlane_xor`。
- **L3785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_any_ty],`.
  **L3785 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_any_ty],`。
- **L3786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`.
  **L3786 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`。
- **L3787 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3787 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3788 EN**: Blank line separating nearby declarations or logic blocks.
  **L3788 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3789 EN**: Comment explains nearby logic, invariants, or intent: `llvm.amdgcn.permlane.idx.gen <src0> <src1>`.
  **L3789 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.amdgcn.permlane.idx.gen <src0> <src1>`。
- **L3790 EN**: Declares TableGen def `int_amdgcn_permlane_idx_gen`.
  **L3790 CN**: 声明 TableGen def `int_amdgcn_permlane_idx_gen`。
- **L3791 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty],`.
  **L3791 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty],`。
- **L3792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3792 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3793 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3793 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3794 EN**: Blank line separating nearby declarations or logic blocks.
  **L3794 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3795 EN**: Declares TableGen def `int_amdgcn_perm_pk16_b4_u4`.
  **L3795 CN**: 声明 TableGen def `int_amdgcn_perm_pk16_b4_u4`。
- **L3796 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_v2i32_ty]>;`.
  **L3796 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_v2i32_ty]>;`。
- **L3797 EN**: Blank line separating nearby declarations or logic blocks.
  **L3797 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3798 EN**: Declares TableGen def `int_amdgcn_perm_pk16_b6_u4`.
  **L3798 CN**: 声明 TableGen def `int_amdgcn_perm_pk16_b6_u4`。
- **L3799 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v3i32_ty], [llvm_i32_ty, llvm_i64_ty, llvm_v2i32_ty]>;`.
  **L3799 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v3i32_ty], [llvm_i32_ty, llvm_i64_ty, llvm_v2i32_ty]>;`。
- **L3800 EN**: Blank line separating nearby declarations or logic blocks.
  **L3800 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3801 EN**: Declares TableGen def `int_amdgcn_perm_pk16_b8_u4`.
  **L3801 CN**: 声明 TableGen def `int_amdgcn_perm_pk16_b8_u4`。
- **L3802 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v4i32_ty], [llvm_i64_ty, llvm_i64_ty, llvm_v2i32_ty]>;`.
  **L3802 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v4i32_ty], [llvm_i64_ty, llvm_i64_ty, llvm_v2i32_ty]>;`。
- **L3803 EN**: Blank line separating nearby declarations or logic blocks.
  **L3803 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3804 EN**: Declares class `AMDGPUAddMinMax<LLVMType`.
  **L3804 CN**: 声明 class `AMDGPUAddMinMax<LLVMType`。
- **L3805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[Ty], [Ty, Ty, Ty, llvm_i1_ty /* clamp */],`.
  **L3805 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[Ty], [Ty, Ty, Ty, llvm_i1_ty /* clamp */],`。
- **L3806 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<3>>]>;`.
  **L3806 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<3>>]>;`。
- **L3807 EN**: Blank line separating nearby declarations or logic blocks.
  **L3807 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3808 EN**: Declares TableGen def `int_amdgcn_add_max_i32`.
  **L3808 CN**: 声明 TableGen def `int_amdgcn_add_max_i32`。

### Lines 3809-3840

````tablegen
def int_amdgcn_add_max_u32 : AMDGPUAddMinMax<llvm_i32_ty, "add_max_u32">;
def int_amdgcn_add_min_i32 : AMDGPUAddMinMax<llvm_i32_ty, "add_min_i32">;
def int_amdgcn_add_min_u32 : AMDGPUAddMinMax<llvm_i32_ty, "add_min_u32">;

def int_amdgcn_pk_add_max_i16 : AMDGPUAddMinMax<llvm_v2i16_ty, "pk_add_max_i16">;
def int_amdgcn_pk_add_max_u16 : AMDGPUAddMinMax<llvm_v2i16_ty, "pk_add_max_u16">;
def int_amdgcn_pk_add_min_i16 : AMDGPUAddMinMax<llvm_v2i16_ty, "pk_add_min_i16">;
def int_amdgcn_pk_add_min_u16 : AMDGPUAddMinMax<llvm_v2i16_ty, "pk_add_min_u16">;

class AMDGPUCooperativeAtomicStore<LLVMType Ty> : Intrinsic <
  [],
  [llvm_anyptr_ty,         // pointer to store to
   Ty,                     // value to store
   llvm_i32_ty,            // C ABI Atomic Ordering ID
   llvm_metadata_ty],      // syncscope
  [IntrWriteMem, WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<2>>,
   IntrNoCallback, IntrNoFree, IntrConvergent],
  "",
  [SDNPMemOperand, SDNPMayStore]
>;

class AMDGPUCooperativeAtomicLoad<LLVMType Ty> : Intrinsic <
  [Ty],
  [llvm_anyptr_ty,         // pointer to load from
   llvm_i32_ty,            // C ABI Atomic Ordering ID
   llvm_metadata_ty],      // syncscope
  [IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>,
   IntrNoCallback, IntrNoFree, IntrConvergent],
  "",
  [SDNPMemOperand, SDNPMayLoad]
>;

````
- **L3809 EN**: Declares TableGen def `int_amdgcn_add_max_u32`.
  **L3809 CN**: 声明 TableGen def `int_amdgcn_add_max_u32`。
- **L3810 EN**: Declares TableGen def `int_amdgcn_add_min_i32`.
  **L3810 CN**: 声明 TableGen def `int_amdgcn_add_min_i32`。
- **L3811 EN**: Declares TableGen def `int_amdgcn_add_min_u32`.
  **L3811 CN**: 声明 TableGen def `int_amdgcn_add_min_u32`。
- **L3812 EN**: Blank line separating nearby declarations or logic blocks.
  **L3812 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3813 EN**: Declares TableGen def `int_amdgcn_pk_add_max_i16`.
  **L3813 CN**: 声明 TableGen def `int_amdgcn_pk_add_max_i16`。
- **L3814 EN**: Declares TableGen def `int_amdgcn_pk_add_max_u16`.
  **L3814 CN**: 声明 TableGen def `int_amdgcn_pk_add_max_u16`。
- **L3815 EN**: Declares TableGen def `int_amdgcn_pk_add_min_i16`.
  **L3815 CN**: 声明 TableGen def `int_amdgcn_pk_add_min_i16`。
- **L3816 EN**: Declares TableGen def `int_amdgcn_pk_add_min_u16`.
  **L3816 CN**: 声明 TableGen def `int_amdgcn_pk_add_min_u16`。
- **L3817 EN**: Blank line separating nearby declarations or logic blocks.
  **L3817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3818 EN**: Declares class `AMDGPUCooperativeAtomicStore<LLVMType`.
  **L3818 CN**: 声明 class `AMDGPUCooperativeAtomicStore<LLVMType`。
- **L3819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L3819 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L3820 EN**: Continues the surrounding expression or declaration: `[llvm_anyptr_ty,         // pointer to store to`.
  **L3820 CN**: 继续构造周围的表达式或声明：`[llvm_anyptr_ty,         // pointer to store to`。
- **L3821 EN**: Continues the surrounding expression or declaration: `Ty,                     // value to store`.
  **L3821 CN**: 继续构造周围的表达式或声明：`Ty,                     // value to store`。
- **L3822 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,            // C ABI Atomic Ordering ID`.
  **L3822 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,            // C ABI Atomic Ordering ID`。
- **L3823 EN**: Continues the surrounding expression or declaration: `llvm_metadata_ty],      // syncscope`.
  **L3823 CN**: 继续构造周围的表达式或声明：`llvm_metadata_ty],      // syncscope`。
- **L3824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem, WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<2>>,`.
  **L3824 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem, WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<2>>,`。
- **L3825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrNoCallback, IntrNoFree, IntrConvergent],`.
  **L3825 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrNoCallback, IntrNoFree, IntrConvergent],`。
- **L3826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"",`.
  **L3826 CN**: 继续一个多行参数列表、初始化器或聚合项：`"",`。
- **L3827 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand, SDNPMayStore]`.
  **L3827 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand, SDNPMayStore]`。
- **L3828 EN**: Executes a standalone statement or declaration: `>;`.
  **L3828 CN**: 执行一条独立语句或声明：`>;`。
- **L3829 EN**: Blank line separating nearby declarations or logic blocks.
  **L3829 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3830 EN**: Declares class `AMDGPUCooperativeAtomicLoad<LLVMType`.
  **L3830 CN**: 声明 class `AMDGPUCooperativeAtomicLoad<LLVMType`。
- **L3831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Ty],`.
  **L3831 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Ty],`。
- **L3832 EN**: Continues the surrounding expression or declaration: `[llvm_anyptr_ty,         // pointer to load from`.
  **L3832 CN**: 继续构造周围的表达式或声明：`[llvm_anyptr_ty,         // pointer to load from`。
- **L3833 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,            // C ABI Atomic Ordering ID`.
  **L3833 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,            // C ABI Atomic Ordering ID`。
- **L3834 EN**: Continues the surrounding expression or declaration: `llvm_metadata_ty],      // syncscope`.
  **L3834 CN**: 继续构造周围的表达式或声明：`llvm_metadata_ty],      // syncscope`。
- **L3835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>,`.
  **L3835 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>,`。
- **L3836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrNoCallback, IntrNoFree, IntrConvergent],`.
  **L3836 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrNoCallback, IntrNoFree, IntrConvergent],`。
- **L3837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"",`.
  **L3837 CN**: 继续一个多行参数列表、初始化器或聚合项：`"",`。
- **L3838 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand, SDNPMayLoad]`.
  **L3838 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand, SDNPMayLoad]`。
- **L3839 EN**: Executes a standalone statement or declaration: `>;`.
  **L3839 CN**: 执行一条独立语句或声明：`>;`。
- **L3840 EN**: Blank line separating nearby declarations or logic blocks.
  **L3840 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3841-3872

````tablegen
// TODO: We may want to drop _relaxed and use an atomic ordering operand instead.
def int_amdgcn_cooperative_atomic_load_32x4B : AMDGPUCooperativeAtomicLoad<llvm_i32_ty>;
def int_amdgcn_cooperative_atomic_store_32x4B : AMDGPUCooperativeAtomicStore<llvm_i32_ty>;
def int_amdgcn_cooperative_atomic_load_16x8B : AMDGPUCooperativeAtomicLoad<llvm_v2i32_ty>;
def int_amdgcn_cooperative_atomic_store_16x8B : AMDGPUCooperativeAtomicStore<llvm_v2i32_ty>;
def int_amdgcn_cooperative_atomic_load_8x16B : AMDGPUCooperativeAtomicLoad<llvm_v4i32_ty>;
def int_amdgcn_cooperative_atomic_store_8x16B : AMDGPUCooperativeAtomicStore<llvm_v4i32_ty>;

//===----------------------------------------------------------------------===//
// Special Intrinsics for backend internal use only. No frontend
// should emit calls to these.
// ===----------------------------------------------------------------------===//
//
// Control-flow intrinsics in LLVM IR are convergent because they represent the
// wave CFG, i.e., sets of threads that are "converged" or "execute in
// lock-step". But they exist during a small window in the lowering process,
// inserted after the structurizer and then translated to equivalent MIR
// pseudos. So rather than create convergence tokens for these builtins, we
// simply mark them as not convergent.
//
// This is really a workaround to allow control flow lowering in the presence of
// convergence control tokens. The corresponding MIR pseudos are marked as
// having side effects, which is sufficient to prevent optimizations without
// having to mark them as convergent.
def int_amdgcn_if : Intrinsic<[llvm_i1_ty, llvm_anyint_ty],
  [llvm_i1_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

def int_amdgcn_else : Intrinsic<[llvm_i1_ty, llvm_anyint_ty],
  [llvm_anyint_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

````
- **L3841 EN**: Comment records a pending task or caution: `TODO: We may want to drop _relaxed and use an atomic ordering operand instead.`.
  **L3841 CN**: 注释记录了待办事项或注意点：`TODO: We may want to drop _relaxed and use an atomic ordering operand instead.`。
- **L3842 EN**: Declares TableGen def `int_amdgcn_cooperative_atomic_load_32x4B`.
  **L3842 CN**: 声明 TableGen def `int_amdgcn_cooperative_atomic_load_32x4B`。
- **L3843 EN**: Declares TableGen def `int_amdgcn_cooperative_atomic_store_32x4B`.
  **L3843 CN**: 声明 TableGen def `int_amdgcn_cooperative_atomic_store_32x4B`。
- **L3844 EN**: Declares TableGen def `int_amdgcn_cooperative_atomic_load_16x8B`.
  **L3844 CN**: 声明 TableGen def `int_amdgcn_cooperative_atomic_load_16x8B`。
- **L3845 EN**: Declares TableGen def `int_amdgcn_cooperative_atomic_store_16x8B`.
  **L3845 CN**: 声明 TableGen def `int_amdgcn_cooperative_atomic_store_16x8B`。
- **L3846 EN**: Declares TableGen def `int_amdgcn_cooperative_atomic_load_8x16B`.
  **L3846 CN**: 声明 TableGen def `int_amdgcn_cooperative_atomic_load_8x16B`。
- **L3847 EN**: Declares TableGen def `int_amdgcn_cooperative_atomic_store_8x16B`.
  **L3847 CN**: 声明 TableGen def `int_amdgcn_cooperative_atomic_store_8x16B`。
- **L3848 EN**: Blank line separating nearby declarations or logic blocks.
  **L3848 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3849 EN**: Banner comment marking a file or section boundary.
  **L3849 CN**: 横幅注释，用于标记文件或章节边界。
- **L3850 EN**: Comment explains nearby logic, invariants, or intent: `Special Intrinsics for backend internal use only. No frontend`.
  **L3850 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Special Intrinsics for backend internal use only. No frontend`。
- **L3851 EN**: Comment explains nearby logic, invariants, or intent: `should emit calls to these.`.
  **L3851 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`should emit calls to these.`。
- **L3852 EN**: Comment explains nearby logic, invariants, or intent: `===----------------------------------------------------------------------===//`.
  **L3852 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`===----------------------------------------------------------------------===//`。
- **L3853 EN**: Separator comment used for visual grouping.
  **L3853 CN**: 用于视觉分组的分隔注释。
- **L3854 EN**: Comment explains nearby logic, invariants, or intent: `Control-flow intrinsics in LLVM IR are convergent because they represent the`.
  **L3854 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Control-flow intrinsics in LLVM IR are convergent because they represent the`。
- **L3855 EN**: Comment explains nearby logic, invariants, or intent: `wave CFG, i.e., sets of threads that are "converged" or "execute in`.
  **L3855 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wave CFG, i.e., sets of threads that are "converged" or "execute in`。
- **L3856 EN**: Comment explains nearby logic, invariants, or intent: `lock-step". But they exist during a small window in the lowering process,`.
  **L3856 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lock-step". But they exist during a small window in the lowering process,`。
- **L3857 EN**: Comment explains nearby logic, invariants, or intent: `inserted after the structurizer and then translated to equivalent MIR`.
  **L3857 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`inserted after the structurizer and then translated to equivalent MIR`。
- **L3858 EN**: Comment explains nearby logic, invariants, or intent: `pseudos. So rather than create convergence tokens for these builtins, we`.
  **L3858 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`pseudos. So rather than create convergence tokens for these builtins, we`。
- **L3859 EN**: Comment explains nearby logic, invariants, or intent: `simply mark them as not convergent.`.
  **L3859 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`simply mark them as not convergent.`。
- **L3860 EN**: Separator comment used for visual grouping.
  **L3860 CN**: 用于视觉分组的分隔注释。
- **L3861 EN**: Comment explains nearby logic, invariants, or intent: `This is really a workaround to allow control flow lowering in the presence of`.
  **L3861 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is really a workaround to allow control flow lowering in the presence of`。
- **L3862 EN**: Comment explains nearby logic, invariants, or intent: `convergence control tokens. The corresponding MIR pseudos are marked as`.
  **L3862 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`convergence control tokens. The corresponding MIR pseudos are marked as`。
- **L3863 EN**: Comment explains nearby logic, invariants, or intent: `having side effects, which is sufficient to prevent optimizations without`.
  **L3863 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`having side effects, which is sufficient to prevent optimizations without`。
- **L3864 EN**: Comment explains nearby logic, invariants, or intent: `having to mark them as convergent.`.
  **L3864 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`having to mark them as convergent.`。
- **L3865 EN**: Declares TableGen def `int_amdgcn_if`.
  **L3865 CN**: 声明 TableGen def `int_amdgcn_if`。
- **L3866 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L3866 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L3867 EN**: Executes a standalone statement or declaration: `>;`.
  **L3867 CN**: 执行一条独立语句或声明：`>;`。
- **L3868 EN**: Blank line separating nearby declarations or logic blocks.
  **L3868 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3869 EN**: Declares TableGen def `int_amdgcn_else`.
  **L3869 CN**: 声明 TableGen def `int_amdgcn_else`。
- **L3870 EN**: Continues the surrounding expression or declaration: `[llvm_anyint_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L3870 CN**: 继续构造周围的表达式或声明：`[llvm_anyint_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L3871 EN**: Executes a standalone statement or declaration: `>;`.
  **L3871 CN**: 执行一条独立语句或声明：`>;`。
- **L3872 EN**: Blank line separating nearby declarations or logic blocks.
  **L3872 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3873-3904

````tablegen
def int_amdgcn_if_break : Intrinsic<[llvm_anyint_ty],
  [llvm_i1_ty, LLVMMatchType<0>],
  [IntrNoMem, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

def int_amdgcn_loop : Intrinsic<[llvm_i1_ty],
  [llvm_anyint_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

def int_amdgcn_end_cf : Intrinsic<[], [llvm_anyint_ty],
  [IntrWillReturn, IntrNoCallback, IntrNoFree]>;

// Represent unreachable in a divergent region.
def int_amdgcn_unreachable : Intrinsic<[], [], [IntrConvergent, IntrNoCallback, IntrNoFree]>;

// Emit 2.5 ulp, no denormal division. Should only be inserted by
// pass based on !fpmath metadata.
def int_amdgcn_fdiv_fast : PureIntrinsic<
  [llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;

// Async instructions increment ASYNCcnt which is modeled as InaccessibleMem.
class AMDGPUAsyncClusterLoadLDS : Intrinsic <
  [],
  [global_ptr_ty,          // Base global pointer to load from
   local_ptr_ty,           // LDS base pointer to store to
   llvm_i32_ty,            // offset
   llvm_i32_ty,            // gfx12+ cachepolicy:
                           //   bits [0-2] = th
                           //   bits [3-4] = scope
   llvm_i32_ty],           // workgroup broadcast mask (to M0)
  [IntrInaccessibleMemOrArgMemOnly, ReadOnly<ArgIndex<0>>, WriteOnly<ArgIndex<1>>,  NoCapture<ArgIndex<0>>,
   NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree],
````
- **L3873 EN**: Declares TableGen def `int_amdgcn_if_break`.
  **L3873 CN**: 声明 TableGen def `int_amdgcn_if_break`。
- **L3874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i1_ty, LLVMMatchType<0>],`.
  **L3874 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i1_ty, LLVMMatchType<0>],`。
- **L3875 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L3875 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L3876 EN**: Executes a standalone statement or declaration: `>;`.
  **L3876 CN**: 执行一条独立语句或声明：`>;`。
- **L3877 EN**: Blank line separating nearby declarations or logic blocks.
  **L3877 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3878 EN**: Declares TableGen def `int_amdgcn_loop`.
  **L3878 CN**: 声明 TableGen def `int_amdgcn_loop`。
- **L3879 EN**: Continues the surrounding expression or declaration: `[llvm_anyint_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L3879 CN**: 继续构造周围的表达式或声明：`[llvm_anyint_ty], [IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L3880 EN**: Executes a standalone statement or declaration: `>;`.
  **L3880 CN**: 执行一条独立语句或声明：`>;`。
- **L3881 EN**: Blank line separating nearby declarations or logic blocks.
  **L3881 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3882 EN**: Declares TableGen def `int_amdgcn_end_cf`.
  **L3882 CN**: 声明 TableGen def `int_amdgcn_end_cf`。
- **L3883 EN**: Executes a standalone statement or declaration: `[IntrWillReturn, IntrNoCallback, IntrNoFree]>;`.
  **L3883 CN**: 执行一条独立语句或声明：`[IntrWillReturn, IntrNoCallback, IntrNoFree]>;`。
- **L3884 EN**: Blank line separating nearby declarations or logic blocks.
  **L3884 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3885 EN**: Comment explains nearby logic, invariants, or intent: `Represent unreachable in a divergent region.`.
  **L3885 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Represent unreachable in a divergent region.`。
- **L3886 EN**: Declares TableGen def `int_amdgcn_unreachable`.
  **L3886 CN**: 声明 TableGen def `int_amdgcn_unreachable`。
- **L3887 EN**: Blank line separating nearby declarations or logic blocks.
  **L3887 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3888 EN**: Comment explains nearby logic, invariants, or intent: `Emit 2.5 ulp, no denormal division. Should only be inserted by`.
  **L3888 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Emit 2.5 ulp, no denormal division. Should only be inserted by`。
- **L3889 EN**: Comment explains nearby logic, invariants, or intent: `pass based on !fpmath metadata.`.
  **L3889 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`pass based on !fpmath metadata.`。
- **L3890 EN**: Declares TableGen def `int_amdgcn_fdiv_fast`.
  **L3890 CN**: 声明 TableGen def `int_amdgcn_fdiv_fast`。
- **L3891 EN**: Executes a standalone statement or declaration: `[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L3891 CN**: 执行一条独立语句或声明：`[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L3892 EN**: Blank line separating nearby declarations or logic blocks.
  **L3892 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3893 EN**: Comment explains nearby logic, invariants, or intent: `Async instructions increment ASYNCcnt which is modeled as InaccessibleMem.`.
  **L3893 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Async instructions increment ASYNCcnt which is modeled as InaccessibleMem.`。
- **L3894 EN**: Declares class `AMDGPUAsyncClusterLoadLDS`.
  **L3894 CN**: 声明 class `AMDGPUAsyncClusterLoadLDS`。
- **L3895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L3895 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L3896 EN**: Continues the surrounding expression or declaration: `[global_ptr_ty,          // Base global pointer to load from`.
  **L3896 CN**: 继续构造周围的表达式或声明：`[global_ptr_ty,          // Base global pointer to load from`。
- **L3897 EN**: Continues the surrounding expression or declaration: `local_ptr_ty,           // LDS base pointer to store to`.
  **L3897 CN**: 继续构造周围的表达式或声明：`local_ptr_ty,           // LDS base pointer to store to`。
- **L3898 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,            // offset`.
  **L3898 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,            // offset`。
- **L3899 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,            // gfx12+ cachepolicy:`.
  **L3899 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,            // gfx12+ cachepolicy:`。
- **L3900 EN**: Comment explains nearby logic, invariants, or intent: `bits [0-2] = th`.
  **L3900 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [0-2] = th`。
- **L3901 EN**: Comment explains nearby logic, invariants, or intent: `bits [3-4] = scope`.
  **L3901 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [3-4] = scope`。
- **L3902 EN**: Continues logic associated with callable symbol `mask`.
  **L3902 CN**: 继续与可调用符号 `mask` 相关的逻辑。
- **L3903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, ReadOnly<ArgIndex<0>>, WriteOnly<ArgIndex<1>>,  NoCapture<ArgIndex<0>>,`.
  **L3903 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, ReadOnly<ArgIndex<0>>, WriteOnly<ArgIndex<1>>,  NoCapture<ArgIndex<0>>,`。
- **L3904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree],`.
  **L3904 CN**: 继续一个多行参数列表、初始化器或聚合项：`NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree],`。

### Lines 3905-3936

````tablegen
  "", [SDNPMemOperand]
>;

class AMDGPUAsyncGlobalLoadToLDS : Intrinsic <
  [],
  [global_ptr_ty,          // Base global pointer to load from
   local_ptr_ty,           // LDS base pointer to store to.
   llvm_i32_ty,            // offset
   llvm_i32_ty],           // gfx12+ cachepolicy:
                           //   bits [0-2] = th
                           //   bits [3-4] = scope
  [IntrInaccessibleMemOrArgMemOnly, ReadOnly<ArgIndex<0>>, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<0>>,
   NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree],
  "", [SDNPMemOperand]
>;

class AMDGPUAsyncGlobalStoreFromLDS : Intrinsic <
  [],
  [global_ptr_ty,          // Base global pointer to store to
   local_ptr_ty,           // LDS base pointer to load from
   llvm_i32_ty,            // offset
   llvm_i32_ty],           // gfx12+ cachepolicy:
                           //   bits [0-2] = th
                           //   bits [3-4] = scope
  [IntrInaccessibleMemOrArgMemOnly, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>, NoCapture<ArgIndex<0>>,
   NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree],
  "", [SDNPMemOperand]
>;

def int_amdgcn_cluster_load_async_to_lds_b8     :
  ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b8">, AMDGPUAsyncClusterLoadLDS;
def int_amdgcn_cluster_load_async_to_lds_b32    :
````
- **L3905 EN**: Continues the surrounding expression or declaration: `"", [SDNPMemOperand]`.
  **L3905 CN**: 继续构造周围的表达式或声明：`"", [SDNPMemOperand]`。
- **L3906 EN**: Executes a standalone statement or declaration: `>;`.
  **L3906 CN**: 执行一条独立语句或声明：`>;`。
- **L3907 EN**: Blank line separating nearby declarations or logic blocks.
  **L3907 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3908 EN**: Declares class `AMDGPUAsyncGlobalLoadToLDS`.
  **L3908 CN**: 声明 class `AMDGPUAsyncGlobalLoadToLDS`。
- **L3909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L3909 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L3910 EN**: Continues the surrounding expression or declaration: `[global_ptr_ty,          // Base global pointer to load from`.
  **L3910 CN**: 继续构造周围的表达式或声明：`[global_ptr_ty,          // Base global pointer to load from`。
- **L3911 EN**: Continues the surrounding expression or declaration: `local_ptr_ty,           // LDS base pointer to store to.`.
  **L3911 CN**: 继续构造周围的表达式或声明：`local_ptr_ty,           // LDS base pointer to store to.`。
- **L3912 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,            // offset`.
  **L3912 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,            // offset`。
- **L3913 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],           // gfx12+ cachepolicy:`.
  **L3913 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],           // gfx12+ cachepolicy:`。
- **L3914 EN**: Comment explains nearby logic, invariants, or intent: `bits [0-2] = th`.
  **L3914 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [0-2] = th`。
- **L3915 EN**: Comment explains nearby logic, invariants, or intent: `bits [3-4] = scope`.
  **L3915 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [3-4] = scope`。
- **L3916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, ReadOnly<ArgIndex<0>>, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`.
  **L3916 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, ReadOnly<ArgIndex<0>>, WriteOnly<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`。
- **L3917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree],`.
  **L3917 CN**: 继续一个多行参数列表、初始化器或聚合项：`NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree],`。
- **L3918 EN**: Continues the surrounding expression or declaration: `"", [SDNPMemOperand]`.
  **L3918 CN**: 继续构造周围的表达式或声明：`"", [SDNPMemOperand]`。
- **L3919 EN**: Executes a standalone statement or declaration: `>;`.
  **L3919 CN**: 执行一条独立语句或声明：`>;`。
- **L3920 EN**: Blank line separating nearby declarations or logic blocks.
  **L3920 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3921 EN**: Declares class `AMDGPUAsyncGlobalStoreFromLDS`.
  **L3921 CN**: 声明 class `AMDGPUAsyncGlobalStoreFromLDS`。
- **L3922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L3922 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L3923 EN**: Continues the surrounding expression or declaration: `[global_ptr_ty,          // Base global pointer to store to`.
  **L3923 CN**: 继续构造周围的表达式或声明：`[global_ptr_ty,          // Base global pointer to store to`。
- **L3924 EN**: Continues the surrounding expression or declaration: `local_ptr_ty,           // LDS base pointer to load from`.
  **L3924 CN**: 继续构造周围的表达式或声明：`local_ptr_ty,           // LDS base pointer to load from`。
- **L3925 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,            // offset`.
  **L3925 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,            // offset`。
- **L3926 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],           // gfx12+ cachepolicy:`.
  **L3926 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],           // gfx12+ cachepolicy:`。
- **L3927 EN**: Comment explains nearby logic, invariants, or intent: `bits [0-2] = th`.
  **L3927 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [0-2] = th`。
- **L3928 EN**: Comment explains nearby logic, invariants, or intent: `bits [3-4] = scope`.
  **L3928 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [3-4] = scope`。
- **L3929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`.
  **L3929 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`。
- **L3930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree],`.
  **L3930 CN**: 继续一个多行参数列表、初始化器或聚合项：`NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, IntrWillReturn, IntrNoCallback, IntrNoFree],`。
- **L3931 EN**: Continues the surrounding expression or declaration: `"", [SDNPMemOperand]`.
  **L3931 CN**: 继续构造周围的表达式或声明：`"", [SDNPMemOperand]`。
- **L3932 EN**: Executes a standalone statement or declaration: `>;`.
  **L3932 CN**: 执行一条独立语句或声明：`>;`。
- **L3933 EN**: Blank line separating nearby declarations or logic blocks.
  **L3933 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3934 EN**: Declares TableGen def `int_amdgcn_cluster_load_async_to_lds_b8`.
  **L3934 CN**: 声明 TableGen def `int_amdgcn_cluster_load_async_to_lds_b8`。
- **L3935 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b8">, AMDGPUAsyncClusterLoadLDS;`.
  **L3935 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b8">, AMDGPUAsyncClusterLoadLDS;`。
- **L3936 EN**: Declares TableGen def `int_amdgcn_cluster_load_async_to_lds_b32`.
  **L3936 CN**: 声明 TableGen def `int_amdgcn_cluster_load_async_to_lds_b32`。

### Lines 3937-3968

````tablegen
  ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b32">, AMDGPUAsyncClusterLoadLDS;
def int_amdgcn_cluster_load_async_to_lds_b64    :
  ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b64">, AMDGPUAsyncClusterLoadLDS;
def int_amdgcn_cluster_load_async_to_lds_b128   :
  ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b128">, AMDGPUAsyncClusterLoadLDS;

def int_amdgcn_global_load_async_to_lds_b8      :
  ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b8">, AMDGPUAsyncGlobalLoadToLDS;
def int_amdgcn_global_load_async_to_lds_b32     :
  ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b32">, AMDGPUAsyncGlobalLoadToLDS;
def int_amdgcn_global_load_async_to_lds_b64      :
  ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b64">, AMDGPUAsyncGlobalLoadToLDS;
def int_amdgcn_global_load_async_to_lds_b128    :
  ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b128">, AMDGPUAsyncGlobalLoadToLDS;

def int_amdgcn_global_store_async_from_lds_b8   :
  ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b8">, AMDGPUAsyncGlobalStoreFromLDS;
def int_amdgcn_global_store_async_from_lds_b32  :
  ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b32">, AMDGPUAsyncGlobalStoreFromLDS;
def int_amdgcn_global_store_async_from_lds_b64  :
  ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b64">, AMDGPUAsyncGlobalStoreFromLDS;
def int_amdgcn_global_store_async_from_lds_b128 :
  ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b128">, AMDGPUAsyncGlobalStoreFromLDS;

// WMMA intrinsics.
class AMDGPUWmmaIntrinsicModsABClamp<LLVMType AB, LLVMType CD> :
  Intrinsic<
    [CD], // %D
    [
      llvm_i1_ty,       // %A_mod: 0 -- none, 1 -- neg
      AB,               // %A
      llvm_i1_ty,       // %B_mod: 0 -- none, 1 -- neg
````
- **L3937 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b32">, AMDGPUAsyncClusterLoadLDS;`.
  **L3937 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b32">, AMDGPUAsyncClusterLoadLDS;`。
- **L3938 EN**: Declares TableGen def `int_amdgcn_cluster_load_async_to_lds_b64`.
  **L3938 CN**: 声明 TableGen def `int_amdgcn_cluster_load_async_to_lds_b64`。
- **L3939 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b64">, AMDGPUAsyncClusterLoadLDS;`.
  **L3939 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b64">, AMDGPUAsyncClusterLoadLDS;`。
- **L3940 EN**: Declares TableGen def `int_amdgcn_cluster_load_async_to_lds_b128`.
  **L3940 CN**: 声明 TableGen def `int_amdgcn_cluster_load_async_to_lds_b128`。
- **L3941 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b128">, AMDGPUAsyncClusterLoadLDS;`.
  **L3941 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_cluster_load_async_to_lds_b128">, AMDGPUAsyncClusterLoadLDS;`。
- **L3942 EN**: Blank line separating nearby declarations or logic blocks.
  **L3942 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3943 EN**: Declares TableGen def `int_amdgcn_global_load_async_to_lds_b8`.
  **L3943 CN**: 声明 TableGen def `int_amdgcn_global_load_async_to_lds_b8`。
- **L3944 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b8">, AMDGPUAsyncGlobalLoadToLDS;`.
  **L3944 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b8">, AMDGPUAsyncGlobalLoadToLDS;`。
- **L3945 EN**: Declares TableGen def `int_amdgcn_global_load_async_to_lds_b32`.
  **L3945 CN**: 声明 TableGen def `int_amdgcn_global_load_async_to_lds_b32`。
- **L3946 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b32">, AMDGPUAsyncGlobalLoadToLDS;`.
  **L3946 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b32">, AMDGPUAsyncGlobalLoadToLDS;`。
- **L3947 EN**: Declares TableGen def `int_amdgcn_global_load_async_to_lds_b64`.
  **L3947 CN**: 声明 TableGen def `int_amdgcn_global_load_async_to_lds_b64`。
- **L3948 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b64">, AMDGPUAsyncGlobalLoadToLDS;`.
  **L3948 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b64">, AMDGPUAsyncGlobalLoadToLDS;`。
- **L3949 EN**: Declares TableGen def `int_amdgcn_global_load_async_to_lds_b128`.
  **L3949 CN**: 声明 TableGen def `int_amdgcn_global_load_async_to_lds_b128`。
- **L3950 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b128">, AMDGPUAsyncGlobalLoadToLDS;`.
  **L3950 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_global_load_async_to_lds_b128">, AMDGPUAsyncGlobalLoadToLDS;`。
- **L3951 EN**: Blank line separating nearby declarations or logic blocks.
  **L3951 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3952 EN**: Declares TableGen def `int_amdgcn_global_store_async_from_lds_b8`.
  **L3952 CN**: 声明 TableGen def `int_amdgcn_global_store_async_from_lds_b8`。
- **L3953 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b8">, AMDGPUAsyncGlobalStoreFromLDS;`.
  **L3953 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b8">, AMDGPUAsyncGlobalStoreFromLDS;`。
- **L3954 EN**: Declares TableGen def `int_amdgcn_global_store_async_from_lds_b32`.
  **L3954 CN**: 声明 TableGen def `int_amdgcn_global_store_async_from_lds_b32`。
- **L3955 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b32">, AMDGPUAsyncGlobalStoreFromLDS;`.
  **L3955 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b32">, AMDGPUAsyncGlobalStoreFromLDS;`。
- **L3956 EN**: Declares TableGen def `int_amdgcn_global_store_async_from_lds_b64`.
  **L3956 CN**: 声明 TableGen def `int_amdgcn_global_store_async_from_lds_b64`。
- **L3957 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b64">, AMDGPUAsyncGlobalStoreFromLDS;`.
  **L3957 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b64">, AMDGPUAsyncGlobalStoreFromLDS;`。
- **L3958 EN**: Declares TableGen def `int_amdgcn_global_store_async_from_lds_b128`.
  **L3958 CN**: 声明 TableGen def `int_amdgcn_global_store_async_from_lds_b128`。
- **L3959 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b128">, AMDGPUAsyncGlobalStoreFromLDS;`.
  **L3959 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_amdgcn_global_store_async_from_lds_b128">, AMDGPUAsyncGlobalStoreFromLDS;`。
- **L3960 EN**: Blank line separating nearby declarations or logic blocks.
  **L3960 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3961 EN**: Comment explains nearby logic, invariants, or intent: `WMMA intrinsics.`.
  **L3961 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA intrinsics.`。
- **L3962 EN**: Declares class `AMDGPUWmmaIntrinsicModsABClamp<LLVMType`.
  **L3962 CN**: 声明 class `AMDGPUWmmaIntrinsicModsABClamp<LLVMType`。
- **L3963 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L3963 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L3964 EN**: Continues the surrounding expression or declaration: `[CD], // %D`.
  **L3964 CN**: 继续构造周围的表达式或声明：`[CD], // %D`。
- **L3965 EN**: Continues the surrounding expression or declaration: `[`.
  **L3965 CN**: 继续构造周围的表达式或声明：`[`。
- **L3966 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %A_mod: 0 -- none, 1 -- neg`.
  **L3966 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %A_mod: 0 -- none, 1 -- neg`。
- **L3967 EN**: Continues the surrounding expression or declaration: `AB,               // %A`.
  **L3967 CN**: 继续构造周围的表达式或声明：`AB,               // %A`。
- **L3968 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %B_mod: 0 -- none, 1 -- neg`.
  **L3968 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %B_mod: 0 -- none, 1 -- neg`。

### Lines 3969-4000

````tablegen
      LLVMMatchType<1>, // %B
      LLVMMatchType<0>,               // %C
      llvm_i1_ty,       // matrix_a_reuse
      llvm_i1_ty,       // matrix_b_reuse
      llvm_i1_ty,       // %clamp
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>,
     IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]
>;

class AMDGPUWmmaIntrinsicModsC<LLVMType AB, LLVMType CD> :
  Intrinsic<
    [CD], // %D
    [
      AB,               // %A
      LLVMMatchType<1>, // %B
      llvm_i16_ty,      // %C_mod: 0 - none, 1 - neg, 2 - abs, 3 - neg(abs)
      LLVMMatchType<0>,               // %C
      llvm_i1_ty,       // matrix_a_reuse
      llvm_i1_ty,       // matrix_b_reuse
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>,
     IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]
>;

class AMDGPUWmmaIntrinsicF4ModsC<LLVMType A, LLVMType B, LLVMType CD> :
  Intrinsic<
    [CD], // %D
    [
      A,                // %A
      B,                // %B
      llvm_i16_ty,      // %C_mod: 0 - none, 1 - neg, 2 - abs, 3 - neg(abs)
````
- **L3969 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<1>, // %B`.
  **L3969 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<1>, // %B`。
- **L3970 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,               // %C`.
  **L3970 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,               // %C`。
- **L3971 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_a_reuse`.
  **L3971 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_a_reuse`。
- **L3972 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_b_reuse`.
  **L3972 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_b_reuse`。
- **L3973 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %clamp`.
  **L3973 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %clamp`。
- **L3974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3974 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>,`.
  **L3975 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>,`。
- **L3976 EN**: Continues the surrounding expression or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]`.
  **L3976 CN**: 继续构造周围的表达式或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]`。
- **L3977 EN**: Executes a standalone statement or declaration: `>;`.
  **L3977 CN**: 执行一条独立语句或声明：`>;`。
- **L3978 EN**: Blank line separating nearby declarations or logic blocks.
  **L3978 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3979 EN**: Declares class `AMDGPUWmmaIntrinsicModsC<LLVMType`.
  **L3979 CN**: 声明 class `AMDGPUWmmaIntrinsicModsC<LLVMType`。
- **L3980 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L3980 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L3981 EN**: Continues the surrounding expression or declaration: `[CD], // %D`.
  **L3981 CN**: 继续构造周围的表达式或声明：`[CD], // %D`。
- **L3982 EN**: Continues the surrounding expression or declaration: `[`.
  **L3982 CN**: 继续构造周围的表达式或声明：`[`。
- **L3983 EN**: Continues the surrounding expression or declaration: `AB,               // %A`.
  **L3983 CN**: 继续构造周围的表达式或声明：`AB,               // %A`。
- **L3984 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<1>, // %B`.
  **L3984 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<1>, // %B`。
- **L3985 EN**: Continues logic associated with callable symbol `neg`.
  **L3985 CN**: 继续与可调用符号 `neg` 相关的逻辑。
- **L3986 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,               // %C`.
  **L3986 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,               // %C`。
- **L3987 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_a_reuse`.
  **L3987 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_a_reuse`。
- **L3988 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_b_reuse`.
  **L3988 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_b_reuse`。
- **L3989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L3989 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L3990 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>,`.
  **L3990 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>,`。
- **L3991 EN**: Continues the surrounding expression or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]`.
  **L3991 CN**: 继续构造周围的表达式或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]`。
- **L3992 EN**: Executes a standalone statement or declaration: `>;`.
  **L3992 CN**: 执行一条独立语句或声明：`>;`。
- **L3993 EN**: Blank line separating nearby declarations or logic blocks.
  **L3993 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3994 EN**: Declares class `AMDGPUWmmaIntrinsicF4ModsC<LLVMType`.
  **L3994 CN**: 声明 class `AMDGPUWmmaIntrinsicF4ModsC<LLVMType`。
- **L3995 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L3995 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L3996 EN**: Continues the surrounding expression or declaration: `[CD], // %D`.
  **L3996 CN**: 继续构造周围的表达式或声明：`[CD], // %D`。
- **L3997 EN**: Continues the surrounding expression or declaration: `[`.
  **L3997 CN**: 继续构造周围的表达式或声明：`[`。
- **L3998 EN**: Continues the surrounding expression or declaration: `A,                // %A`.
  **L3998 CN**: 继续构造周围的表达式或声明：`A,                // %A`。
- **L3999 EN**: Continues the surrounding expression or declaration: `B,                // %B`.
  **L3999 CN**: 继续构造周围的表达式或声明：`B,                // %B`。
- **L4000 EN**: Continues logic associated with callable symbol `neg`.
  **L4000 CN**: 继续与可调用符号 `neg` 相关的逻辑。

### Lines 4001-4032

````tablegen
      LLVMMatchType<0>,               // %C
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>,
     IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison
     ]
>;

class AMDGPUWmmaIntrinsicModsAll<LLVMType AB, LLVMType CD> :
  Intrinsic<
    [CD], // %D
    [
      llvm_i1_ty,       // %A_mod: 0 -- none, 1 -- neg
      AB,               // %A
      llvm_i1_ty,       // %B_mod: 0 -- none, 1 -- neg
      LLVMMatchType<1>, // %B
      llvm_i16_ty,      // %C_mod: 0 -- none, 1 -- neg, 2 -- abs, 3 -- neg(abs)
      LLVMMatchType<0>,               // %C
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

// D and C are of different types.
class AMDGPUWmmaIntrinsicModsCDiff<LLVMType DstTy, LLVMType AB, LLVMType C> :
  Intrinsic<
    [DstTy],     // %D
    [
      AB,                // %A
      LLVMMatchType<1>,  // %B
      llvm_i16_ty,       // %C_mod: 0 -- none, 1 -- neg, 2 -- abs, 3 -- neg(abs)
      C,                 // %C
      llvm_i1_ty,        // matrix_a_reuse
      llvm_i1_ty,        // matrix_b_reuse
````
- **L4001 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,               // %C`.
  **L4001 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,               // %C`。
- **L4002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L4002 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L4003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>,`.
  **L4003 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>,`。
- **L4004 EN**: Continues the surrounding expression or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison`.
  **L4004 CN**: 继续构造周围的表达式或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison`。
- **L4005 EN**: Continues the surrounding expression or declaration: `]`.
  **L4005 CN**: 继续构造周围的表达式或声明：`]`。
- **L4006 EN**: Executes a standalone statement or declaration: `>;`.
  **L4006 CN**: 执行一条独立语句或声明：`>;`。
- **L4007 EN**: Blank line separating nearby declarations or logic blocks.
  **L4007 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4008 EN**: Declares class `AMDGPUWmmaIntrinsicModsAll<LLVMType`.
  **L4008 CN**: 声明 class `AMDGPUWmmaIntrinsicModsAll<LLVMType`。
- **L4009 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4009 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4010 EN**: Continues the surrounding expression or declaration: `[CD], // %D`.
  **L4010 CN**: 继续构造周围的表达式或声明：`[CD], // %D`。
- **L4011 EN**: Continues the surrounding expression or declaration: `[`.
  **L4011 CN**: 继续构造周围的表达式或声明：`[`。
- **L4012 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %A_mod: 0 -- none, 1 -- neg`.
  **L4012 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %A_mod: 0 -- none, 1 -- neg`。
- **L4013 EN**: Continues the surrounding expression or declaration: `AB,               // %A`.
  **L4013 CN**: 继续构造周围的表达式或声明：`AB,               // %A`。
- **L4014 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %B_mod: 0 -- none, 1 -- neg`.
  **L4014 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %B_mod: 0 -- none, 1 -- neg`。
- **L4015 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<1>, // %B`.
  **L4015 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<1>, // %B`。
- **L4016 EN**: Continues logic associated with callable symbol `neg`.
  **L4016 CN**: 继续与可调用符号 `neg` 相关的逻辑。
- **L4017 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>,               // %C`.
  **L4017 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>,               // %C`。
- **L4018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L4018 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L4019 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L4019 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L4020 EN**: Executes a standalone statement or declaration: `>;`.
  **L4020 CN**: 执行一条独立语句或声明：`>;`。
- **L4021 EN**: Blank line separating nearby declarations or logic blocks.
  **L4021 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4022 EN**: Comment explains nearby logic, invariants, or intent: `D and C are of different types.`.
  **L4022 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`D and C are of different types.`。
- **L4023 EN**: Declares class `AMDGPUWmmaIntrinsicModsCDiff<LLVMType`.
  **L4023 CN**: 声明 class `AMDGPUWmmaIntrinsicModsCDiff<LLVMType`。
- **L4024 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4024 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4025 EN**: Continues the surrounding expression or declaration: `[DstTy],     // %D`.
  **L4025 CN**: 继续构造周围的表达式或声明：`[DstTy],     // %D`。
- **L4026 EN**: Continues the surrounding expression or declaration: `[`.
  **L4026 CN**: 继续构造周围的表达式或声明：`[`。
- **L4027 EN**: Continues the surrounding expression or declaration: `AB,                // %A`.
  **L4027 CN**: 继续构造周围的表达式或声明：`AB,                // %A`。
- **L4028 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<1>,  // %B`.
  **L4028 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<1>,  // %B`。
- **L4029 EN**: Continues logic associated with callable symbol `neg`.
  **L4029 CN**: 继续与可调用符号 `neg` 相关的逻辑。
- **L4030 EN**: Continues the surrounding expression or declaration: `C,                 // %C`.
  **L4030 CN**: 继续构造周围的表达式或声明：`C,                 // %C`。
- **L4031 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,        // matrix_a_reuse`.
  **L4031 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,        // matrix_a_reuse`。
- **L4032 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,        // matrix_b_reuse`.
  **L4032 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,        // matrix_b_reuse`。

### Lines 4033-4064

````tablegen
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>,
     IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

class AMDGPUWmmaIntrinsicModsC_MatrixFMT :
  Intrinsic<
    [llvm_anyfloat_ty], // %D
    [
      llvm_i32_ty,      // matrix_a_fmt
      llvm_anyint_ty,   // %A
      llvm_i32_ty,      // matrix_b_fmt
      llvm_anyint_ty,   // %B
      llvm_i16_ty,      // %C_mod: 0 - none, 1 - neg, 2 - abs, 3 - neg(abs)
      LLVMMatchType<0>, // %C
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree]
>;

class AMDGPUWmmaScaleIntrinsicModsC<LLVMType scale_ty> :
  Intrinsic<
    [llvm_anyfloat_ty], // %D
    [
      llvm_i32_ty,      // matrix_a_fmt
      llvm_anyint_ty,   // %A
      llvm_i32_ty,      // matrix_b_fmt
      llvm_anyint_ty,   // %B
      llvm_i16_ty,      // %C_mod: 0 - none, 1 - neg, 2 - abs, 3 - neg(abs)
      LLVMMatchType<0>, // %C
      llvm_i32_ty,      // matrix_a_scale
      llvm_i32_ty,      // matrix_a_scale_fmt
      scale_ty,         // matrix a scale exponential
````
- **L4033 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L4033 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L4034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>,`.
  **L4034 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>,`。
- **L4035 EN**: Continues the surrounding expression or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L4035 CN**: 继续构造周围的表达式或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L4036 EN**: Executes a standalone statement or declaration: `>;`.
  **L4036 CN**: 执行一条独立语句或声明：`>;`。
- **L4037 EN**: Blank line separating nearby declarations or logic blocks.
  **L4037 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4038 EN**: Declares class `AMDGPUWmmaIntrinsicModsC_MatrixFMT`.
  **L4038 CN**: 声明 class `AMDGPUWmmaIntrinsicModsC_MatrixFMT`。
- **L4039 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4039 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4040 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], // %D`.
  **L4040 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], // %D`。
- **L4041 EN**: Continues the surrounding expression or declaration: `[`.
  **L4041 CN**: 继续构造周围的表达式或声明：`[`。
- **L4042 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_a_fmt`.
  **L4042 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_a_fmt`。
- **L4043 EN**: Continues the surrounding expression or declaration: `llvm_anyint_ty,   // %A`.
  **L4043 CN**: 继续构造周围的表达式或声明：`llvm_anyint_ty,   // %A`。
- **L4044 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_b_fmt`.
  **L4044 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_b_fmt`。
- **L4045 EN**: Continues the surrounding expression or declaration: `llvm_anyint_ty,   // %B`.
  **L4045 CN**: 继续构造周围的表达式或声明：`llvm_anyint_ty,   // %B`。
- **L4046 EN**: Continues logic associated with callable symbol `neg`.
  **L4046 CN**: 继续与可调用符号 `neg` 相关的逻辑。
- **L4047 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L4047 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L4048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L4048 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L4049 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree]`.
  **L4049 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, IntrWillReturn, IntrNoCallback, IntrNoFree]`。
- **L4050 EN**: Executes a standalone statement or declaration: `>;`.
  **L4050 CN**: 执行一条独立语句或声明：`>;`。
- **L4051 EN**: Blank line separating nearby declarations or logic blocks.
  **L4051 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4052 EN**: Declares class `AMDGPUWmmaScaleIntrinsicModsC<LLVMType`.
  **L4052 CN**: 声明 class `AMDGPUWmmaScaleIntrinsicModsC<LLVMType`。
- **L4053 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4053 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4054 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], // %D`.
  **L4054 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], // %D`。
- **L4055 EN**: Continues the surrounding expression or declaration: `[`.
  **L4055 CN**: 继续构造周围的表达式或声明：`[`。
- **L4056 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_a_fmt`.
  **L4056 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_a_fmt`。
- **L4057 EN**: Continues the surrounding expression or declaration: `llvm_anyint_ty,   // %A`.
  **L4057 CN**: 继续构造周围的表达式或声明：`llvm_anyint_ty,   // %A`。
- **L4058 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_b_fmt`.
  **L4058 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_b_fmt`。
- **L4059 EN**: Continues the surrounding expression or declaration: `llvm_anyint_ty,   // %B`.
  **L4059 CN**: 继续构造周围的表达式或声明：`llvm_anyint_ty,   // %B`。
- **L4060 EN**: Continues logic associated with callable symbol `neg`.
  **L4060 CN**: 继续与可调用符号 `neg` 相关的逻辑。
- **L4061 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L4061 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L4062 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_a_scale`.
  **L4062 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_a_scale`。
- **L4063 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_a_scale_fmt`.
  **L4063 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_a_scale_fmt`。
- **L4064 EN**: Continues the surrounding expression or declaration: `scale_ty,         // matrix a scale exponential`.
  **L4064 CN**: 继续构造周围的表达式或声明：`scale_ty,         // matrix a scale exponential`。

### Lines 4065-4096

````tablegen
      llvm_i32_ty,      // matrix_b_scale
      llvm_i32_ty,      // matrix_b_scale_fmt
      scale_ty,         // matrix b scale exponential
      llvm_i1_ty,       // matrix_a_reuse
      llvm_i1_ty,       // matrix_b_reuse
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>,
     ImmArg<ArgIndex<7>>, ImmArg<ArgIndex<9>>, ImmArg<ArgIndex<10>>, ImmArg<ArgIndex<12>>, ImmArg<ArgIndex<13>>,
     IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]
>;

class AMDGPUWmmaScaleF4IntrinsicModsC<LLVMType scale_ty> :
  Intrinsic<
    [llvm_anyfloat_ty], // %D
    [
      llvm_anyint_ty,   // %A
      llvm_anyint_ty,   // %B
      llvm_i16_ty,      // %C_mod: 0 - none, 1 - neg, 2 - abs, 3 - neg(abs)
      LLVMMatchType<0>, // %C
      llvm_i32_ty,      // matrix_a_scale
      llvm_i32_ty,      // matrix_a_scale_fmt
      scale_ty,         // matrix a scale exponential
      llvm_i32_ty,      // matrix_b_scale
      llvm_i32_ty,      // matrix_b_scale_fmt
      scale_ty,         // matrix b scale exponential
      llvm_i1_ty,       // matrix_a_reuse
      llvm_i1_ty,       // matrix_b_reuse
    ],
    [IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<7>>,
     ImmArg<ArgIndex<8>>, ImmArg<ArgIndex<10>>, ImmArg<ArgIndex<11>>,
     IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]
>;
````
- **L4065 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_b_scale`.
  **L4065 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_b_scale`。
- **L4066 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_b_scale_fmt`.
  **L4066 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_b_scale_fmt`。
- **L4067 EN**: Continues the surrounding expression or declaration: `scale_ty,         // matrix b scale exponential`.
  **L4067 CN**: 继续构造周围的表达式或声明：`scale_ty,         // matrix b scale exponential`。
- **L4068 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_a_reuse`.
  **L4068 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_a_reuse`。
- **L4069 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_b_reuse`.
  **L4069 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_b_reuse`。
- **L4070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L4070 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L4071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>,`.
  **L4071 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<6>>,`。
- **L4072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<7>>, ImmArg<ArgIndex<9>>, ImmArg<ArgIndex<10>>, ImmArg<ArgIndex<12>>, ImmArg<ArgIndex<13>>,`.
  **L4072 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<7>>, ImmArg<ArgIndex<9>>, ImmArg<ArgIndex<10>>, ImmArg<ArgIndex<12>>, ImmArg<ArgIndex<13>>,`。
- **L4073 EN**: Continues the surrounding expression or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]`.
  **L4073 CN**: 继续构造周围的表达式或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]`。
- **L4074 EN**: Executes a standalone statement or declaration: `>;`.
  **L4074 CN**: 执行一条独立语句或声明：`>;`。
- **L4075 EN**: Blank line separating nearby declarations or logic blocks.
  **L4075 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4076 EN**: Declares class `AMDGPUWmmaScaleF4IntrinsicModsC<LLVMType`.
  **L4076 CN**: 声明 class `AMDGPUWmmaScaleF4IntrinsicModsC<LLVMType`。
- **L4077 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4077 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4078 EN**: Continues the surrounding expression or declaration: `[llvm_anyfloat_ty], // %D`.
  **L4078 CN**: 继续构造周围的表达式或声明：`[llvm_anyfloat_ty], // %D`。
- **L4079 EN**: Continues the surrounding expression or declaration: `[`.
  **L4079 CN**: 继续构造周围的表达式或声明：`[`。
- **L4080 EN**: Continues the surrounding expression or declaration: `llvm_anyint_ty,   // %A`.
  **L4080 CN**: 继续构造周围的表达式或声明：`llvm_anyint_ty,   // %A`。
- **L4081 EN**: Continues the surrounding expression or declaration: `llvm_anyint_ty,   // %B`.
  **L4081 CN**: 继续构造周围的表达式或声明：`llvm_anyint_ty,   // %B`。
- **L4082 EN**: Continues logic associated with callable symbol `neg`.
  **L4082 CN**: 继续与可调用符号 `neg` 相关的逻辑。
- **L4083 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L4083 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L4084 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_a_scale`.
  **L4084 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_a_scale`。
- **L4085 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_a_scale_fmt`.
  **L4085 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_a_scale_fmt`。
- **L4086 EN**: Continues the surrounding expression or declaration: `scale_ty,         // matrix a scale exponential`.
  **L4086 CN**: 继续构造周围的表达式或声明：`scale_ty,         // matrix a scale exponential`。
- **L4087 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_b_scale`.
  **L4087 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_b_scale`。
- **L4088 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // matrix_b_scale_fmt`.
  **L4088 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // matrix_b_scale_fmt`。
- **L4089 EN**: Continues the surrounding expression or declaration: `scale_ty,         // matrix b scale exponential`.
  **L4089 CN**: 继续构造周围的表达式或声明：`scale_ty,         // matrix b scale exponential`。
- **L4090 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_a_reuse`.
  **L4090 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_a_reuse`。
- **L4091 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_b_reuse`.
  **L4091 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_b_reuse`。
- **L4092 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L4092 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L4093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<7>>,`.
  **L4093 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>, ImmArg<ArgIndex<7>>,`。
- **L4094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<8>>, ImmArg<ArgIndex<10>>, ImmArg<ArgIndex<11>>,`.
  **L4094 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<8>>, ImmArg<ArgIndex<10>>, ImmArg<ArgIndex<11>>,`。
- **L4095 EN**: Continues the surrounding expression or declaration: `IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]`.
  **L4095 CN**: 继续构造周围的表达式或声明：`IntrWillReturn, IntrNoCallback, IntrNoFree, IntrNoCreateUndefOrPoison]`。
- **L4096 EN**: Executes a standalone statement or declaration: `>;`.
  **L4096 CN**: 执行一条独立语句或声明：`>;`。

### Lines 4097-4128

````tablegen

defset list<Intrinsic> AMDGPUWMMAIntrinsicsGFX1250 = {
def int_amdgcn_wmma_f32_16x16x4_f32       : AMDGPUWmmaIntrinsicModsC<llvm_anyfloat_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x32_bf16     : AMDGPUWmmaIntrinsicModsC<llvm_anyfloat_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x32_f16      : AMDGPUWmmaIntrinsicModsC<llvm_anyfloat_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f16_16x16x32_f16      : AMDGPUWmmaIntrinsicModsC<llvm_anyfloat_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_bf16_16x16x32_bf16    : AMDGPUWmmaIntrinsicModsC<llvm_anyfloat_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_bf16f32_16x16x32_bf16 : AMDGPUWmmaIntrinsicModsCDiff<llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x64_fp8_fp8  : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x64_fp8_bf8  : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x64_bf8_fp8  : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x64_bf8_bf8  : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f16_16x16x64_fp8_fp8  : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f16_16x16x64_fp8_bf8  : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f16_16x16x64_bf8_fp8  : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f16_16x16x64_bf8_bf8  : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f16_16x16x128_fp8_fp8 : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f16_16x16x128_fp8_bf8 : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f16_16x16x128_bf8_fp8 : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f16_16x16x128_bf8_bf8 : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x128_fp8_fp8 : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x128_fp8_bf8 : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x128_bf8_fp8 : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_f32_16x16x128_bf8_bf8 : AMDGPUWmmaIntrinsicModsC<llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_i32_16x16x64_iu8      : AMDGPUWmmaIntrinsicModsABClamp<llvm_anyint_ty, llvm_anyint_ty>;
def int_amdgcn_wmma_f32_16x16x128_f8f6f4  : AMDGPUWmmaIntrinsicModsC_MatrixFMT;
def int_amdgcn_wmma_scale_f32_16x16x128_f8f6f4   : AMDGPUWmmaScaleIntrinsicModsC<llvm_i32_ty>;
def int_amdgcn_wmma_scale16_f32_16x16x128_f8f6f4 : AMDGPUWmmaScaleIntrinsicModsC<llvm_i64_ty>;
def int_amdgcn_wmma_f32_32x16x128_f4       : AMDGPUWmmaIntrinsicF4ModsC<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty>;
def int_amdgcn_wmma_scale_f32_32x16x128_f4 : AMDGPUWmmaScaleF4IntrinsicModsC<llvm_i32_ty>;
def int_amdgcn_wmma_scale16_f32_32x16x128_f4 : AMDGPUWmmaScaleF4IntrinsicModsC<llvm_i64_ty>;
}
````
- **L4097 EN**: Blank line separating nearby declarations or logic blocks.
  **L4097 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4098 EN**: Continues the surrounding expression or declaration: `defset list<Intrinsic> AMDGPUWMMAIntrinsicsGFX1250 = {`.
  **L4098 CN**: 继续构造周围的表达式或声明：`defset list<Intrinsic> AMDGPUWMMAIntrinsicsGFX1250 = {`。
- **L4099 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x4_f32`.
  **L4099 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x4_f32`。
- **L4100 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x32_bf16`.
  **L4100 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x32_bf16`。
- **L4101 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x32_f16`.
  **L4101 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x32_f16`。
- **L4102 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x32_f16`.
  **L4102 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x32_f16`。
- **L4103 EN**: Declares TableGen def `int_amdgcn_wmma_bf16_16x16x32_bf16`.
  **L4103 CN**: 声明 TableGen def `int_amdgcn_wmma_bf16_16x16x32_bf16`。
- **L4104 EN**: Declares TableGen def `int_amdgcn_wmma_bf16f32_16x16x32_bf16`.
  **L4104 CN**: 声明 TableGen def `int_amdgcn_wmma_bf16f32_16x16x32_bf16`。
- **L4105 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x64_fp8_fp8`.
  **L4105 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x64_fp8_fp8`。
- **L4106 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x64_fp8_bf8`.
  **L4106 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x64_fp8_bf8`。
- **L4107 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x64_bf8_fp8`.
  **L4107 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x64_bf8_fp8`。
- **L4108 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x64_bf8_bf8`.
  **L4108 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x64_bf8_bf8`。
- **L4109 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x64_fp8_fp8`.
  **L4109 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x64_fp8_fp8`。
- **L4110 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x64_fp8_bf8`.
  **L4110 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x64_fp8_bf8`。
- **L4111 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x64_bf8_fp8`.
  **L4111 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x64_bf8_fp8`。
- **L4112 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x64_bf8_bf8`.
  **L4112 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x64_bf8_bf8`。
- **L4113 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x128_fp8_fp8`.
  **L4113 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x128_fp8_fp8`。
- **L4114 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x128_fp8_bf8`.
  **L4114 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x128_fp8_bf8`。
- **L4115 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x128_bf8_fp8`.
  **L4115 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x128_bf8_fp8`。
- **L4116 EN**: Declares TableGen def `int_amdgcn_wmma_f16_16x16x128_bf8_bf8`.
  **L4116 CN**: 声明 TableGen def `int_amdgcn_wmma_f16_16x16x128_bf8_bf8`。
- **L4117 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x128_fp8_fp8`.
  **L4117 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x128_fp8_fp8`。
- **L4118 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x128_fp8_bf8`.
  **L4118 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x128_fp8_bf8`。
- **L4119 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x128_bf8_fp8`.
  **L4119 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x128_bf8_fp8`。
- **L4120 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x128_bf8_bf8`.
  **L4120 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x128_bf8_bf8`。
- **L4121 EN**: Declares TableGen def `int_amdgcn_wmma_i32_16x16x64_iu8`.
  **L4121 CN**: 声明 TableGen def `int_amdgcn_wmma_i32_16x16x64_iu8`。
- **L4122 EN**: Declares TableGen def `int_amdgcn_wmma_f32_16x16x128_f8f6f4`.
  **L4122 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_16x16x128_f8f6f4`。
- **L4123 EN**: Declares TableGen def `int_amdgcn_wmma_scale_f32_16x16x128_f8f6f4`.
  **L4123 CN**: 声明 TableGen def `int_amdgcn_wmma_scale_f32_16x16x128_f8f6f4`。
- **L4124 EN**: Declares TableGen def `int_amdgcn_wmma_scale16_f32_16x16x128_f8f6f4`.
  **L4124 CN**: 声明 TableGen def `int_amdgcn_wmma_scale16_f32_16x16x128_f8f6f4`。
- **L4125 EN**: Declares TableGen def `int_amdgcn_wmma_f32_32x16x128_f4`.
  **L4125 CN**: 声明 TableGen def `int_amdgcn_wmma_f32_32x16x128_f4`。
- **L4126 EN**: Declares TableGen def `int_amdgcn_wmma_scale_f32_32x16x128_f4`.
  **L4126 CN**: 声明 TableGen def `int_amdgcn_wmma_scale_f32_32x16x128_f4`。
- **L4127 EN**: Declares TableGen def `int_amdgcn_wmma_scale16_f32_32x16x128_f4`.
  **L4127 CN**: 声明 TableGen def `int_amdgcn_wmma_scale16_f32_32x16x128_f4`。
- **L4128 EN**: Closes the current lexical scope or compound statement.
  **L4128 CN**: 结束当前词法作用域或复合语句块。

### Lines 4129-4160

````tablegen

class AMDGPUSWmmacIntrinsicABIdx<LLVMType A, LLVMType B, LLVMType CD, LLVMType Index> :
  Intrinsic<
    [CD], // %D
    [
      llvm_i1_ty,       // %A_mod:  0 - none, 1 - neg
      A,                // %A
      llvm_i1_ty,       // %B_mod:  0 - none, 1 - neg
      B,                // %B
      LLVMMatchType<0>, // %C
      Index,            // %Sparsity index for A
      llvm_i1_ty,       // matrix_a_reuse
      llvm_i1_ty,       // matrix_b_reuse
    ],
    [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison,
     ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>]
>;

class AMDGPUSWmmacIntrinsicABIdxClamp<LLVMType A, LLVMType B, LLVMType CD, LLVMType Index> :
  Intrinsic<
    [CD], // %D
    [
      llvm_i1_ty,       // %A_mod:  0 - none, 1 - neg
      A,                // %A
      llvm_i1_ty,       // %B_mod:  0 - none, 1 - neg
      B,                // %B
      LLVMMatchType<0>, // %C
      Index,            // %Sparsity index for A
      llvm_i1_ty,       // matrix_a_reuse
      llvm_i1_ty,       // matrix_b_reuse
      llvm_i1_ty,       // %clamp
    ],
````
- **L4129 EN**: Blank line separating nearby declarations or logic blocks.
  **L4129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4130 EN**: Declares class `AMDGPUSWmmacIntrinsicABIdx<LLVMType`.
  **L4130 CN**: 声明 class `AMDGPUSWmmacIntrinsicABIdx<LLVMType`。
- **L4131 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4131 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4132 EN**: Continues the surrounding expression or declaration: `[CD], // %D`.
  **L4132 CN**: 继续构造周围的表达式或声明：`[CD], // %D`。
- **L4133 EN**: Continues the surrounding expression or declaration: `[`.
  **L4133 CN**: 继续构造周围的表达式或声明：`[`。
- **L4134 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %A_mod:  0 - none, 1 - neg`.
  **L4134 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %A_mod:  0 - none, 1 - neg`。
- **L4135 EN**: Continues the surrounding expression or declaration: `A,                // %A`.
  **L4135 CN**: 继续构造周围的表达式或声明：`A,                // %A`。
- **L4136 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %B_mod:  0 - none, 1 - neg`.
  **L4136 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %B_mod:  0 - none, 1 - neg`。
- **L4137 EN**: Continues the surrounding expression or declaration: `B,                // %B`.
  **L4137 CN**: 继续构造周围的表达式或声明：`B,                // %B`。
- **L4138 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L4138 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L4139 EN**: Continues the surrounding expression or declaration: `Index,            // %Sparsity index for A`.
  **L4139 CN**: 继续构造周围的表达式或声明：`Index,            // %Sparsity index for A`。
- **L4140 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_a_reuse`.
  **L4140 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_a_reuse`。
- **L4141 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_b_reuse`.
  **L4141 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_b_reuse`。
- **L4142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L4142 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L4143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison,`.
  **L4143 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison,`。
- **L4144 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>]`.
  **L4144 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>]`。
- **L4145 EN**: Executes a standalone statement or declaration: `>;`.
  **L4145 CN**: 执行一条独立语句或声明：`>;`。
- **L4146 EN**: Blank line separating nearby declarations or logic blocks.
  **L4146 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4147 EN**: Declares class `AMDGPUSWmmacIntrinsicABIdxClamp<LLVMType`.
  **L4147 CN**: 声明 class `AMDGPUSWmmacIntrinsicABIdxClamp<LLVMType`。
- **L4148 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4148 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4149 EN**: Continues the surrounding expression or declaration: `[CD], // %D`.
  **L4149 CN**: 继续构造周围的表达式或声明：`[CD], // %D`。
- **L4150 EN**: Continues the surrounding expression or declaration: `[`.
  **L4150 CN**: 继续构造周围的表达式或声明：`[`。
- **L4151 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %A_mod:  0 - none, 1 - neg`.
  **L4151 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %A_mod:  0 - none, 1 - neg`。
- **L4152 EN**: Continues the surrounding expression or declaration: `A,                // %A`.
  **L4152 CN**: 继续构造周围的表达式或声明：`A,                // %A`。
- **L4153 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %B_mod:  0 - none, 1 - neg`.
  **L4153 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %B_mod:  0 - none, 1 - neg`。
- **L4154 EN**: Continues the surrounding expression or declaration: `B,                // %B`.
  **L4154 CN**: 继续构造周围的表达式或声明：`B,                // %B`。
- **L4155 EN**: Continues the surrounding expression or declaration: `LLVMMatchType<0>, // %C`.
  **L4155 CN**: 继续构造周围的表达式或声明：`LLVMMatchType<0>, // %C`。
- **L4156 EN**: Continues the surrounding expression or declaration: `Index,            // %Sparsity index for A`.
  **L4156 CN**: 继续构造周围的表达式或声明：`Index,            // %Sparsity index for A`。
- **L4157 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_a_reuse`.
  **L4157 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_a_reuse`。
- **L4158 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // matrix_b_reuse`.
  **L4158 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // matrix_b_reuse`。
- **L4159 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,       // %clamp`.
  **L4159 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,       // %clamp`。
- **L4160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L4160 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。

### Lines 4161-4192

````tablegen
    [IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison,
     ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>, ImmArg<ArgIndex<8>>]
>;

defset list<Intrinsic> AMDGPUSWMMACIntrinsicsGFX1250 = {
def int_amdgcn_swmmac_f32_16x16x64_f16      : AMDGPUSWmmacIntrinsicABIdx<llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x64_bf16     : AMDGPUSWmmacIntrinsicABIdx<llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f16_16x16x64_f16      : AMDGPUSWmmacIntrinsicABIdx<llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_bf16_16x16x64_bf16    : AMDGPUSWmmacIntrinsicABIdx<llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_bf16f32_16x16x64_bf16 : AMDGPUSWmmacIntrinsicABIdx<llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x128_fp8_fp8 : AMDGPUSWmmacIntrinsicIdxReuse<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x128_fp8_bf8 : AMDGPUSWmmacIntrinsicIdxReuse<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x128_bf8_fp8 : AMDGPUSWmmacIntrinsicIdxReuse<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f32_16x16x128_bf8_bf8 : AMDGPUSWmmacIntrinsicIdxReuse<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f16_16x16x128_fp8_fp8 : AMDGPUSWmmacIntrinsicIdxReuse<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f16_16x16x128_fp8_bf8 : AMDGPUSWmmacIntrinsicIdxReuse<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f16_16x16x128_bf8_fp8 : AMDGPUSWmmacIntrinsicIdxReuse<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_f16_16x16x128_bf8_bf8 : AMDGPUSWmmacIntrinsicIdxReuse<llvm_anyint_ty, llvm_anyint_ty, llvm_anyfloat_ty, llvm_anyint_ty>;
def int_amdgcn_swmmac_i32_16x16x128_iu8     : AMDGPUSWmmacIntrinsicABIdxClamp<llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty, llvm_anyint_ty>;
}

class AMDGPUTensorLoadStore:
  Intrinsic<
    [],
    [llvm_v4i32_ty, // D# group 0
     llvm_v8i32_ty, // D# group 1
     llvm_v4i32_ty, // D# group 2: group 2 and 3 should be zero-initialized for D# up to 2D.
     llvm_v4i32_ty, // D# group 3:
     llvm_v8i32_ty, // D# group 4: reserved for future targets, use <8 x i32> zeroinitializer for now.
                    //   This argument will be silently ignored.
     llvm_i32_ty],  // cachepolicy:
                    //   bits [0-2] = th
````
- **L4161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison,`.
  **L4161 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrConvergent, IntrWillReturn, IntrNoCreateUndefOrPoison,`。
- **L4162 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>, ImmArg<ArgIndex<8>>]`.
  **L4162 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<6>>, ImmArg<ArgIndex<7>>, ImmArg<ArgIndex<8>>]`。
- **L4163 EN**: Executes a standalone statement or declaration: `>;`.
  **L4163 CN**: 执行一条独立语句或声明：`>;`。
- **L4164 EN**: Blank line separating nearby declarations or logic blocks.
  **L4164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4165 EN**: Continues the surrounding expression or declaration: `defset list<Intrinsic> AMDGPUSWMMACIntrinsicsGFX1250 = {`.
  **L4165 CN**: 继续构造周围的表达式或声明：`defset list<Intrinsic> AMDGPUSWMMACIntrinsicsGFX1250 = {`。
- **L4166 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x64_f16`.
  **L4166 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x64_f16`。
- **L4167 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x64_bf16`.
  **L4167 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x64_bf16`。
- **L4168 EN**: Declares TableGen def `int_amdgcn_swmmac_f16_16x16x64_f16`.
  **L4168 CN**: 声明 TableGen def `int_amdgcn_swmmac_f16_16x16x64_f16`。
- **L4169 EN**: Declares TableGen def `int_amdgcn_swmmac_bf16_16x16x64_bf16`.
  **L4169 CN**: 声明 TableGen def `int_amdgcn_swmmac_bf16_16x16x64_bf16`。
- **L4170 EN**: Declares TableGen def `int_amdgcn_swmmac_bf16f32_16x16x64_bf16`.
  **L4170 CN**: 声明 TableGen def `int_amdgcn_swmmac_bf16f32_16x16x64_bf16`。
- **L4171 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x128_fp8_fp8`.
  **L4171 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x128_fp8_fp8`。
- **L4172 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x128_fp8_bf8`.
  **L4172 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x128_fp8_bf8`。
- **L4173 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x128_bf8_fp8`.
  **L4173 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x128_bf8_fp8`。
- **L4174 EN**: Declares TableGen def `int_amdgcn_swmmac_f32_16x16x128_bf8_bf8`.
  **L4174 CN**: 声明 TableGen def `int_amdgcn_swmmac_f32_16x16x128_bf8_bf8`。
- **L4175 EN**: Declares TableGen def `int_amdgcn_swmmac_f16_16x16x128_fp8_fp8`.
  **L4175 CN**: 声明 TableGen def `int_amdgcn_swmmac_f16_16x16x128_fp8_fp8`。
- **L4176 EN**: Declares TableGen def `int_amdgcn_swmmac_f16_16x16x128_fp8_bf8`.
  **L4176 CN**: 声明 TableGen def `int_amdgcn_swmmac_f16_16x16x128_fp8_bf8`。
- **L4177 EN**: Declares TableGen def `int_amdgcn_swmmac_f16_16x16x128_bf8_fp8`.
  **L4177 CN**: 声明 TableGen def `int_amdgcn_swmmac_f16_16x16x128_bf8_fp8`。
- **L4178 EN**: Declares TableGen def `int_amdgcn_swmmac_f16_16x16x128_bf8_bf8`.
  **L4178 CN**: 声明 TableGen def `int_amdgcn_swmmac_f16_16x16x128_bf8_bf8`。
- **L4179 EN**: Declares TableGen def `int_amdgcn_swmmac_i32_16x16x128_iu8`.
  **L4179 CN**: 声明 TableGen def `int_amdgcn_swmmac_i32_16x16x128_iu8`。
- **L4180 EN**: Closes the current lexical scope or compound statement.
  **L4180 CN**: 结束当前词法作用域或复合语句块。
- **L4181 EN**: Blank line separating nearby declarations or logic blocks.
  **L4181 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4182 EN**: Declares class `AMDGPUTensorLoadStore`.
  **L4182 CN**: 声明 class `AMDGPUTensorLoadStore`。
- **L4183 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4183 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L4184 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L4185 EN**: Continues the surrounding expression or declaration: `[llvm_v4i32_ty, // D# group 0`.
  **L4185 CN**: 继续构造周围的表达式或声明：`[llvm_v4i32_ty, // D# group 0`。
- **L4186 EN**: Continues the surrounding expression or declaration: `llvm_v8i32_ty, // D# group 1`.
  **L4186 CN**: 继续构造周围的表达式或声明：`llvm_v8i32_ty, // D# group 1`。
- **L4187 EN**: Continues the surrounding expression or declaration: `llvm_v4i32_ty, // D# group 2: group 2 and 3 should be zero-initialized for D# up to 2D.`.
  **L4187 CN**: 继续构造周围的表达式或声明：`llvm_v4i32_ty, // D# group 2: group 2 and 3 should be zero-initialized for D# up to 2D.`。
- **L4188 EN**: Continues the surrounding expression or declaration: `llvm_v4i32_ty, // D# group 3:`.
  **L4188 CN**: 继续构造周围的表达式或声明：`llvm_v4i32_ty, // D# group 3:`。
- **L4189 EN**: Continues the surrounding expression or declaration: `llvm_v8i32_ty, // D# group 4: reserved for future targets, use <8 x i32> zeroinitializer for now.`.
  **L4189 CN**: 继续构造周围的表达式或声明：`llvm_v8i32_ty, // D# group 4: reserved for future targets, use <8 x i32> zeroinitializer for now.`。
- **L4190 EN**: Comment explains nearby logic, invariants, or intent: `This argument will be silently ignored.`.
  **L4190 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This argument will be silently ignored.`。
- **L4191 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],  // cachepolicy:`.
  **L4191 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],  // cachepolicy:`。
- **L4192 EN**: Comment explains nearby logic, invariants, or intent: `bits [0-2] = th`.
  **L4192 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [0-2] = th`。

### Lines 4193-4224

````tablegen
                    //   bits [3-4] = scope
    [IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<5>>, IntrWillReturn, IntrConvergent, IntrNoCallback, IntrNoFree],
    "", [SDNPMemOperand]
  >;

def int_amdgcn_tensor_load_to_lds    : ClangBuiltin<"__builtin_amdgcn_tensor_load_to_lds">, AMDGPUTensorLoadStore;
def int_amdgcn_tensor_store_from_lds : ClangBuiltin<"__builtin_amdgcn_tensor_store_from_lds">, AMDGPUTensorLoadStore;

class AMDGPUClusterLoad<LLVMType ptr_ty>:
  Intrinsic<
    [llvm_any_ty],
    [ptr_ty,
     llvm_i32_ty,      // gfx12+ cachepolicy:
                       //   bits [0-2] = th
                       //   bits [3-4] = scope
     llvm_i32_ty],     // workgroup broadcast mask (in M0)
    [IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, IntrWillReturn, IntrConvergent,
     NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>, IntrNoCallback, IntrNoFree],
    "", [SDNPMemOperand]
  >;

def int_amdgcn_cluster_load_b32         : AMDGPUClusterLoad<global_ptr_ty>;
def int_amdgcn_cluster_load_b64         : AMDGPUClusterLoad<global_ptr_ty>;
def int_amdgcn_cluster_load_b128        : AMDGPUClusterLoad<global_ptr_ty>;

class AMDGPULoadMonitor<LLVMType ptr_ty>:
  Intrinsic<
    [llvm_any_ty],
    [ptr_ty,
     llvm_i32_ty,            // C ABI Atomic Ordering ID
     llvm_metadata_ty],      // syncscope
    [IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>,
````
- **L4193 EN**: Comment explains nearby logic, invariants, or intent: `bits [3-4] = scope`.
  **L4193 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [3-4] = scope`。
- **L4194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<5>>, IntrWillReturn, IntrConvergent, IntrNoCallback, IntrNoFree],`.
  **L4194 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<5>>, IntrWillReturn, IntrConvergent, IntrNoCallback, IntrNoFree],`。
- **L4195 EN**: Continues the surrounding expression or declaration: `"", [SDNPMemOperand]`.
  **L4195 CN**: 继续构造周围的表达式或声明：`"", [SDNPMemOperand]`。
- **L4196 EN**: Executes a standalone statement or declaration: `>;`.
  **L4196 CN**: 执行一条独立语句或声明：`>;`。
- **L4197 EN**: Blank line separating nearby declarations or logic blocks.
  **L4197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4198 EN**: Declares TableGen def `int_amdgcn_tensor_load_to_lds`.
  **L4198 CN**: 声明 TableGen def `int_amdgcn_tensor_load_to_lds`。
- **L4199 EN**: Declares TableGen def `int_amdgcn_tensor_store_from_lds`.
  **L4199 CN**: 声明 TableGen def `int_amdgcn_tensor_store_from_lds`。
- **L4200 EN**: Blank line separating nearby declarations or logic blocks.
  **L4200 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4201 EN**: Declares class `AMDGPUClusterLoad<LLVMType`.
  **L4201 CN**: 声明 class `AMDGPUClusterLoad<LLVMType`。
- **L4202 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4202 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty],`.
  **L4203 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty],`。
- **L4204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ptr_ty,`.
  **L4204 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ptr_ty,`。
- **L4205 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // gfx12+ cachepolicy:`.
  **L4205 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // gfx12+ cachepolicy:`。
- **L4206 EN**: Comment explains nearby logic, invariants, or intent: `bits [0-2] = th`.
  **L4206 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [0-2] = th`。
- **L4207 EN**: Comment explains nearby logic, invariants, or intent: `bits [3-4] = scope`.
  **L4207 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bits [3-4] = scope`。
- **L4208 EN**: Continues logic associated with callable symbol `mask`.
  **L4208 CN**: 继续与可调用符号 `mask` 相关的逻辑。
- **L4209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, IntrWillReturn, IntrConvergent,`.
  **L4209 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, IntrWillReturn, IntrConvergent,`。
- **L4210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>, IntrNoCallback, IntrNoFree],`.
  **L4210 CN**: 继续一个多行参数列表、初始化器或聚合项：`NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>, IntrNoCallback, IntrNoFree],`。
- **L4211 EN**: Continues the surrounding expression or declaration: `"", [SDNPMemOperand]`.
  **L4211 CN**: 继续构造周围的表达式或声明：`"", [SDNPMemOperand]`。
- **L4212 EN**: Executes a standalone statement or declaration: `>;`.
  **L4212 CN**: 执行一条独立语句或声明：`>;`。
- **L4213 EN**: Blank line separating nearby declarations or logic blocks.
  **L4213 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4214 EN**: Declares TableGen def `int_amdgcn_cluster_load_b32`.
  **L4214 CN**: 声明 TableGen def `int_amdgcn_cluster_load_b32`。
- **L4215 EN**: Declares TableGen def `int_amdgcn_cluster_load_b64`.
  **L4215 CN**: 声明 TableGen def `int_amdgcn_cluster_load_b64`。
- **L4216 EN**: Declares TableGen def `int_amdgcn_cluster_load_b128`.
  **L4216 CN**: 声明 TableGen def `int_amdgcn_cluster_load_b128`。
- **L4217 EN**: Blank line separating nearby declarations or logic blocks.
  **L4217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4218 EN**: Declares class `AMDGPULoadMonitor<LLVMType`.
  **L4218 CN**: 声明 class `AMDGPULoadMonitor<LLVMType`。
- **L4219 EN**: Continues the surrounding expression or declaration: `Intrinsic<`.
  **L4219 CN**: 继续构造周围的表达式或声明：`Intrinsic<`。
- **L4220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty],`.
  **L4220 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty],`。
- **L4221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ptr_ty,`.
  **L4221 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ptr_ty,`。
- **L4222 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,            // C ABI Atomic Ordering ID`.
  **L4222 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,            // C ABI Atomic Ordering ID`。
- **L4223 EN**: Continues the surrounding expression or declaration: `llvm_metadata_ty],      // syncscope`.
  **L4223 CN**: 继续构造周围的表达式或声明：`llvm_metadata_ty],      // syncscope`。
- **L4224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>,`.
  **L4224 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrReadMem, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>,`。

### Lines 4225-4250

````tablegen
     IntrWillReturn, IntrConvergent, IntrNoCallback, IntrNoFree],
    "",
    [SDNPMemOperand, SDNPMayLoad]
  >;

def int_amdgcn_flat_load_monitor_b32    : AMDGPULoadMonitor<flat_ptr_ty>;
def int_amdgcn_flat_load_monitor_b64    : AMDGPULoadMonitor<flat_ptr_ty>;
def int_amdgcn_flat_load_monitor_b128   : AMDGPULoadMonitor<flat_ptr_ty>;
def int_amdgcn_global_load_monitor_b32  : AMDGPULoadMonitor<global_ptr_ty>;
def int_amdgcn_global_load_monitor_b64  : AMDGPULoadMonitor<global_ptr_ty>;
def int_amdgcn_global_load_monitor_b128 : AMDGPULoadMonitor<global_ptr_ty>;

/// Emit an addrspacecast without null pointer checking.
/// Should only be inserted by a pass based on analysis of an addrspacecast's src.
def int_amdgcn_addrspacecast_nonnull : DefaultAttrsIntrinsic<
  [llvm_anyptr_ty], [llvm_anyptr_ty],
  [IntrNoMem, IntrSpeculatable]
>;

/// Make it clear to the backend that this value is really dead. For instance,
/// when used as an input to a phi node, it will make it possible for the
/// backend to allocate the dead lanes for operations within the corresponding
/// incoming block.
def int_amdgcn_dead: DefaultAttrsIntrinsic<[llvm_any_ty], [],
    [IntrNoMem]>;
}
````
- **L4225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrWillReturn, IntrConvergent, IntrNoCallback, IntrNoFree],`.
  **L4225 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrWillReturn, IntrConvergent, IntrNoCallback, IntrNoFree],`。
- **L4226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"",`.
  **L4226 CN**: 继续一个多行参数列表、初始化器或聚合项：`"",`。
- **L4227 EN**: Continues the surrounding expression or declaration: `[SDNPMemOperand, SDNPMayLoad]`.
  **L4227 CN**: 继续构造周围的表达式或声明：`[SDNPMemOperand, SDNPMayLoad]`。
- **L4228 EN**: Executes a standalone statement or declaration: `>;`.
  **L4228 CN**: 执行一条独立语句或声明：`>;`。
- **L4229 EN**: Blank line separating nearby declarations or logic blocks.
  **L4229 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4230 EN**: Declares TableGen def `int_amdgcn_flat_load_monitor_b32`.
  **L4230 CN**: 声明 TableGen def `int_amdgcn_flat_load_monitor_b32`。
- **L4231 EN**: Declares TableGen def `int_amdgcn_flat_load_monitor_b64`.
  **L4231 CN**: 声明 TableGen def `int_amdgcn_flat_load_monitor_b64`。
- **L4232 EN**: Declares TableGen def `int_amdgcn_flat_load_monitor_b128`.
  **L4232 CN**: 声明 TableGen def `int_amdgcn_flat_load_monitor_b128`。
- **L4233 EN**: Declares TableGen def `int_amdgcn_global_load_monitor_b32`.
  **L4233 CN**: 声明 TableGen def `int_amdgcn_global_load_monitor_b32`。
- **L4234 EN**: Declares TableGen def `int_amdgcn_global_load_monitor_b64`.
  **L4234 CN**: 声明 TableGen def `int_amdgcn_global_load_monitor_b64`。
- **L4235 EN**: Declares TableGen def `int_amdgcn_global_load_monitor_b128`.
  **L4235 CN**: 声明 TableGen def `int_amdgcn_global_load_monitor_b128`。
- **L4236 EN**: Blank line separating nearby declarations or logic blocks.
  **L4236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4237 EN**: Comment explains nearby logic, invariants, or intent: `Emit an addrspacecast without null pointer checking.`.
  **L4237 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Emit an addrspacecast without null pointer checking.`。
- **L4238 EN**: Comment explains nearby logic, invariants, or intent: `Should only be inserted by a pass based on analysis of an addrspacecast's src.`.
  **L4238 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Should only be inserted by a pass based on analysis of an addrspacecast's src.`。
- **L4239 EN**: Declares TableGen def `int_amdgcn_addrspacecast_nonnull`.
  **L4239 CN**: 声明 TableGen def `int_amdgcn_addrspacecast_nonnull`。
- **L4240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty], [llvm_anyptr_ty],`.
  **L4240 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty], [llvm_anyptr_ty],`。
- **L4241 EN**: Continues the surrounding expression or declaration: `[IntrNoMem, IntrSpeculatable]`.
  **L4241 CN**: 继续构造周围的表达式或声明：`[IntrNoMem, IntrSpeculatable]`。
- **L4242 EN**: Executes a standalone statement or declaration: `>;`.
  **L4242 CN**: 执行一条独立语句或声明：`>;`。
- **L4243 EN**: Blank line separating nearby declarations or logic blocks.
  **L4243 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4244 EN**: Comment explains nearby logic, invariants, or intent: `Make it clear to the backend that this value is really dead. For instance,`.
  **L4244 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Make it clear to the backend that this value is really dead. For instance,`。
- **L4245 EN**: Comment explains nearby logic, invariants, or intent: `when used as an input to a phi node, it will make it possible for the`.
  **L4245 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`when used as an input to a phi node, it will make it possible for the`。
- **L4246 EN**: Comment explains nearby logic, invariants, or intent: `backend to allocate the dead lanes for operations within the corresponding`.
  **L4246 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`backend to allocate the dead lanes for operations within the corresponding`。
- **L4247 EN**: Comment explains nearby logic, invariants, or intent: `incoming block.`.
  **L4247 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`incoming block.`。
- **L4248 EN**: Declares TableGen def `int_amdgcn_dead`.
  **L4248 CN**: 声明 TableGen def `int_amdgcn_dead`。
- **L4249 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L4249 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L4250 EN**: Closes the current lexical scope or compound statement.
  **L4250 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **ORC JIT runtime and remote execution / ORC JIT 运行时与远程执行**
- **Function-level IR management / 函数级 IR 管理**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Metadata representation / 元数据表示**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
