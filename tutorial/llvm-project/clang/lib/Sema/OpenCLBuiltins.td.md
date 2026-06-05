# OpenCLBuiltins.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Sema/OpenCLBuiltins.td`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: This file contains TableGen definitions for OpenCL builtin function.
- **Purpose (CN) / 用途（中文）**: 该文件在 Clang 的语义分析子系统中声明与 OpenCLBuiltins 相关的逻辑。对应英文说明：This file contains TableGen definitions for OpenCL builtin function。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
//==--- OpenCLBuiltins.td - OpenCL builtin declarations -------------------===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains TableGen definitions for OpenCL builtin function
// declarations.  In case of an unresolved function name in OpenCL, Clang will
// check for a function described in this file when -fdeclare-opencl-builtins
// is specified.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
//              Definitions of miscellaneous basic entities.
//===----------------------------------------------------------------------===//
// Versions of OpenCL
class Version<int _Version> {
  int ID = _Version;
}
def CLAll : Version<  0>;
```

- **L1**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L2**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L3**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L4**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L5**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L6**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L7**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L8**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L9**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L10**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L11**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L12**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L13**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L14**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L15**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L16**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L17**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L18**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L19**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L20**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L21**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L22**: Begins the declaration of class `Version`. / 开始声明 class `Version`。
- **L23**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L24**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L25**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 26-50 / 第 26-50 行

```tablegen
def CL10  : Version<100>;
def CL11  : Version<110>;
def CL12  : Version<120>;
def CL20  : Version<200>;

// Address spaces
// Pointer types need to be assigned an address space.
class AddressSpace<string _AS> {
  string Name = _AS;
}
def DefaultAS    : AddressSpace<"clang::LangAS::Default">;
def PrivateAS    : AddressSpace<"clang::LangAS::opencl_private">;
def GlobalAS     : AddressSpace<"clang::LangAS::opencl_global">;
def ConstantAS   : AddressSpace<"clang::LangAS::opencl_constant">;
def LocalAS      : AddressSpace<"clang::LangAS::opencl_local">;
def GenericAS    : AddressSpace<"clang::LangAS::opencl_generic">;

// OpenCL language extension.
class AbstractExtension<string _Ext> {
  // One or more OpenCL extensions, space separated.  Each extension must be
  // a valid extension name for the opencl extension pragma.
  string ExtName = _Ext;
}

// Extension associated to a builtin function.
```

- **L26**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L27**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L28**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L29**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L30**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L31**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L32**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L33**: Begins the declaration of class `AddressSpace`. / 开始声明 class `AddressSpace`。
- **L34**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L35**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L36**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L37**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L38**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L39**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L40**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L41**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L42**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L43**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L44**: Begins the declaration of class `AbstractExtension`. / 开始声明 class `AbstractExtension`。
- **L45**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L46**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L47**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L48**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L49**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L50**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。

### Lines 51-75 / 第 51-75 行

```tablegen
class FunctionExtension<string _Ext> : AbstractExtension<_Ext>;

// Extension associated to a type.  This enables implicit conditionalization of
// builtin function overloads containing a type that depends on an extension.
// During overload resolution, when a builtin function overload contains a type
// with a TypeExtension, those overloads are skipped when the extension is
// disabled.
class TypeExtension<string _Ext> : AbstractExtension<_Ext>;

// Concatenate zero or more space-separated extensions in NewExts to Base and
// return the resulting FunctionExtension in ret.
class concatExtension<FunctionExtension Base, string NewExts> {
  FunctionExtension ret = FunctionExtension<
    !cond(
      // Return Base extension if NewExts is empty,
      !empty(NewExts) : Base.ExtName,

      // otherwise, return NewExts if Base extension is empty,
      !empty(Base.ExtName) : NewExts,

      // otherwise, concatenate NewExts to Base.
      true : Base.ExtName # " " # NewExts
    )
  >;
}
```

- **L51**: Begins the declaration of class `FunctionExtension`. / 开始声明 class `FunctionExtension`。
- **L52**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L53**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L54**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L55**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L56**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L57**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L58**: Begins the declaration of class `TypeExtension`. / 开始声明 class `TypeExtension`。
- **L59**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L60**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L61**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L62**: Begins the declaration of class `concatExtension`. / 开始声明 class `concatExtension`。
- **L63**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L64**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L65**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L66**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L67**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L68**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L69**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L70**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L71**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L72**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L73**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L74**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L75**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 76-100 / 第 76-100 行

```tablegen

// TypeExtension definitions.
def NoTypeExt   : TypeExtension<"">;
def Fp16TypeExt : TypeExtension<"cl_khr_fp16">;
def Fp64TypeExt : TypeExtension<"cl_khr_fp64">;
def Atomic64TypeExt : TypeExtension<"cl_khr_int64_base_atomics cl_khr_int64_extended_atomics">;
def AtomicFp64TypeExt : TypeExtension<"cl_khr_int64_base_atomics cl_khr_int64_extended_atomics cl_khr_fp64">;

// FunctionExtension definitions.
def FuncExtNone                          : FunctionExtension<"">;
def FuncExtKhrSubgroups                  : FunctionExtension<"__opencl_subgroup_builtins">;
def FuncExtKhrSubgroupExtendedTypes      : FunctionExtension<"cl_khr_subgroup_extended_types">;
def FuncExtKhrSubgroupNonUniformVote     : FunctionExtension<"cl_khr_subgroup_non_uniform_vote">;
def FuncExtKhrSubgroupBallot             : FunctionExtension<"cl_khr_subgroup_ballot">;
def FuncExtKhrSubgroupNonUniformArithmetic: FunctionExtension<"cl_khr_subgroup_non_uniform_arithmetic">;
def FuncExtKhrSubgroupShuffle            : FunctionExtension<"cl_khr_subgroup_shuffle">;
def FuncExtKhrSubgroupShuffleRelative    : FunctionExtension<"cl_khr_subgroup_shuffle_relative">;
def FuncExtKhrSubgroupClusteredReduce    : FunctionExtension<"cl_khr_subgroup_clustered_reduce">;
def FuncExtKhrExtendedBitOps             : FunctionExtension<"cl_khr_extended_bit_ops">;
def FuncExtKhrGlobalInt32BaseAtomics     : FunctionExtension<"cl_khr_global_int32_base_atomics">;
def FuncExtKhrGlobalInt32ExtendedAtomics : FunctionExtension<"cl_khr_global_int32_extended_atomics">;
def FuncExtKhrLocalInt32BaseAtomics      : FunctionExtension<"cl_khr_local_int32_base_atomics">;
def FuncExtKhrLocalInt32ExtendedAtomics  : FunctionExtension<"cl_khr_local_int32_extended_atomics">;
def FuncExtKhrInt64BaseAtomics           : FunctionExtension<"cl_khr_int64_base_atomics">;
def FuncExtKhrInt64ExtendedAtomics       : FunctionExtension<"cl_khr_int64_extended_atomics">;
```

- **L76**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L77**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L78**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L79**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L80**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L81**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L82**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L83**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L84**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L85**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L86**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L87**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L88**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L89**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L90**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L91**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L92**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L93**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L94**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L95**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L96**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L97**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L98**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L99**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L100**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 101-125 / 第 101-125 行

```tablegen
def FuncExtKhrMipmapImage                : FunctionExtension<"cl_khr_mipmap_image">;
def FuncExtKhrMipmapImageWrites          : FunctionExtension<"cl_khr_mipmap_image_writes">;
def FuncExtKhrGlMsaaSharing              : FunctionExtension<"cl_khr_gl_msaa_sharing">;

def FuncExtOpenCLCDeviceEnqueue          : FunctionExtension<"__opencl_c_device_enqueue">;
def FuncExtOpenCLCGenericAddressSpace    : FunctionExtension<"__opencl_c_generic_address_space">;
def FuncExtOpenCLCNamedAddressSpaceBuiltins : FunctionExtension<"__opencl_c_named_address_space_builtins">;
def FuncExtOpenCLCPipes                  : FunctionExtension<"__opencl_c_pipes">;
def FuncExtOpenCLCWGCollectiveFunctions  : FunctionExtension<"__opencl_c_work_group_collective_functions">;
def FuncExtOpenCLCReadWriteImages        : FunctionExtension<"__opencl_c_read_write_images">;
def FuncExtFloatAtomicsFp16GlobalASLoadStore  : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp16_global_atomic_load_store">;
def FuncExtFloatAtomicsFp16LocalASLoadStore   : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp16_local_atomic_load_store">;
def FuncExtFloatAtomicsFp16GenericASLoadStore : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp16_global_atomic_load_store __opencl_c_ext_fp16_local_atomic_load_store">;
def FuncExtFloatAtomicsFp16GlobalASAdd        : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp16_global_atomic_add">;
def FuncExtFloatAtomicsFp32GlobalASAdd        : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp32_global_atomic_add">;
def FuncExtFloatAtomicsFp64GlobalASAdd        : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp64_global_atomic_add">;
def FuncExtFloatAtomicsFp16LocalASAdd         : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp16_local_atomic_add">;
def FuncExtFloatAtomicsFp32LocalASAdd         : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp32_local_atomic_add">;
def FuncExtFloatAtomicsFp64LocalASAdd         : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp64_local_atomic_add">;
def FuncExtFloatAtomicsFp16GenericASAdd       : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp16_local_atomic_add __opencl_c_ext_fp16_global_atomic_add">;
def FuncExtFloatAtomicsFp32GenericASAdd       : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp32_local_atomic_add __opencl_c_ext_fp32_global_atomic_add">;
def FuncExtFloatAtomicsFp64GenericASAdd       : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp64_local_atomic_add __opencl_c_ext_fp64_global_atomic_add">;
def FuncExtFloatAtomicsFp16GlobalASMinMax     : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp16_global_atomic_min_max">;
def FuncExtFloatAtomicsFp32GlobalASMinMax     : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp32_global_atomic_min_max">;
def FuncExtFloatAtomicsFp64GlobalASMinMax     : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp64_global_atomic_min_max">;
```

- **L101**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L102**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L103**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L104**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L105**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L106**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L107**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L108**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L109**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L110**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L111**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L112**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L113**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L114**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L115**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L116**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L117**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L118**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L119**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L120**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L121**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L122**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L123**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L124**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L125**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 126-150 / 第 126-150 行

```tablegen
def FuncExtFloatAtomicsFp16LocalASMinMax      : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp16_local_atomic_min_max">;
def FuncExtFloatAtomicsFp32LocalASMinMax      : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp32_local_atomic_min_max">;
def FuncExtFloatAtomicsFp64LocalASMinMax      : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp64_local_atomic_min_max">;
def FuncExtFloatAtomicsFp16GenericASMinMax    : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp16_local_atomic_min_max __opencl_c_ext_fp16_global_atomic_min_max">;
def FuncExtFloatAtomicsFp32GenericASMinMax    : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp32_local_atomic_min_max __opencl_c_ext_fp32_global_atomic_min_max">;
def FuncExtFloatAtomicsFp64GenericASMinMax    : FunctionExtension<"cl_ext_float_atomics __opencl_c_ext_fp64_local_atomic_min_max __opencl_c_ext_fp64_global_atomic_min_max">;

// Not a real extension, but a workaround to add C++ for OpenCL specific builtins.
def FuncExtOpenCLCxx                     : FunctionExtension<"__cplusplus">;

// Arm extensions.
def ArmIntegerDotProductInt8                   : FunctionExtension<"cl_arm_integer_dot_product_int8">;
def ArmIntegerDotProductAccumulateInt8         : FunctionExtension<"cl_arm_integer_dot_product_accumulate_int8">;
def ArmIntegerDotProductAccumulateInt16        : FunctionExtension<"cl_arm_integer_dot_product_accumulate_int16">;
def ArmIntegerDotProductAccumulateSaturateInt8 : FunctionExtension<"cl_arm_integer_dot_product_accumulate_saturate_int8">;

// AMD extensions.
def AMDMediaOps : FunctionExtension<"cl_amd_media_ops">;
def AMDMediaOps2 : FunctionExtension<"cl_amd_media_ops2">;

// Qualified Type.  These map to ASTContext::QualType.
class QualType<string _TypeExpr, bit _IsAbstract=0> {
  // Expression to obtain the QualType inside OCL2Qual.
  // E.g. TypeExpr="Context.IntTy" for the int type.
  string TypeExpr = _TypeExpr;
```

- **L126**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L127**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L128**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L129**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L130**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L131**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L132**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L133**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L134**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L135**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L136**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L137**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L138**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L139**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L140**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L141**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L142**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L143**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L144**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L145**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L146**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L147**: Begins the declaration of class `QualType`. / 开始声明 class `QualType`。
- **L148**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L149**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L150**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。

### Lines 151-175 / 第 151-175 行

```tablegen
  // Some QualTypes in this file represent an abstract type for which there is
  // no corresponding AST QualType, e.g. a GenType or an `image2d_t` type
  // without access qualifiers.
  bit IsAbstract = _IsAbstract;
}

// List of integers.
class IntList<string _Name, list<int> _List> {
  string Name = _Name;
  list<int> List = _List;
}

//===----------------------------------------------------------------------===//
//                      OpenCL C classes for types
//===----------------------------------------------------------------------===//
// OpenCL C basic data types (int, float, image2d_t, ...).
// Its child classes can represent concrete types (e.g. VectorType) or
// abstract types (e.g. GenType).
class Type<string _Name, QualType _QTExpr> {
  // Name of the Type.
  string Name = _Name;
  // QualType associated with this type.
  QualType QTExpr = _QTExpr;
  // Size of the vector (if applicable).
  int VecWidth = 1;
```

- **L151**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L152**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L153**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L154**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L155**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L156**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L157**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L158**: Begins the declaration of class `IntList`. / 开始声明 class `IntList`。
- **L159**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L160**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L161**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L162**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L163**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L164**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L165**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L166**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L167**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L168**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L169**: Begins the declaration of class `Type`. / 开始声明 class `Type`。
- **L170**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L171**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L172**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L173**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L174**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L175**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。

### Lines 176-200 / 第 176-200 行

```tablegen
  // Is a pointer.
  bit IsPointer = 0;
  // "const" qualifier.
  bit IsConst = 0;
  // "volatile" qualifier.
  bit IsVolatile = 0;
  // Access qualifier. Must be one of ("RO", "WO", "RW").
  string AccessQualifier = "";
  // Address space.
  string AddrSpace = DefaultAS.Name;
  // Extension that needs to be enabled to expose a builtin that uses this type.
  TypeExtension Extension = NoTypeExt;
}

// OpenCL vector types (e.g. int2, int3, int16, float8, ...).
class VectorType<Type _Ty, int _VecWidth> : Type<_Ty.Name, _Ty.QTExpr> {
  let VecWidth = _VecWidth;
  let AccessQualifier = "";
  // Inherited fields
  let IsPointer = _Ty.IsPointer;
  let IsConst = _Ty.IsConst;
  let IsVolatile = _Ty.IsVolatile;
  let AddrSpace = _Ty.AddrSpace;
  let Extension = _Ty.Extension;
}
```

- **L176**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L177**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L178**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L179**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L180**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L181**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L182**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L183**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L184**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L185**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L186**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L187**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L188**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L189**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L190**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L191**: Begins the declaration of class `VectorType`. / 开始声明 class `VectorType`。
- **L192**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L193**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L194**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L195**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L196**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L197**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L198**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L199**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L200**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 201-225 / 第 201-225 行

```tablegen

// OpenCL pointer types (e.g. int*, float*, ...).
class PointerType<Type _Ty, AddressSpace _AS = DefaultAS> :
    Type<_Ty.Name, _Ty.QTExpr> {
  let AddrSpace = _AS.Name;
  // Inherited fields
  let VecWidth = _Ty.VecWidth;
  let IsPointer = 1;
  let IsConst = _Ty.IsConst;
  let IsVolatile = _Ty.IsVolatile;
  let AccessQualifier = _Ty.AccessQualifier;
  let Extension = _Ty.Extension;
}

// OpenCL const types (e.g. const int).
class ConstType<Type _Ty> : Type<_Ty.Name, _Ty.QTExpr> {
  let IsConst = 1;
  // Inherited fields
  let VecWidth = _Ty.VecWidth;
  let IsPointer = _Ty.IsPointer;
  let IsVolatile = _Ty.IsVolatile;
  let AccessQualifier = _Ty.AccessQualifier;
  let AddrSpace = _Ty.AddrSpace;
  let Extension = _Ty.Extension;
}
```

- **L201**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L202**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L203**: Begins the declaration of class `PointerType`. / 开始声明 class `PointerType`。
- **L204**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L205**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L206**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L207**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L208**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L209**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L210**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L211**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L212**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L213**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L214**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L215**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L216**: Begins the declaration of class `ConstType`. / 开始声明 class `ConstType`。
- **L217**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L218**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L219**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L220**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L221**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L222**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L223**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L224**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L225**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 226-250 / 第 226-250 行

```tablegen

// OpenCL volatile types (e.g. volatile int).
class VolatileType<Type _Ty> : Type<_Ty.Name, _Ty.QTExpr> {
  let IsVolatile = 1;
  // Inherited fields
  let VecWidth = _Ty.VecWidth;
  let IsPointer = _Ty.IsPointer;
  let IsConst = _Ty.IsConst;
  let AccessQualifier = _Ty.AccessQualifier;
  let AddrSpace = _Ty.AddrSpace;
  let Extension = _Ty.Extension;
}

// OpenCL image types (e.g. image2d).
class ImageType<Type _Ty, string _AccessQualifier> :
    Type<_Ty.Name, QualType<_Ty.QTExpr.TypeExpr # _AccessQualifier # "Ty", 0>> {
  let VecWidth = 0;
  let AccessQualifier = _AccessQualifier;
  // Inherited fields
  let IsPointer = _Ty.IsPointer;
  let IsConst = _Ty.IsConst;
  let IsVolatile = _Ty.IsVolatile;
  let AddrSpace = _Ty.AddrSpace;
  // Add TypeExtensions for writable "image3d_t" and "read_write" image types.
  let Extension = !cond(
```

- **L226**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L227**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L228**: Begins the declaration of class `VolatileType`. / 开始声明 class `VolatileType`。
- **L229**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L230**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L231**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L232**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L233**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L234**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L235**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L236**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L237**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L238**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L239**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L240**: Begins the declaration of class `ImageType`. / 开始声明 class `ImageType`。
- **L241**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L242**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L243**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L244**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L245**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L246**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L247**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L248**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L249**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L250**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 251-275 / 第 251-275 行

```tablegen
      !and(!eq(_Ty.Name, "image3d_t"), !eq(_AccessQualifier, "WO")) : TypeExtension<"cl_khr_3d_image_writes">,
      !and(!eq(_Ty.Name, "image3d_t"), !eq(_AccessQualifier, "RW")) : TypeExtension<"cl_khr_3d_image_writes __opencl_c_read_write_images">,
      !or(!eq(_Ty.Name, "image2d_depth_t"), !eq(_Ty.Name, "image2d_array_depth_t")) : TypeExtension<"__opencl_depth_image_builtins">,
      !eq(_AccessQualifier, "RW") : TypeExtension<"__opencl_c_read_write_images">,
      true : _Ty.Extension);
}

// OpenCL enum type (e.g. memory_scope).
class EnumType<string _Name> :
    Type<_Name, QualType<"getOpenCLEnumType(S, \"" # _Name # "\")", 0>> {
}

// OpenCL typedef type (e.g. cl_mem_fence_flags).
class TypedefType<string _Name> :
    Type<_Name, QualType<"getOpenCLTypedefType(S, \"" # _Name # "\")", 0>> {
}

// List of Types.
class TypeList<list<Type> _Type> {
  list<Type> List = _Type;
}

// A GenericType is an abstract type that defines a set of types as a
// combination of Types and vector sizes.
//
```

- **L251**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L252**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L253**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L254**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L255**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L256**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L257**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L258**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L259**: Begins the declaration of class `EnumType`. / 开始声明 class `EnumType`。
- **L260**: This line completes a function, method, or callable signature and opens its body. / 这一行完成函数、方法或可调用对象的签名，并打开其函数体。
- **L261**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L262**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L263**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L264**: Begins the declaration of class `TypedefType`. / 开始声明 class `TypedefType`。
- **L265**: This line completes a function, method, or callable signature and opens its body. / 这一行完成函数、方法或可调用对象的签名，并打开其函数体。
- **L266**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L267**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L268**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L269**: Begins the declaration of class `TypeList`. / 开始声明 class `TypeList`。
- **L270**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L271**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L272**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L273**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L274**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L275**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。

### Lines 276-300 / 第 276-300 行

```tablegen
// For example, if TypeList = <int, float> and VectorList = <1, 2, 4>, then it
// represents <int, int2, int4, float, float2, float4>.
//
// Some rules apply when using multiple GenericType arguments in a declaration:
//   1. The number of vector sizes must be equal or 1 for all gentypes in a
//      declaration.
//   2. The number of Types must be equal or 1 for all gentypes in a
//      declaration.
//   3. Generic types are combined by iterating over all generic types at once.
//      For example, for the following GenericTypes
//        GenT1 = GenericType<half, [1, 2]> and
//        GenT2 = GenericType<float, int, [1, 2]>
//      A declaration f(GenT1, GenT2) results in the combinations
//        f(half, float), f(half2, float2), f(half, int), f(half2, int2) .
//   4. "sgentype" from the OpenCL specification is supported by specifying
//      a single vector size.
//      For example, for the following GenericTypes
//        GenT = GenericType<half, int, [1, 2]> and
//        SGenT = GenericType<half, int, [1]>
//      A declaration f(GenT, SGenT) results in the combinations
//        f(half, half), f(half2, half), f(int, int), f(int2, int) .
class GenericType<string _Ty, TypeList _TypeList, IntList _VectorList> :
    Type<_Ty, QualType<"null", 1>> {
  // Possible element types of the generic type.
  TypeList TypeList = _TypeList;
```

- **L276**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L277**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L278**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L279**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L280**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L281**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L282**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L283**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L284**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L285**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L286**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L287**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L288**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L289**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L290**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L291**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L292**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L293**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L294**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L295**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L296**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L297**: Begins the declaration of class `GenericType`. / 开始声明 class `GenericType`。
- **L298**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L299**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L300**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。

### Lines 301-325 / 第 301-325 行

```tablegen
  // Possible vector sizes of the types in the TypeList.
  IntList VectorList = _VectorList;
  // The VecWidth field is ignored for GenericTypes. Use VectorList instead.
  let VecWidth = 0;
}

// Builtin function attributes.
def Attr {
  list<bit> None = [0, 0, 0];
  list<bit> Pure = [1, 0, 0];
  list<bit> Const = [0, 1, 0];
  list<bit> Convergent = [0, 0, 1];
}

//===----------------------------------------------------------------------===//
//                      OpenCL C class for builtin functions
//===----------------------------------------------------------------------===//
class Builtin<string _Name, list<Type> _Signature, list<bit> _Attributes = Attr.None> {
  // Name of the builtin function
  string Name = _Name;
  // List of types used by the function. The first one is the return type and
  // the following are the arguments. The list must have at least one element
  // (the return type).
  list<Type> Signature = _Signature;
  // Function attribute __attribute__((pure))
```

- **L301**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L302**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L303**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L304**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L305**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L306**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L307**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L308**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L309**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L310**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L311**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L312**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L313**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L314**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L315**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L316**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L317**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L318**: Begins the declaration of class `Builtin`. / 开始声明 class `Builtin`。
- **L319**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L320**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L321**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L322**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L323**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L324**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L325**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。

### Lines 326-350 / 第 326-350 行

```tablegen
  bit IsPure = _Attributes[0];
  // Function attribute __attribute__((const))
  bit IsConst = _Attributes[1];
  // Function attribute __attribute__((convergent))
  bit IsConv = _Attributes[2];
  // OpenCL extensions to which the function belongs.
  FunctionExtension Extension = FuncExtNone;
  // Version of OpenCL from which the function is available (e.g.: CL10).
  // MinVersion is inclusive.
  Version MinVersion = CL10;
  // Version of OpenCL from which the function is not supported anymore.
  // MaxVersion is exclusive.
  // CLAll makes the function available for all versions.
  Version MaxVersion = CLAll;
}

//===----------------------------------------------------------------------===//
//                 Definitions of OpenCL C types
//===----------------------------------------------------------------------===//

// OpenCL v1.0/1.2/2.0 s6.1.1: Built-in Scalar Data Types.
def Bool      : Type<"bool",      QualType<"Context.BoolTy">>;
def Char      : Type<"char",      QualType<"Context.CharTy">>;
def UChar     : Type<"uchar",     QualType<"Context.UnsignedCharTy">>;
def Short     : Type<"short",     QualType<"Context.ShortTy">>;
```

- **L326**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L327**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L328**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L329**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L330**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L331**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L332**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L333**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L334**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L335**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L336**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L337**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L338**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L339**: Assigns or initializes a value used by the surrounding logic. / 对周边逻辑使用的值进行赋值或初始化。
- **L340**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L341**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L342**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L343**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L344**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L345**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L346**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L347**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L348**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L349**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L350**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 351-375 / 第 351-375 行

```tablegen
def UShort    : Type<"ushort",    QualType<"Context.UnsignedShortTy">>;
def Int       : Type<"int",       QualType<"Context.IntTy">>;
def UInt      : Type<"uint",      QualType<"Context.UnsignedIntTy">>;
def Long      : Type<"long",      QualType<"Context.LongTy">>;
def ULong     : Type<"ulong",     QualType<"Context.UnsignedLongTy">>;
def Float     : Type<"float",     QualType<"Context.FloatTy">>;
let Extension = Fp64TypeExt in {
  def Double    : Type<"double",    QualType<"Context.DoubleTy">>;
}

// The half type for builtins that require the cl_khr_fp16 extension.
let Extension = Fp16TypeExt in {
  def Half      : Type<"half",      QualType<"Context.HalfTy">>;
}

// Without the cl_khr_fp16 extension, the half type can only be used to declare
// a pointer.  Define const and non-const pointer types in all address spaces.
// Use the "__half" alias to allow the TableGen emitter to distinguish the
// (extensionless) pointee type of these pointer-to-half types from the "half"
// type defined above that already carries the cl_khr_fp16 extension.
foreach AS = [PrivateAS, GlobalAS, ConstantAS, LocalAS, GenericAS] in {
  def "HalfPtr" # AS      : PointerType<Type<"__half", QualType<"Context.HalfTy">>, AS>;
  def "HalfPtrConst" # AS : PointerType<ConstType<Type<"__half", QualType<"Context.HalfTy">>>, AS>;
}

```

- **L351**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L352**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L353**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L354**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L355**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L356**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L357**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L358**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L359**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L360**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L361**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L362**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L363**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L364**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L365**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L366**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L367**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L368**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L369**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L370**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L371**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L372**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L373**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L374**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L375**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 376-400 / 第 376-400 行

```tablegen
def Size      : Type<"size_t",    QualType<"Context.getSizeType()">>;
def PtrDiff   : Type<"ptrdiff_t", QualType<"Context.getPointerDiffType()">>;
def IntPtr    : Type<"intptr_t",  QualType<"Context.getIntPtrType()">>;
def UIntPtr   : Type<"uintptr_t", QualType<"Context.getUIntPtrType()">>;
def Void      : Type<"void",      QualType<"Context.VoidTy">>;

// OpenCL v1.0/1.2/2.0 s6.1.2: Built-in Vector Data Types.
// Built-in vector data types are created by TableGen's OpenCLBuiltinEmitter.

// OpenCL v1.0/1.2/2.0 s6.1.3: Other Built-in Data Types.
// The image definitions are "abstract".  They should not be used without
// specifying an access qualifier (RO/WO/RW).
def Image1d               : Type<"image1d_t", QualType<"Context.OCLImage1d", 1>>;
def Image2d               : Type<"image2d_t", QualType<"Context.OCLImage2d", 1>>;
def Image3d               : Type<"image3d_t", QualType<"Context.OCLImage3d", 1>>;
def Image1dArray          : Type<"image1d_array_t", QualType<"Context.OCLImage1dArray", 1>>;
def Image1dBuffer         : Type<"image1d_buffer_t", QualType<"Context.OCLImage1dBuffer", 1>>;
def Image2dArray          : Type<"image2d_array_t", QualType<"Context.OCLImage2dArray", 1>>;
def Image2dDepth          : Type<"image2d_depth_t", QualType<"Context.OCLImage2dDepth", 1>>;
def Image2dArrayDepth     : Type<"image2d_array_depth_t", QualType<"Context.OCLImage2dArrayDepth", 1>>;
def Image2dMsaa           : Type<"image2d_msaa_t", QualType<"Context.OCLImage2dMSAA", 1>>;
def Image2dArrayMsaa      : Type<"image2d_array_msaa_t", QualType<"Context.OCLImage2dArrayMSAA", 1>>;
def Image2dMsaaDepth      : Type<"image2d_msaa_depth_t", QualType<"Context.OCLImage2dMSAADepth", 1>>;
def Image2dArrayMsaaDepth : Type<"image2d_array_msaa_depth_t", QualType<"Context.OCLImage2dArrayMSAADepth", 1>>;

```

- **L376**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L377**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L378**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L379**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L380**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L381**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L382**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L383**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L384**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L385**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L386**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L387**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L388**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L389**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L390**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L391**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L392**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L393**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L394**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L395**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L396**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L397**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L398**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L399**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L400**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 401-425 / 第 401-425 行

```tablegen
def Sampler               : Type<"sampler_t", QualType<"Context.OCLSamplerTy">>;
def ClkEvent              : Type<"clk_event_t", QualType<"Context.OCLClkEventTy">>;
def Event                 : Type<"event_t", QualType<"Context.OCLEventTy">>;
def Queue                 : Type<"queue_t", QualType<"Context.OCLQueueTy">>;
def ReserveId             : Type<"reserve_id_t", QualType<"Context.OCLReserveIDTy">>;
def MemFenceFlags         : TypedefType<"cl_mem_fence_flags">;
def ClkProfilingInfo      : TypedefType<"clk_profiling_info">;
def NDRange               : TypedefType<"ndrange_t">;

// OpenCL v2.0 s6.13.11: Atomic integer and floating-point types.
def AtomicInt             : Type<"atomic_int", QualType<"Context.getAtomicType(Context.IntTy)">>;
def AtomicUInt            : Type<"atomic_uint", QualType<"Context.getAtomicType(Context.UnsignedIntTy)">>;
let Extension = Atomic64TypeExt in {
  def AtomicLong            : Type<"atomic_long", QualType<"Context.getAtomicType(Context.LongTy)">>;
  def AtomicULong           : Type<"atomic_ulong", QualType<"Context.getAtomicType(Context.UnsignedLongTy)">>;
}
def AtomicFloat           : Type<"atomic_float", QualType<"Context.getAtomicType(Context.FloatTy)">>;
let Extension = AtomicFp64TypeExt in {
  def AtomicDouble          : Type<"atomic_double", QualType<"Context.getAtomicType(Context.DoubleTy)">>;
}
def AtomicHalf            : Type<"atomic_half", QualType<"Context.getAtomicType(Context.HalfTy)">>;
def AtomicIntPtr          : Type<"atomic_intptr_t", QualType<"Context.getAtomicType(Context.getIntPtrType())">>;
def AtomicUIntPtr         : Type<"atomic_uintptr_t", QualType<"Context.getAtomicType(Context.getUIntPtrType())">>;
def AtomicSize            : Type<"atomic_size_t", QualType<"Context.getAtomicType(Context.getSizeType())">>;
def AtomicPtrDiff         : Type<"atomic_ptrdiff_t", QualType<"Context.getAtomicType(Context.getPointerDiffType())">>;
```

- **L401**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L402**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L403**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L404**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L405**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L406**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L407**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L408**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L409**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L410**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L411**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L412**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L413**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L414**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L415**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L416**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L417**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L418**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L419**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L420**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L421**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L422**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L423**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L424**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L425**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。

### Lines 426-450 / 第 426-450 行

```tablegen

def AtomicFlag            : TypedefType<"atomic_flag">;
def MemoryOrder           : EnumType<"memory_order">;
def MemoryScope           : EnumType<"memory_scope">;

//===----------------------------------------------------------------------===//
//                 Definitions of OpenCL gentype variants
//===----------------------------------------------------------------------===//
// The OpenCL specification often uses "gentype" in builtin function
// declarations to indicate that a builtin function is available with various
// argument and return types.  The types represented by "gentype" vary between
// different parts of the specification.  The following definitions capture
// the different type lists for gentypes in different parts of the
// specification.

// Vector width lists.
def VecAndScalar: IntList<"VecAndScalar", [1, 2, 3, 4, 8, 16]>;
def VecNoScalar : IntList<"VecNoScalar", [2, 3, 4, 8, 16]>;
def Vec1        : IntList<"Vec1", [1]>;
def Vec1234     : IntList<"Vec1234", [1, 2, 3, 4]>;

// Type lists.
def TLAll           : TypeList<[Char,  UChar, Short,  UShort, Int,  UInt, Long,  ULong, Float, Double, Half]>;
def TLFloat         : TypeList<[Float, Double, Half]>;
def TLSignedInts    : TypeList<[Char, Short, Int, Long]>;
```

- **L426**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L427**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L428**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L429**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L430**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L431**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L432**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L433**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L434**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L435**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L436**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L437**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L438**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L439**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L440**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L441**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L442**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L443**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L444**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L445**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L446**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L447**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L448**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L449**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L450**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 451-475 / 第 451-475 行

```tablegen
def TLUnsignedInts  : TypeList<[UChar, UShort, UInt, ULong]>;

def TLIntLongFloats : TypeList<[Int, UInt, Long, ULong, Float, Double, Half]>;

// All unsigned integer types twice, to facilitate unsigned return types for e.g.
// uchar abs(char) and
// uchar abs(uchar).
def TLAllUIntsTwice : TypeList<[UChar, UChar, UShort, UShort, UInt, UInt, ULong, ULong]>;

def TLAllInts       : TypeList<[Char, UChar, Short, UShort, Int, UInt, Long, ULong]>;

// GenType definitions for multiple base types (e.g. all floating point types,
// or all integer types).
// All types
def AGenType1              : GenericType<"AGenType1", TLAll, Vec1>;
def AGenTypeN              : GenericType<"AGenTypeN", TLAll, VecAndScalar>;
def AGenTypeNNoScalar      : GenericType<"AGenTypeNNoScalar", TLAll, VecNoScalar>;
// All integer
def AIGenType1             : GenericType<"AIGenType1", TLAllInts, Vec1>;
def AIGenTypeN             : GenericType<"AIGenTypeN", TLAllInts, VecAndScalar>;
def AIGenTypeNNoScalar     : GenericType<"AIGenTypeNNoScalar", TLAllInts, VecNoScalar>;
// All integer to unsigned
def AI2UGenTypeN           : GenericType<"AI2UGenTypeN", TLAllUIntsTwice, VecAndScalar>;
// Signed integer
def SGenTypeN              : GenericType<"SGenTypeN", TLSignedInts, VecAndScalar>;
```

- **L451**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L452**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L453**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L454**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L455**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L456**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L457**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L458**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L459**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L460**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L461**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L462**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L463**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L464**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L465**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L466**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L467**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L468**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L469**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L470**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L471**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L472**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L473**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L474**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L475**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 476-500 / 第 476-500 行

```tablegen
// Unsigned integer
def UGenTypeN              : GenericType<"UGenTypeN", TLUnsignedInts, VecAndScalar>;
// Float
def FGenTypeN              : GenericType<"FGenTypeN", TLFloat, VecAndScalar>;
// (u)int, (u)long, and all floats
def IntLongFloatGenType1   : GenericType<"IntLongFloatGenType1", TLIntLongFloats, Vec1>;
// (u)char and (u)short
def CharShortGenType1      : GenericType<"CharShortGenType1",
                                 TypeList<[Char, UChar, Short, UShort]>, Vec1>;

// GenType definitions for every single base type (e.g. fp32 only).
// Names are like: GenTypeFloatVecAndScalar.
foreach Type = [Char, UChar, Short, UShort,
                Int, UInt, Long, ULong,
                Float, Double, Half] in {
  foreach VecSizes = [VecAndScalar, VecNoScalar] in {
    def "GenType" # Type # VecSizes :
              GenericType<"GenType" # Type # VecSizes,
                          TypeList<[Type]>, VecSizes>;
  }
}

// GenType definitions for vec1234.
foreach Type = [Float, Double, Half] in {
  def "GenType" # Type # Vec1234 :
```

- **L476**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L477**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L478**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L479**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L480**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L481**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L482**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L483**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L484**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L485**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L486**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L487**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L488**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L489**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L490**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L491**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L492**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L493**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L494**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L495**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L496**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L497**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L498**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L499**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L500**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 501-525 / 第 501-525 行

```tablegen
              GenericType<"GenType" # Type # Vec1234,
                          TypeList<[Type]>, Vec1234>;
}


//===----------------------------------------------------------------------===//
//                 Definitions of OpenCL builtin functions
//===----------------------------------------------------------------------===//
//--------------------------------------------------------------------
// OpenCL v1.1/1.2/2.0 s6.2.3 - Explicit conversions.
// OpenCL v2.0 Extensions s5.1.1 and s6.1.1 - Conversions.

// Generate the convert_* builtins functions.
foreach RType = [Float, Double, Half, Char, UChar, Short,
                 UShort, Int, UInt, Long, ULong] in {
  foreach IType = [Float, Double, Half, Char, UChar, Short,
                   UShort, Int, UInt, Long, ULong] in {
    // Conversions to integer type have a sat and non-sat variant.
    foreach sat = !cond(!eq(RType.Name, "float") : [""],
                        !eq(RType.Name, "double") : [""],
                        !eq(RType.Name, "half") : [""],
                        1 : ["", "_sat"]) in {
      foreach rnd = ["", "_rte", "_rtn", "_rtp", "_rtz"] in {
        def : Builtin<"convert_" # RType.Name # sat # rnd, [RType, IType],
                      Attr.Const>;
```

- **L501**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L502**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L503**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L504**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L505**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L506**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L507**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L508**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L509**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L510**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L511**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L512**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L513**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L514**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L515**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L516**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L517**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L518**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L519**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L520**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L521**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L522**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L523**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L524**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L525**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 526-550 / 第 526-550 行

```tablegen
        foreach v = [2, 3, 4, 8, 16] in {
          def : Builtin<"convert_" # RType.Name # v # sat # rnd,
                        [VectorType<RType, v>, VectorType<IType, v>],
                        Attr.Const>;
        }
      }
    }
  }
}

//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.1, v1.2 s6.12.1, v2.0 s6.13.1 - Work-item Functions
// --- Table 7 ---
def : Builtin<"get_work_dim", [UInt], Attr.Const>;
foreach name = ["get_global_size", "get_global_id", "get_local_size",
                "get_local_id", "get_num_groups", "get_group_id",
                "get_global_offset"] in {
  def : Builtin<name, [Size, UInt], Attr.Const>;
}

let MinVersion = CL20 in {
  def : Builtin<"get_enqueued_local_size", [Size, UInt]>;
  foreach name = ["get_global_linear_id", "get_local_linear_id"] in {
    def : Builtin<name, [Size]>;
  }
```

- **L526**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L527**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L528**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L529**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L530**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L531**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L532**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L533**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L534**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L535**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L536**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L537**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L538**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L539**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L540**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L541**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L542**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L543**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L544**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L545**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L546**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L547**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L548**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L549**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L550**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 551-575 / 第 551-575 行

```tablegen
}


//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.2, v1.2 s6.12.2, v2.0 s6.13.2 - Math functions
// OpenCL Extension v2.0 s5.1.2 and s6.1.2 - Math Functions
// --- Table 8 ---
// --- 1 argument ---
foreach name = ["acos", "acosh", "acospi",
                "asin", "asinh", "asinpi",
                "atan", "atanh", "atanpi",
                "cbrt", "ceil",
                "cos", "cosh", "cospi",
                "erfc", "erf",
                "exp", "exp2", "exp10", "expm1",
                "fabs", "floor",
                "log", "log2", "log10", "log1p", "logb",
                "rint", "round", "rsqrt",
                "sin", "sinh", "sinpi",
                "sqrt",
                "tan", "tanh", "tanpi",
                "tgamma", "trunc",
                "lgamma"] in {
    def : Builtin<name, [FGenTypeN, FGenTypeN], Attr.Const>;
}
```

- **L551**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L552**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L553**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L554**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L555**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L556**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L557**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L558**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L559**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L560**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L561**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L562**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L563**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L564**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L565**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L566**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L567**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L568**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L569**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L570**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L571**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L572**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L573**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L574**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L575**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 576-600 / 第 576-600 行

```tablegen
foreach name = ["nan"] in {
  def : Builtin<name, [GenTypeFloatVecAndScalar, GenTypeUIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecAndScalar, GenTypeULongVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecAndScalar, GenTypeUShortVecAndScalar], Attr.Const>;
}

// --- 2 arguments ---
foreach name = ["atan2", "atan2pi", "copysign", "fdim", "fmod", "hypot",
                "maxmag", "minmag", "nextafter", "pow", "powr",
                "remainder"] in {
  def : Builtin<name, [FGenTypeN, FGenTypeN, FGenTypeN], Attr.Const>;
}
foreach name = ["fmax", "fmin"] in {
  def : Builtin<name, [FGenTypeN, FGenTypeN, FGenTypeN], Attr.Const>;
  def : Builtin<name, [GenTypeFloatVecNoScalar, GenTypeFloatVecNoScalar, Float], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecNoScalar, GenTypeDoubleVecNoScalar, Double], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecNoScalar, GenTypeHalfVecNoScalar, Half], Attr.Const>;
}
foreach name = ["ilogb"] in {
  def : Builtin<name, [GenTypeIntVecAndScalar, GenTypeFloatVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeIntVecAndScalar, GenTypeDoubleVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeIntVecAndScalar, GenTypeHalfVecAndScalar], Attr.Const>;
}
foreach name = ["ldexp"] in {
  def : Builtin<name, [GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar, GenTypeIntVecAndScalar], Attr.Const>;
```

- **L576**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L577**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L578**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L579**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L580**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L581**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L582**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L583**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L584**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L585**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L586**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L587**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L588**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L589**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L590**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L591**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L592**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L593**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L594**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L595**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L596**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L597**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L598**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L599**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L600**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 601-625 / 第 601-625 行

```tablegen
  def : Builtin<name, [GenTypeFloatVecNoScalar, GenTypeFloatVecNoScalar, Int], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecAndScalar, GenTypeDoubleVecAndScalar, GenTypeIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecNoScalar, GenTypeDoubleVecNoScalar, Int], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecAndScalar, GenTypeHalfVecAndScalar, GenTypeIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecNoScalar, GenTypeHalfVecNoScalar, Int], Attr.Const>;
}
foreach name = ["pown", "rootn"] in {
  def : Builtin<name, [GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar, GenTypeIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecAndScalar, GenTypeDoubleVecAndScalar, GenTypeIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecAndScalar, GenTypeHalfVecAndScalar, GenTypeIntVecAndScalar], Attr.Const>;
}

// --- 3 arguments ---
foreach name = ["fma", "mad"] in {
  def : Builtin<name, [FGenTypeN, FGenTypeN, FGenTypeN, FGenTypeN], Attr.Const>;
}

// The following math builtins take pointer arguments.  Which overloads are
// available depends on whether the generic address space feature is enabled.
multiclass MathWithPointer<list<AddressSpace> addrspaces> {
  foreach AS = addrspaces in {
    foreach name = ["fract", "modf", "sincos"] in {
      def : Builtin<name, [FGenTypeN, FGenTypeN, PointerType<FGenTypeN, AS>]>;
    }
    foreach name = ["frexp", "lgamma_r"] in {
```

- **L601**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L602**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L603**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L604**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L605**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L606**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L607**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L608**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L609**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L610**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L611**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L612**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L613**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L614**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L615**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L616**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L617**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L618**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L619**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L620**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L621**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L622**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L623**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L624**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L625**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 626-650 / 第 626-650 行

```tablegen
      foreach Type = [GenTypeFloatVecAndScalar, GenTypeDoubleVecAndScalar, GenTypeHalfVecAndScalar] in {
        def : Builtin<name, [Type, Type, PointerType<GenTypeIntVecAndScalar, AS>]>;
      }
    }
    foreach name = ["remquo"] in {
      foreach Type = [GenTypeFloatVecAndScalar, GenTypeDoubleVecAndScalar, GenTypeHalfVecAndScalar] in {
        def : Builtin<name, [Type, Type, Type, PointerType<GenTypeIntVecAndScalar, AS>]>;
      }
    }
  }
}

let Extension = FuncExtOpenCLCNamedAddressSpaceBuiltins in {
  defm : MathWithPointer<[GlobalAS, LocalAS, PrivateAS]>;
}
let Extension = FuncExtOpenCLCGenericAddressSpace in {
  defm : MathWithPointer<[GenericAS]>;
}

// --- Table 9 ---
foreach name = ["half_cos",
                "half_exp", "half_exp2", "half_exp10",
                "half_log", "half_log2", "half_log10",
                "half_recip", "half_rsqrt",
                "half_sin", "half_sqrt", "half_tan",
```

- **L626**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L627**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L628**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L629**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L630**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L631**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L632**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L633**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L634**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L635**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L636**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L637**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L638**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L639**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L640**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L641**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L642**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L643**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L644**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L645**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L646**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L647**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L648**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L649**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L650**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 651-675 / 第 651-675 行

```tablegen
                "native_cos",
                "native_exp", "native_exp2", "native_exp10",
                "native_log", "native_log2", "native_log10",
                "native_recip", "native_rsqrt",
                "native_sin", "native_sqrt", "native_tan"] in {
  def : Builtin<name, [GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar], Attr.Const>;
}
foreach name = ["half_divide", "half_powr",
                "native_divide", "native_powr"] in {
  def : Builtin<name, [GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar], Attr.Const>;
}

//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.3, v1.2 s6.12.3, v2.0 s6.13.3 - Integer Functions
// --- Table 10 ---
// --- 1 argument ---
foreach name = ["abs"] in {
  def : Builtin<name, [AI2UGenTypeN, AIGenTypeN], Attr.Const>;
}
def : Builtin<"clz", [AIGenTypeN, AIGenTypeN], Attr.Const>;
let MinVersion = CL12 in {
  def : Builtin<"popcount", [AIGenTypeN, AIGenTypeN], Attr.Const>;
}
let MinVersion = CL20 in {
  foreach name = ["ctz"] in {
```

- **L651**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L652**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L653**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L654**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L655**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L656**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L657**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L658**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L659**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L660**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L661**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L662**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L663**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L664**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L665**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L666**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L667**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L668**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L669**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L670**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L671**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L672**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L673**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L674**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L675**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 676-700 / 第 676-700 行

```tablegen
    def : Builtin<name, [AIGenTypeN, AIGenTypeN], Attr.Const>;
  }
}

// --- 2 arguments ---
foreach name = ["abs_diff"] in {
  def : Builtin<name, [AI2UGenTypeN, AIGenTypeN, AIGenTypeN], Attr.Const>;
}
foreach name = ["add_sat", "hadd", "rhadd", "mul_hi", "rotate", "sub_sat"] in {
  def : Builtin<name, [AIGenTypeN, AIGenTypeN, AIGenTypeN], Attr.Const>;
}
foreach name = ["max", "min"] in {
  def : Builtin<name, [AIGenTypeN, AIGenTypeN, AIGenTypeN], Attr.Const>;
  def : Builtin<name, [AIGenTypeNNoScalar, AIGenTypeNNoScalar, AIGenType1], Attr.Const>;
}
foreach name = ["upsample"] in {
  def : Builtin<name, [GenTypeShortVecAndScalar, GenTypeCharVecAndScalar, GenTypeUCharVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeUShortVecAndScalar, GenTypeUCharVecAndScalar, GenTypeUCharVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeIntVecAndScalar, GenTypeShortVecAndScalar, GenTypeUShortVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeUIntVecAndScalar, GenTypeUShortVecAndScalar, GenTypeUShortVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeLongVecAndScalar, GenTypeIntVecAndScalar, GenTypeUIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeULongVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar], Attr.Const>;
}

// --- 3 arguments ---
```

- **L676**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L677**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L678**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L679**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L680**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L681**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L682**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L683**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L684**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L685**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L686**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L687**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L688**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L689**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L690**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L691**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L692**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L693**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L694**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L695**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L696**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L697**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L698**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L699**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L700**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。

### Lines 701-725 / 第 701-725 行

```tablegen
foreach name = ["clamp"] in {
  def : Builtin<name, [AIGenTypeN, AIGenTypeN, AIGenTypeN, AIGenTypeN], Attr.Const>;
  def : Builtin<name, [AIGenTypeNNoScalar, AIGenTypeNNoScalar, AIGenType1, AIGenType1], Attr.Const>;
}
foreach name = ["mad_hi", "mad_sat"] in {
  def : Builtin<name, [AIGenTypeN, AIGenTypeN, AIGenTypeN, AIGenTypeN], Attr.Const>;
}

// --- Table 11 ---
foreach name = ["mad24"] in {
  def : Builtin<name, [GenTypeIntVecAndScalar, GenTypeIntVecAndScalar, GenTypeIntVecAndScalar, GenTypeIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar], Attr.Const>;
}
foreach name = ["mul24"] in {
  def : Builtin<name, [GenTypeIntVecAndScalar, GenTypeIntVecAndScalar, GenTypeIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar], Attr.Const>;
}

//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.4, v1.2 s6.12.4, v2.0 s6.13.4 - Common Functions
// OpenCL Extension v2.0 s5.1.3 and s6.1.3 - Common Functions
// --- Table 12 ---
// --- 1 argument ---
foreach name = ["degrees", "radians", "sign"] in {
  def : Builtin<name, [FGenTypeN, FGenTypeN], Attr.Const>;
```

- **L701**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L702**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L703**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L704**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L705**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L706**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L707**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L708**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L709**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L710**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L711**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L712**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L713**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L714**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L715**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L716**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L717**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L718**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L719**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L720**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L721**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L722**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L723**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L724**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L725**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 726-750 / 第 726-750 行

```tablegen
}

// --- 2 arguments ---
foreach name = ["max", "min"] in {
  def : Builtin<name, [FGenTypeN, FGenTypeN, FGenTypeN], Attr.Const>;
  def : Builtin<name, [GenTypeFloatVecNoScalar, GenTypeFloatVecNoScalar, Float], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecNoScalar, GenTypeDoubleVecNoScalar, Double], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecNoScalar, GenTypeHalfVecNoScalar, Half], Attr.Const>;
}
foreach name = ["step"] in {
  def : Builtin<name, [FGenTypeN, FGenTypeN, FGenTypeN], Attr.Const>;
  def : Builtin<name, [GenTypeFloatVecNoScalar, Float, GenTypeFloatVecNoScalar], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecNoScalar, Double, GenTypeDoubleVecNoScalar], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecNoScalar, Half, GenTypeHalfVecNoScalar], Attr.Const>;
}

// --- 3 arguments ---
foreach name = ["clamp"] in {
  def : Builtin<name, [FGenTypeN, FGenTypeN, FGenTypeN, FGenTypeN], Attr.Const>;
  def : Builtin<name, [GenTypeFloatVecNoScalar, GenTypeFloatVecNoScalar, Float, Float], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecNoScalar, GenTypeDoubleVecNoScalar, Double, Double], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecNoScalar, GenTypeHalfVecNoScalar, Half, Half], Attr.Const>;
}
foreach name = ["mix"] in {
  def : Builtin<name, [FGenTypeN, FGenTypeN, FGenTypeN, FGenTypeN], Attr.Const>;
```

- **L726**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L727**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L728**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L729**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L730**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L731**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L732**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L733**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L734**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L735**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L736**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L737**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L738**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L739**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L740**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L741**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L742**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L743**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L744**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L745**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L746**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L747**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L748**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L749**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L750**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 751-775 / 第 751-775 行

```tablegen
  def : Builtin<name, [GenTypeFloatVecNoScalar, GenTypeFloatVecNoScalar, GenTypeFloatVecNoScalar, Float], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecNoScalar, GenTypeDoubleVecNoScalar, GenTypeDoubleVecNoScalar, Double], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecNoScalar, GenTypeHalfVecNoScalar, GenTypeHalfVecNoScalar, Half], Attr.Const>;
}
foreach name = ["smoothstep"] in {
  def : Builtin<name, [FGenTypeN, FGenTypeN, FGenTypeN, FGenTypeN], Attr.Const>;
  def : Builtin<name, [GenTypeFloatVecNoScalar, Float, Float, GenTypeFloatVecNoScalar], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecNoScalar, Double, Double, GenTypeDoubleVecNoScalar], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecNoScalar, Half, Half, GenTypeHalfVecNoScalar], Attr.Const>;
}


//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.5, v1.2 s6.12.5, v2.0 s6.13.5 - Geometric Functions
// OpenCL Extension v2.0 s5.1.4 and s6.1.4 - Geometric Functions
// --- Table 13 ---
// --- 1 argument ---
foreach name = ["length"] in {
  def : Builtin<name, [Float, GenTypeFloatVec1234], Attr.Const>;
  def : Builtin<name, [Double, GenTypeDoubleVec1234], Attr.Const>;
  def : Builtin<name, [Half, GenTypeHalfVec1234], Attr.Const>;
}
foreach name = ["normalize"] in {
  def : Builtin<name, [GenTypeFloatVec1234, GenTypeFloatVec1234], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVec1234, GenTypeDoubleVec1234], Attr.Const>;
```

- **L751**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L752**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L753**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L754**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L755**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L756**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L757**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L758**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L759**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L760**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L761**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L762**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L763**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L764**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L765**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L766**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L767**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L768**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L769**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L770**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L771**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L772**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L773**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L774**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L775**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 776-800 / 第 776-800 行

```tablegen
  def : Builtin<name, [GenTypeHalfVec1234, GenTypeHalfVec1234], Attr.Const>;
}
foreach name = ["fast_length"] in {
  def : Builtin<name, [Float, GenTypeFloatVec1234], Attr.Const>;
}
foreach name = ["fast_normalize"] in {
  def : Builtin<name, [GenTypeFloatVec1234, GenTypeFloatVec1234], Attr.Const>;
}

// --- 2 arguments ---
foreach name = ["cross"] in {
  foreach VSize = [3, 4] in {
    def : Builtin<name, [VectorType<Float, VSize>, VectorType<Float, VSize>, VectorType<Float, VSize>], Attr.Const>;
    def : Builtin<name, [VectorType<Double, VSize>, VectorType<Double, VSize>, VectorType<Double, VSize>], Attr.Const>;
    def : Builtin<name, [VectorType<Half, VSize>, VectorType<Half, VSize>, VectorType<Half, VSize>], Attr.Const>;
  }
}
foreach name = ["dot", "distance"] in {
  def : Builtin<name, [Float, GenTypeFloatVec1234, GenTypeFloatVec1234], Attr.Const>;
  def : Builtin<name, [Double, GenTypeDoubleVec1234, GenTypeDoubleVec1234], Attr.Const>;
  def : Builtin<name, [Half, GenTypeHalfVec1234, GenTypeHalfVec1234], Attr.Const>;
}
foreach name = ["fast_distance"] in {
  def : Builtin<name, [Float, GenTypeFloatVec1234, GenTypeFloatVec1234], Attr.Const>;
}
```

- **L776**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L777**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L778**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L779**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L780**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L781**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L782**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L783**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L784**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L785**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L786**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L787**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L788**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L789**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L790**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L791**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L792**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L793**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L794**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L795**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L796**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L797**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L798**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L799**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L800**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 801-825 / 第 801-825 行

```tablegen


//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.6, v1.2 s6.12.6, v2.0 s6.13.6 - Relational Functions
// OpenCL Extension v2.0 s5.1.5 and s6.1.5 - Relational Functions
// --- Table 14 ---
// --- 1 argument ---
foreach name = ["isfinite", "isinf", "isnan", "isnormal", "signbit"] in {
  def : Builtin<name, [GenTypeIntVecAndScalar, GenTypeFloatVecAndScalar], Attr.Const>;
  def : Builtin<name, [Int, Double], Attr.Const>;
  def : Builtin<name, [GenTypeLongVecNoScalar, GenTypeDoubleVecNoScalar], Attr.Const>;
  def : Builtin<name, [Int, Half], Attr.Const>;
  def : Builtin<name, [GenTypeShortVecNoScalar, GenTypeHalfVecNoScalar], Attr.Const>;
}
foreach name = ["any", "all"] in {
  def : Builtin<name, [Int, SGenTypeN], Attr.Const>;
}

// --- 2 arguments ---
foreach name = ["isequal", "isnotequal", "isgreater", "isgreaterequal",
                "isless", "islessequal", "islessgreater", "isordered",
                "isunordered"] in {
  def : Builtin<name, [GenTypeIntVecAndScalar, GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar], Attr.Const>;
  def : Builtin<name, [Int, Double, Double], Attr.Const>;
  def : Builtin<name, [GenTypeLongVecNoScalar, GenTypeDoubleVecNoScalar, GenTypeDoubleVecNoScalar], Attr.Const>;
```

- **L801**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L802**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L803**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L804**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L805**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L806**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L807**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L808**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L809**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L810**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L811**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L812**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L813**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L814**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L815**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L816**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L817**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L818**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L819**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L820**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L821**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L822**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L823**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L824**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L825**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 826-850 / 第 826-850 行

```tablegen
  def : Builtin<name, [Int, Half, Half], Attr.Const>;
  def : Builtin<name, [GenTypeShortVecNoScalar, GenTypeHalfVecNoScalar, GenTypeHalfVecNoScalar], Attr.Const>;
}

// --- 3 arguments ---
foreach name = ["bitselect"] in {
  def : Builtin<name, [AGenTypeN, AGenTypeN, AGenTypeN, AGenTypeN], Attr.Const>;
}
foreach name = ["select"] in {
  def : Builtin<name, [SGenTypeN, SGenTypeN, SGenTypeN, SGenTypeN], Attr.Const>;
  def : Builtin<name, [SGenTypeN, SGenTypeN, SGenTypeN, UGenTypeN], Attr.Const>;
  def : Builtin<name, [UGenTypeN, UGenTypeN, UGenTypeN, UGenTypeN], Attr.Const>;
  def : Builtin<name, [UGenTypeN, UGenTypeN, UGenTypeN, SGenTypeN], Attr.Const>;
  def : Builtin<name, [GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar, GenTypeIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar, GenTypeFloatVecAndScalar, GenTypeUIntVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecAndScalar, GenTypeDoubleVecAndScalar, GenTypeDoubleVecAndScalar, GenTypeLongVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeDoubleVecAndScalar, GenTypeDoubleVecAndScalar, GenTypeDoubleVecAndScalar, GenTypeULongVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecAndScalar, GenTypeHalfVecAndScalar, GenTypeHalfVecAndScalar, GenTypeShortVecAndScalar], Attr.Const>;
  def : Builtin<name, [GenTypeHalfVecAndScalar, GenTypeHalfVecAndScalar, GenTypeHalfVecAndScalar, GenTypeUShortVecAndScalar], Attr.Const>;
}


//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.7, v1.2 s6.12.7, v2.0 s6.13.7 - Vector Data Load and Store Functions
// OpenCL Extension v1.1 s9.3.6 and s9.6.6, v1.2 s9.5.6, v2.0 s5.1.6 and s6.1.6 - Vector Data Load and Store Functions
```

- **L826**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L827**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L828**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L829**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L830**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L831**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L832**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L833**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L834**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L835**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L836**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L837**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L838**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L839**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L840**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L841**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L842**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L843**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L844**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L845**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L846**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L847**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L848**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L849**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L850**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。

### Lines 851-875 / 第 851-875 行

```tablegen
// --- Table 15 ---
multiclass VloadVstore<list<AddressSpace> addrspaces, bit defStores> {
  foreach AS = addrspaces in {
    foreach VSize = [2, 3, 4, 8, 16] in {
      foreach name = ["vload" # VSize] in {
        def : Builtin<name, [VectorType<Char, VSize>, Size, PointerType<ConstType<Char>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<UChar, VSize>, Size, PointerType<ConstType<UChar>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<Short, VSize>, Size, PointerType<ConstType<Short>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<UShort, VSize>, Size, PointerType<ConstType<UShort>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<Int, VSize>, Size, PointerType<ConstType<Int>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<UInt, VSize>, Size, PointerType<ConstType<UInt>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<Long, VSize>, Size, PointerType<ConstType<Long>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<ULong, VSize>, Size, PointerType<ConstType<ULong>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<Float, VSize>, Size, PointerType<ConstType<Float>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<Double, VSize>, Size, PointerType<ConstType<Double>, AS>], Attr.Pure>;
        def : Builtin<name, [VectorType<Half, VSize>, Size, PointerType<ConstType<Half>, AS>], Attr.Pure>;
      }
      if defStores then {
        foreach name = ["vstore" # VSize] in {
          def : Builtin<name, [Void, VectorType<Char, VSize>, Size, PointerType<Char, AS>]>;
          def : Builtin<name, [Void, VectorType<UChar, VSize>, Size, PointerType<UChar, AS>]>;
          def : Builtin<name, [Void, VectorType<Short, VSize>, Size, PointerType<Short, AS>]>;
          def : Builtin<name, [Void, VectorType<UShort, VSize>, Size, PointerType<UShort, AS>]>;
          def : Builtin<name, [Void, VectorType<Int, VSize>, Size, PointerType<Int, AS>]>;
          def : Builtin<name, [Void, VectorType<UInt, VSize>, Size, PointerType<UInt, AS>]>;
```

- **L851**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L852**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L853**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L854**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L855**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L856**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L857**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L858**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L859**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L860**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L861**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L862**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L863**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L864**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L865**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L866**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L867**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L868**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L869**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L870**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L871**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L872**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L873**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L874**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L875**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 876-900 / 第 876-900 行

```tablegen
          def : Builtin<name, [Void, VectorType<Long, VSize>, Size, PointerType<Long, AS>]>;
          def : Builtin<name, [Void, VectorType<ULong, VSize>, Size, PointerType<ULong, AS>]>;
          def : Builtin<name, [Void, VectorType<Float, VSize>, Size, PointerType<Float, AS>]>;
          def : Builtin<name, [Void, VectorType<Double, VSize>, Size, PointerType<Double, AS>]>;
          def : Builtin<name, [Void, VectorType<Half, VSize>, Size, PointerType<Half, AS>]>;
        }
      }
    }
  }
}

let Extension = FuncExtOpenCLCNamedAddressSpaceBuiltins in {
  defm : VloadVstore<[GlobalAS, LocalAS, PrivateAS], 1>;
}
let Extension = FuncExtOpenCLCGenericAddressSpace in {
  defm : VloadVstore<[GenericAS], 1>;
}
// vload with constant address space is available regardless of version.
defm : VloadVstore<[ConstantAS], 0>;

multiclass VloadVstoreHalf<list<AddressSpace> addrspaces, bit defStores> {
  foreach AS = addrspaces in {
    def : Builtin<"vload_half", [Float, Size, !cast<Type>("HalfPtrConst" # AS)], Attr.Pure>;
    foreach VSize = [2, 3, 4, 8, 16] in {
      foreach name = ["vload_half" # VSize, "vloada_half" # VSize] in {
```

- **L876**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L877**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L878**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L879**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L880**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L881**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L882**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L883**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L884**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L885**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L886**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L887**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L888**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L889**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L890**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L891**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L892**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L893**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L894**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L895**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L896**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L897**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L898**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L899**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L900**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 901-925 / 第 901-925 行

```tablegen
        def : Builtin<name, [VectorType<Float, VSize>, Size, !cast<Type>("HalfPtrConst" # AS)], Attr.Pure>;
      }
    }
    if defStores then {
      foreach rnd = ["", "_rte", "_rtz", "_rtp", "_rtn"] in {
        foreach name = ["vstore_half" # rnd] in {
          def : Builtin<name, [Void, Float, Size, !cast<Type>("HalfPtr" # AS)]>;
          def : Builtin<name, [Void, Double, Size, !cast<Type>("HalfPtr" # AS)]>;
        }
        foreach VSize = [2, 3, 4, 8, 16] in {
          foreach name = ["vstore_half" # VSize # rnd, "vstorea_half" # VSize # rnd] in {
            def : Builtin<name, [Void, VectorType<Float, VSize>, Size, !cast<Type>("HalfPtr" # AS)]>;
            def : Builtin<name, [Void, VectorType<Double, VSize>, Size, !cast<Type>("HalfPtr" # AS)]>;
          }
        }
      }
    }
  }
}

let Extension = FuncExtOpenCLCNamedAddressSpaceBuiltins in {
  defm : VloadVstoreHalf<[GlobalAS, LocalAS, PrivateAS], 1>;
}
let Extension = FuncExtOpenCLCGenericAddressSpace in {
  defm : VloadVstoreHalf<[GenericAS], 1>;
```

- **L901**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L902**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L903**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L904**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L905**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L906**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L907**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L908**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L909**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L910**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L911**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L912**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L913**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L914**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L915**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L916**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L917**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L918**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L919**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L920**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L921**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L922**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L923**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L924**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L925**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 926-950 / 第 926-950 行

```tablegen
}
// vload_half and vloada_half with constant address space are available regardless of version.
defm : VloadVstoreHalf<[ConstantAS], 0>;

// OpenCL v3.0 s6.15.8 - Synchronization Functions.
def : Builtin<"barrier", [Void, MemFenceFlags], Attr.Convergent>;
let MinVersion = CL20 in {
  def : Builtin<"work_group_barrier", [Void, MemFenceFlags], Attr.Convergent>;
  def : Builtin<"work_group_barrier", [Void, MemFenceFlags, MemoryScope], Attr.Convergent>;
}

// OpenCL v3.0 s6.15.9 - Legacy Explicit Memory Fence Functions.
def : Builtin<"mem_fence", [Void, MemFenceFlags]>;
def : Builtin<"read_mem_fence", [Void, MemFenceFlags]>;
def : Builtin<"write_mem_fence", [Void, MemFenceFlags]>;

// OpenCL v3.0 s6.15.10 - Address Space Qualifier Functions.
// to_global, to_local, to_private are declared in Builtins.td.

let Extension = FuncExtOpenCLCGenericAddressSpace in {
  // The OpenCL 3.0 specification defines these with a "gentype" argument indicating any builtin
  // type or user-defined type, which cannot be represented currently.  Hence we slightly diverge
  // by providing only the following overloads with a void pointer.
  def : Builtin<"get_fence", [MemFenceFlags, PointerType<Void, GenericAS>]>;
  def : Builtin<"get_fence", [MemFenceFlags, PointerType<ConstType<Void>, GenericAS>]>;
```

- **L926**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L927**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L928**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L929**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L930**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L931**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L932**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L933**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L934**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L935**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L936**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L937**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L938**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L939**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L940**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L941**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L942**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L943**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L944**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L945**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L946**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L947**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L948**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L949**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L950**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 951-975 / 第 951-975 行

```tablegen
}

//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.10, v1.2 s6.12.10, v2.0 s6.13.10: Async Copies from Global to Local Memory, Local to Global Memory, and Prefetch
// OpenCL Extension v2.0 s5.1.7 and s6.1.7: Async Copies from Global to Local Memory, Local to Global Memory, and Prefetch
// --- Table 18 ---
foreach name = ["async_work_group_copy"] in {
  def : Builtin<name, [Event, PointerType<AGenTypeN, LocalAS>, PointerType<ConstType<AGenTypeN>, GlobalAS>, Size, Event]>;
  def : Builtin<name, [Event, PointerType<AGenTypeN, GlobalAS>, PointerType<ConstType<AGenTypeN>, LocalAS>, Size, Event]>;
}
foreach name = ["async_work_group_strided_copy"] in {
  def : Builtin<name, [Event, PointerType<AGenTypeN, LocalAS>, PointerType<ConstType<AGenTypeN>, GlobalAS>, Size, Size, Event]>;
  def : Builtin<name, [Event, PointerType<AGenTypeN, GlobalAS>, PointerType<ConstType<AGenTypeN>, LocalAS>, Size, Size, Event]>;
}
foreach name = ["prefetch"] in {
  def : Builtin<name, [Void, PointerType<ConstType<AGenTypeN>, GlobalAS>, Size]>;
}

// The wait_group_events is declared with an argument of type event_t*.
// The address-space of the pointer parameter is different if the generic address space is available.
multiclass BuiltinWithDefaultPointerArg<AddressSpace AS> {
  foreach name = ["wait_group_events"] in {
    def : Builtin<name, [Void, Int, PointerType<Event, AS>]>;
  }
}
```

- **L951**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L952**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L953**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L954**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L955**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L956**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L957**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L958**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L959**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L960**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L961**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L962**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L963**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L964**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L965**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L966**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L967**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L968**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L969**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L970**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L971**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L972**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L973**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L974**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L975**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 976-1000 / 第 976-1000 行

```tablegen

let Extension = FuncExtOpenCLCNamedAddressSpaceBuiltins in {
  defm : BuiltinWithDefaultPointerArg<PrivateAS>;
}
let Extension = FuncExtOpenCLCGenericAddressSpace in {
  defm : BuiltinWithDefaultPointerArg<GenericAS>;
}

//--------------------------------------------------------------------
// OpenCL v2.0 s6.13.11 - Atomics Functions.
// Functions that use memory_order and cl_mem_fence_flags enums are not
// declared here as the TableGen backend does not handle enums.

// OpenCL v1.0 s9.5, s9.6, s9.7 - Atomic Functions for 32-bit integers
// --- Table 9.1 ---
let Extension = FuncExtKhrGlobalInt32BaseAtomics in {
  foreach Type = [Int, UInt] in {
    foreach name = ["atom_add", "atom_sub", "atom_xchg"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, GlobalAS>, Type]>;
    }
    foreach name = ["atom_inc", "atom_dec"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, GlobalAS>]>;
    }
    foreach name = ["atom_cmpxchg"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, GlobalAS>, Type, Type]>;
```

- **L976**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L977**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L978**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L979**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L980**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L981**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L982**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L983**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L984**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L985**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L986**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L987**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L988**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L989**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L990**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L991**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L992**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L993**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L994**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L995**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L996**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L997**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L998**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L999**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1000**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1001-1025 / 第 1001-1025 行

```tablegen
    }
  }
}
// --- Table 9.3 ---
let Extension = FuncExtKhrLocalInt32BaseAtomics in {
  foreach Type = [Int, UInt] in {
    foreach name = ["atom_add", "atom_sub", "atom_xchg"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, LocalAS>, Type]>;
    }
    foreach name = ["atom_inc", "atom_dec"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, LocalAS>]>;
    }
    foreach name = ["atom_cmpxchg"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, LocalAS>, Type, Type]>;
    }
  }
}
// --- Table 9.5 ---
let Extension = FuncExtKhrInt64BaseAtomics in {
  foreach AS = [GlobalAS, LocalAS] in {
    foreach Type = [Long, ULong] in {
      foreach name = ["atom_add", "atom_sub", "atom_xchg"] in {
        def : Builtin<name, [Type, PointerType<VolatileType<Type>, AS>, Type]>;
      }
      foreach name = ["atom_inc", "atom_dec"] in {
```

- **L1001**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1002**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1003**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1004**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1005**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1006**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1007**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1008**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1009**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1010**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1011**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1012**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1013**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1014**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1015**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1016**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1017**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1018**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1019**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1020**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1021**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1022**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1023**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1024**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1025**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1026-1050 / 第 1026-1050 行

```tablegen
        def : Builtin<name, [Type, PointerType<VolatileType<Type>, AS>]>;
      }
      foreach name = ["atom_cmpxchg"] in {
        def : Builtin<name, [Type, PointerType<VolatileType<Type>, AS>, Type, Type]>;
      }
    }
  }
}
// --- Table 9.2 ---
let Extension = FuncExtKhrGlobalInt32ExtendedAtomics in {
  foreach Type = [Int, UInt] in {
    foreach name = ["atom_min", "atom_max", "atom_and",
                    "atom_or", "atom_xor"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, GlobalAS>, Type]>;
    }
  }
}
// --- Table 9.4 ---
let Extension = FuncExtKhrLocalInt32ExtendedAtomics in {
  foreach Type = [Int, UInt] in {
    foreach name = ["atom_min", "atom_max", "atom_and",
                    "atom_or", "atom_xor"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, LocalAS>, Type]>;
    }
  }
```

- **L1026**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1027**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1028**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1029**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1030**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1031**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1032**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1033**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1034**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1035**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1036**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1037**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1038**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1039**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1040**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1041**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1042**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1043**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1044**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1045**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1046**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1047**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1048**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1049**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1050**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 1051-1075 / 第 1051-1075 行

```tablegen
}
// --- Table 9.6 ---
let Extension = FuncExtKhrInt64ExtendedAtomics in {
  foreach AS = [GlobalAS, LocalAS] in {
    foreach Type = [Long, ULong] in {
      foreach name = ["atom_min", "atom_max", "atom_and",
                      "atom_or", "atom_xor"] in {
        def : Builtin<name, [Type, PointerType<VolatileType<Type>, AS>, Type]>;
      }
    }
  }
}
// OpenCL v1.1 s6.11.1, v1.2 s6.12.11 - Atomic Functions
foreach AS = [GlobalAS, LocalAS] in {
  def : Builtin<"atomic_xchg", [Float, PointerType<VolatileType<Float>, AS>, Float]>;
  foreach Type = [Int, UInt] in {
    foreach name = ["atomic_add", "atomic_sub", "atomic_xchg",
                    "atomic_min", "atomic_max", "atomic_and",
                    "atomic_or", "atomic_xor"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, AS>, Type]>;
    }
    foreach name = ["atomic_inc", "atomic_dec"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, AS>]>;
    }
    foreach name = ["atomic_cmpxchg"] in {
```

- **L1051**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1052**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1053**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1054**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1055**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1056**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1057**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1058**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1059**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1060**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1061**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1062**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1063**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1064**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1065**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1066**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1067**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1068**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1069**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1070**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1071**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1072**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1073**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1074**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1075**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1076-1100 / 第 1076-1100 行

```tablegen
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, AS>, Type, Type]>;
    }
  }
}

let Extension = FuncExtOpenCLCxx in {
  foreach Type = [Int, UInt] in {
    foreach name = ["atomic_add", "atomic_sub", "atomic_xchg",
                    "atomic_min", "atomic_max", "atomic_and",
                    "atomic_or", "atomic_xor"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, GenericAS>, Type]>;
    }
    foreach name = ["atomic_inc", "atomic_dec"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, GenericAS>]>;
    }
    foreach name = ["atomic_cmpxchg"] in {
      def : Builtin<name, [Type, PointerType<VolatileType<Type>, GenericAS>, Type, Type]>;
    }
  }
}

// OpenCL v2.0 s6.13.11 - Atomic Functions.

// An atomic builtin with 2 additional _explicit variants.
multiclass BuiltinAtomicExplicit<string Name, list<Type> Types, FunctionExtension BaseExt> {
```

- **L1076**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1077**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1078**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1079**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1080**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1081**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1082**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1083**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1084**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1085**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1086**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1087**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1088**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1089**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1090**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1091**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1092**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1093**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1094**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1095**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1096**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1097**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1098**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1099**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1100**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1101-1125 / 第 1101-1125 行

```tablegen
  // Without explicit MemoryOrder or MemoryScope.
  let Extension = concatExtension<BaseExt, "__opencl_c_atomic_order_seq_cst __opencl_c_atomic_scope_device">.ret in {
    def : Builtin<Name, Types>;
  }

  // With an explicit MemoryOrder argument.
  let Extension = concatExtension<BaseExt, "__opencl_c_atomic_scope_device">.ret in {
    def : Builtin<Name # "_explicit", !listconcat(Types, [MemoryOrder])>;
  }

  // With explicit MemoryOrder and MemoryScope arguments.
  let Extension = BaseExt in {
    def : Builtin<Name # "_explicit", !listconcat(Types, [MemoryOrder, MemoryScope])>;
  }
}

// OpenCL 2.0 atomic functions that have a pointer argument in a given address space.
multiclass OpenCL2Atomics<AddressSpace addrspace, FunctionExtension BaseExt> {
  foreach TypePair = [[AtomicInt, Int], [AtomicUInt, UInt],
                      [AtomicLong, Long], [AtomicULong, ULong],
                      [AtomicFloat, Float], [AtomicDouble, Double]] in {
    let Extension = BaseExt in {
      def : Builtin<"atomic_init",
          [Void, PointerType<VolatileType<TypePair[0]>, addrspace>, TypePair[1]]>;
    }
```

- **L1101**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1102**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1103**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1104**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1105**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1106**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1107**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1108**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L1109**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1110**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1111**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1112**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1113**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L1114**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1115**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1116**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1117**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1118**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1119**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1120**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1121**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1122**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1123**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1124**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1125**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 1126-1150 / 第 1126-1150 行

```tablegen
    defm : BuiltinAtomicExplicit<"atomic_store",
        [Void, PointerType<VolatileType<TypePair[0]>, addrspace>, TypePair[1]], BaseExt>;
    defm : BuiltinAtomicExplicit<"atomic_load",
        [TypePair[1], PointerType<VolatileType<TypePair[0]>, addrspace>], BaseExt>;
    defm : BuiltinAtomicExplicit<"atomic_exchange",
        [TypePair[1], PointerType<VolatileType<TypePair[0]>, addrspace>, TypePair[1]], BaseExt>;
    foreach Variant = ["weak", "strong"] in {
      foreach exp_ptr_addrspace = !cond(
            !eq(BaseExt, FuncExtOpenCLCGenericAddressSpace): [GenericAS],
            !eq(BaseExt, FuncExtOpenCLCNamedAddressSpaceBuiltins): [GlobalAS, LocalAS, PrivateAS])
          in {
        let Extension = concatExtension<BaseExt, "__opencl_c_atomic_order_seq_cst __opencl_c_atomic_scope_device">.ret in {
          def : Builtin<"atomic_compare_exchange_" # Variant,
              [Bool, PointerType<VolatileType<TypePair[0]>, addrspace>,
               PointerType<TypePair[1], exp_ptr_addrspace>, TypePair[1]]>;
        }
        let Extension = concatExtension<BaseExt, "__opencl_c_atomic_scope_device">.ret in {
          def : Builtin<"atomic_compare_exchange_" # Variant # "_explicit",
              [Bool, PointerType<VolatileType<TypePair[0]>, addrspace>,
               PointerType<TypePair[1], exp_ptr_addrspace>, TypePair[1], MemoryOrder, MemoryOrder]>;
        }
        let Extension = BaseExt in {
          def : Builtin<"atomic_compare_exchange_" # Variant # "_explicit",
              [Bool, PointerType<VolatileType<TypePair[0]>, addrspace>,
               PointerType<TypePair[1], exp_ptr_addrspace>, TypePair[1], MemoryOrder, MemoryOrder, MemoryScope]>;
```

- **L1126**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1127**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1128**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1129**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1130**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1131**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1132**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1133**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1134**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1135**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1136**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1137**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1138**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1139**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1140**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1141**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1142**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1143**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1144**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1145**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1146**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1147**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1148**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1149**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1150**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1151-1175 / 第 1151-1175 行

```tablegen
        }
      }
    }
  }

  foreach TypePair = [[AtomicInt, Int, Int], [AtomicUInt, UInt, UInt],
                      [AtomicLong, Long, Long], [AtomicULong, ULong, ULong],
                      [AtomicUIntPtr, UIntPtr, PtrDiff]] in {
    foreach ModOp = ["add", "sub"] in {
      defm : BuiltinAtomicExplicit<"atomic_fetch_" # ModOp,
          [TypePair[1], PointerType<VolatileType<TypePair[0]>, addrspace>, TypePair[2]], BaseExt>;
    }
  }
  foreach TypePair = [[AtomicInt, Int, Int], [AtomicUInt, UInt, UInt],
                      [AtomicLong, Long, Long], [AtomicULong, ULong, ULong]] in {
    foreach ModOp = ["or", "xor", "and", "min", "max"] in {
      defm : BuiltinAtomicExplicit<"atomic_fetch_" # ModOp,
          [TypePair[1], PointerType<VolatileType<TypePair[0]>, addrspace>, TypePair[2]], BaseExt>;
    }
  }

  defm : BuiltinAtomicExplicit<"atomic_flag_clear",
      [Void, PointerType<VolatileType<AtomicFlag>, addrspace>], BaseExt>;

  defm : BuiltinAtomicExplicit<"atomic_flag_test_and_set",
```

- **L1151**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1152**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1153**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1154**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1155**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1156**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1157**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1158**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1159**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1160**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1161**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1162**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1163**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1164**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1165**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1166**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1167**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1168**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1169**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1170**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1171**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1172**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1173**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1174**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1175**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 1176-1200 / 第 1176-1200 行

```tablegen
      [Bool, PointerType<VolatileType<AtomicFlag>, addrspace>], BaseExt>;
}

let MinVersion = CL20 in {
  def : Builtin<"atomic_work_item_fence", [Void, MemFenceFlags, MemoryOrder, MemoryScope]>;

  defm : OpenCL2Atomics<GenericAS, FuncExtOpenCLCGenericAddressSpace>;
  defm : OpenCL2Atomics<GlobalAS, FuncExtOpenCLCNamedAddressSpaceBuiltins>;
  defm : OpenCL2Atomics<LocalAS, FuncExtOpenCLCNamedAddressSpaceBuiltins>;
}

// The functionality added by cl_ext_float_atomics extension
let MinVersion = CL20 in {
  foreach addrspace = [GlobalAS, LocalAS, GenericAS] in {
    defvar extension_fp16 = !cast<FunctionExtension>("FuncExtFloatAtomicsFp16" # addrspace # "LoadStore");

    defm : BuiltinAtomicExplicit<"atomic_store",
        [Void, PointerType<VolatileType<AtomicHalf>, addrspace>, AtomicHalf], extension_fp16>;
    defm : BuiltinAtomicExplicit<"atomic_load",
        [Half, PointerType<VolatileType<AtomicHalf>, addrspace>], extension_fp16>;
    defm : BuiltinAtomicExplicit<"atomic_exchange",
        [Half, PointerType<VolatileType<AtomicHalf>, addrspace>, Half], extension_fp16>;

    foreach ModOp = ["add", "sub"] in {
      defvar extension_fp16 = !cast<FunctionExtension>("FuncExtFloatAtomicsFp16" # addrspace # "Add");
```

- **L1176**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1177**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1178**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1179**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1180**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1181**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1182**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1183**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1184**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1185**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1186**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1187**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1188**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1189**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1190**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L1191**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1192**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1193**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1194**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1195**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1196**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1197**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1198**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1199**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1200**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。

### Lines 1201-1225 / 第 1201-1225 行

```tablegen
      defvar extension_fp32 = !cast<FunctionExtension>("FuncExtFloatAtomicsFp32" # addrspace # "Add");
      defvar extension_fp64 = !cast<FunctionExtension>("FuncExtFloatAtomicsFp64" # addrspace # "Add");

      defm : BuiltinAtomicExplicit<"atomic_fetch_" # ModOp,
          [Half, PointerType<VolatileType<AtomicHalf>, addrspace>, Half], extension_fp16>;
      defm : BuiltinAtomicExplicit<"atomic_fetch_" # ModOp,
          [Float, PointerType<VolatileType<AtomicFloat>, addrspace>, Float], extension_fp32>;
      defm : BuiltinAtomicExplicit<"atomic_fetch_" # ModOp,
          [Double, PointerType<VolatileType<AtomicDouble>, addrspace>, Double], extension_fp64>;
    }

    foreach ModOp = ["min", "max"] in {
      defvar extension_fp16 = !cast<FunctionExtension>("FuncExtFloatAtomicsFp16" # addrspace # "MinMax");
      defvar extension_fp32 = !cast<FunctionExtension>("FuncExtFloatAtomicsFp32" # addrspace # "MinMax");
      defvar extension_fp64 = !cast<FunctionExtension>("FuncExtFloatAtomicsFp64" # addrspace # "MinMax");

      defm : BuiltinAtomicExplicit<"atomic_fetch_" # ModOp,
          [Half, PointerType<VolatileType<AtomicHalf>, addrspace>, Half], extension_fp16>;
      defm : BuiltinAtomicExplicit<"atomic_fetch_" # ModOp,
          [Float, PointerType<VolatileType<AtomicFloat>, addrspace>, Float], extension_fp32>;
      defm : BuiltinAtomicExplicit<"atomic_fetch_" # ModOp,
          [Double, PointerType<VolatileType<AtomicDouble>, addrspace>, Double], extension_fp64>;
    }
  }
}
```

- **L1201**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L1202**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L1203**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1204**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1205**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1206**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1207**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1208**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1209**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1210**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1211**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1212**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1213**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L1214**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L1215**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L1216**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1217**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1218**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1219**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1220**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1221**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1222**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1223**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1224**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1225**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 1226-1250 / 第 1226-1250 行

```tablegen

//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.12, v1.2 s6.12.12, v2.0 s6.13.12 - Miscellaneous Vector Functions
// --- Table 19 ---
foreach VSize1 = [2, 4, 8, 16] in {
  foreach VSize2 = [2, 4, 8, 16] in {
    foreach VecAndMaskType = [[Char, UChar], [UChar, UChar],
                              [Short, UShort], [UShort, UShort],
                              [Int, UInt], [UInt, UInt],
                              [Long, ULong], [ULong, ULong],
                              [Float, UInt], [Double, ULong], [Half, UShort]] in {
      def : Builtin<"shuffle", [VectorType<VecAndMaskType[0], VSize1>,
                                VectorType<VecAndMaskType[0], VSize2>,
                                VectorType<VecAndMaskType[1], VSize1>],
                               Attr.Const>;
    }
  }
}
foreach VSize1 = [2, 4, 8, 16] in {
  foreach VSize2 = [2, 4, 8, 16] in {
    foreach VecAndMaskType = [[Char, UChar], [UChar, UChar],
                              [Short, UShort], [UShort, UShort],
                              [Int, UInt], [UInt, UInt],
                              [Long, ULong], [ULong, ULong],
                              [Float, UInt], [Double, ULong], [Half, UShort]] in {
```

- **L1226**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1227**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1228**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1229**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1230**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1231**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1232**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1233**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1234**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1235**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1236**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1237**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1238**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1239**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1240**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1241**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1242**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1243**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1244**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1245**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1246**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1247**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1248**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1249**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1250**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1251-1275 / 第 1251-1275 行

```tablegen
      def : Builtin<"shuffle2", [VectorType<VecAndMaskType[0], VSize1>,
                                 VectorType<VecAndMaskType[0], VSize2>,
                                 VectorType<VecAndMaskType[0], VSize2>,
                                 VectorType<VecAndMaskType[1], VSize1>],
                                Attr.Const>;
    }
  }
}

//--------------------------------------------------------------------
// OpenCL v1.1 s6.11.3, v1.2 s6.12.14, v2.0 s6.13.14: Image Read and Write Functions
// OpenCL Extension v2.0 s5.1.8 and s6.1.8: Image Read and Write Functions
// --- Table 22: Image Read Functions with Samplers ---
foreach imgTy = [Image1d] in {
  foreach coordTy = [Int, Float] in {
    def : Builtin<"read_imagef", [VectorType<Float, 4>, ImageType<imgTy, "RO">, Sampler, coordTy], Attr.Pure>;
    def : Builtin<"read_imagei", [VectorType<Int, 4>, ImageType<imgTy, "RO">, Sampler, coordTy], Attr.Pure>;
    def : Builtin<"read_imageui", [VectorType<UInt, 4>, ImageType<imgTy, "RO">, Sampler, coordTy], Attr.Pure>;
  }
}
foreach imgTy = [Image2d, Image1dArray] in {
  foreach coordTy = [Int, Float] in {
    def : Builtin<"read_imagef", [VectorType<Float, 4>, ImageType<imgTy, "RO">, Sampler, VectorType<coordTy, 2>], Attr.Pure>;
    def : Builtin<"read_imagei", [VectorType<Int, 4>, ImageType<imgTy, "RO">, Sampler, VectorType<coordTy, 2>], Attr.Pure>;
    def : Builtin<"read_imageui", [VectorType<UInt, 4>, ImageType<imgTy, "RO">, Sampler, VectorType<coordTy, 2>], Attr.Pure>;
```

- **L1251**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1252**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1253**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1254**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1255**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1256**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1257**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1258**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1259**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1260**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1261**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1262**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1263**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1264**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1265**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1266**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1267**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1268**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1269**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1270**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1271**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1272**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1273**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1274**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1275**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1276-1300 / 第 1276-1300 行

```tablegen
  }
}
foreach imgTy = [Image3d, Image2dArray] in {
  foreach coordTy = [Int, Float] in {
    def : Builtin<"read_imagef", [VectorType<Float, 4>, ImageType<imgTy, "RO">, Sampler, VectorType<coordTy, 4>], Attr.Pure>;
    def : Builtin<"read_imagei", [VectorType<Int, 4>, ImageType<imgTy, "RO">, Sampler, VectorType<coordTy, 4>], Attr.Pure>;
    def : Builtin<"read_imageui", [VectorType<UInt, 4>, ImageType<imgTy, "RO">, Sampler, VectorType<coordTy, 4>], Attr.Pure>;
  }
}
foreach coordTy = [Int, Float] in {
  def : Builtin<"read_imagef", [Float, ImageType<Image2dDepth, "RO">, Sampler, VectorType<coordTy, 2>], Attr.Pure>;
  def : Builtin<"read_imagef", [Float, ImageType<Image2dArrayDepth, "RO">, Sampler, VectorType<coordTy, 4>], Attr.Pure>;
}

// --- Table 23: Sampler-less Read Functions ---
multiclass ImageReadSamplerless<string aQual> {
  foreach imgTy = [Image2d, Image1dArray] in {
    def : Builtin<"read_imagef", [VectorType<Float, 4>, ImageType<imgTy, aQual>, VectorType<Int, 2>], Attr.Pure>;
    def : Builtin<"read_imagei", [VectorType<Int, 4>, ImageType<imgTy, aQual>, VectorType<Int, 2>], Attr.Pure>;
    def : Builtin<"read_imageui", [VectorType<UInt, 4>, ImageType<imgTy, aQual>, VectorType<Int, 2>], Attr.Pure>;
  }
  foreach imgTy = [Image3d, Image2dArray] in {
    def : Builtin<"read_imagef", [VectorType<Float, 4>, ImageType<imgTy, aQual>, VectorType<Int, 4>], Attr.Pure>;
    def : Builtin<"read_imagei", [VectorType<Int, 4>, ImageType<imgTy, aQual>, VectorType<Int, 4>], Attr.Pure>;
    def : Builtin<"read_imageui", [VectorType<UInt, 4>, ImageType<imgTy, aQual>, VectorType<Int, 4>], Attr.Pure>;
```

- **L1276**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1277**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1278**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1279**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1280**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1281**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1282**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1283**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1284**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1285**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1286**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1287**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1288**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1289**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1290**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1291**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1292**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1293**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1294**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1295**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1296**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1297**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1298**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1299**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1300**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1301-1325 / 第 1301-1325 行

```tablegen
  }
  foreach imgTy = [Image1d, Image1dBuffer] in {
    def : Builtin<"read_imagef", [VectorType<Float, 4>, ImageType<imgTy, aQual>, Int], Attr.Pure>;
    def : Builtin<"read_imagei", [VectorType<Int, 4>, ImageType<imgTy, aQual>, Int], Attr.Pure>;
    def : Builtin<"read_imageui", [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Int], Attr.Pure>;
  }
  def : Builtin<"read_imagef", [Float, ImageType<Image2dDepth, aQual>, VectorType<Int, 2>], Attr.Pure>;
  def : Builtin<"read_imagef", [Float, ImageType<Image2dArrayDepth, aQual>, VectorType<Int, 4>], Attr.Pure>;
}

let MinVersion = CL12 in {
  defm : ImageReadSamplerless<"RO">;
  let Extension = FuncExtOpenCLCReadWriteImages in {
    defm : ImageReadSamplerless<"RW">;
  }
}

// --- Table 24: Image Write Functions ---
multiclass ImageWrite<string aQual> {
  foreach imgTy = [Image2d] in {
    def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, VectorType<Float, 4>]>;
    def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, VectorType<Int, 4>]>;
    def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, VectorType<UInt, 4>]>;
  }
  foreach imgTy = [Image2dArray] in {
```

- **L1301**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1302**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1303**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1304**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1305**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1306**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1307**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1308**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1309**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1310**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1311**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1312**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1313**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1314**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1315**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1316**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1317**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1318**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1319**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1320**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1321**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1322**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1323**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1324**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1325**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1326-1350 / 第 1326-1350 行

```tablegen
    def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, VectorType<Float, 4>]>;
    def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, VectorType<Int, 4>]>;
    def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, VectorType<UInt, 4>]>;
  }
  foreach imgTy = [Image1d, Image1dBuffer] in {
    def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, Int, VectorType<Float, 4>]>;
    def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, Int, VectorType<Int, 4>]>;
    def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, Int, VectorType<UInt, 4>]>;
  }
  foreach imgTy = [Image1dArray] in {
    def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, VectorType<Float, 4>]>;
    def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, VectorType<Int, 4>]>;
    def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, VectorType<UInt, 4>]>;
  }
  foreach imgTy = [Image3d] in {
    def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, VectorType<Float, 4>]>;
    def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, VectorType<Int, 4>]>;
    def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, VectorType<UInt, 4>]>;
  }
  def : Builtin<"write_imagef", [Void, ImageType<Image2dDepth, aQual>, VectorType<Int, 2>, Float]>;
  def : Builtin<"write_imagef", [Void, ImageType<Image2dArrayDepth, aQual>, VectorType<Int, 4>, Float]>;
}

defm : ImageWrite<"WO">;
let Extension = FuncExtOpenCLCReadWriteImages in {
```

- **L1326**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1327**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1328**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1329**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1330**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1331**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1332**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1333**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1334**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1335**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1336**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1337**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1338**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1339**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1340**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1341**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1342**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1343**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1344**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1345**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1346**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1347**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1348**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1349**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1350**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1351-1375 / 第 1351-1375 行

```tablegen
  defm : ImageWrite<"RW">;
}

// --- Table 25: Image Query Functions ---
multiclass ImageQuery<string aQual> {
  foreach imgTy = [Image1d, Image1dBuffer, Image2d, Image3d,
                   Image1dArray, Image2dArray, Image2dDepth,
                   Image2dArrayDepth] in {
    foreach name = ["get_image_width", "get_image_channel_data_type",
                    "get_image_channel_order"] in {
      def : Builtin<name, [Int, ImageType<imgTy, aQual>], Attr.Const>;
    }
  }
  foreach imgTy = [Image2d, Image3d, Image2dArray, Image2dDepth,
                   Image2dArrayDepth] in {
    def : Builtin<"get_image_height", [Int, ImageType<imgTy, aQual>], Attr.Const>;
  }
  def : Builtin<"get_image_depth", [Int, ImageType<Image3d, aQual>], Attr.Const>;
  foreach imgTy = [Image2d, Image2dArray, Image2dDepth,
                   Image2dArrayDepth] in {
    def : Builtin<"get_image_dim", [VectorType<Int, 2>, ImageType<imgTy, aQual>], Attr.Const>;
  }
  def : Builtin<"get_image_dim", [VectorType<Int, 4>, ImageType<Image3d, aQual>], Attr.Const>;
  foreach imgTy = [Image1dArray, Image2dArray, Image2dArrayDepth] in {
    def : Builtin<"get_image_array_size", [Size, ImageType<imgTy, aQual>], Attr.Const>;
```

- **L1351**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1352**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1353**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1354**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1355**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1356**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1357**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1358**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1359**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1360**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1361**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1362**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1363**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1364**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1365**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1366**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1367**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1368**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1369**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1370**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1371**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1372**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1373**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1374**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1375**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1376-1400 / 第 1376-1400 行

```tablegen
  }
}

defm : ImageQuery<"RO">;
defm : ImageQuery<"WO">;
let Extension = FuncExtOpenCLCReadWriteImages in {
  defm : ImageQuery<"RW">;
}

// OpenCL extension v2.0 s5.1.9: Built-in Image Read Functions
// --- Table 8 ---
foreach aQual = ["RO"] in {
  foreach name = ["read_imageh"] in {
    foreach coordTy = [Int, Float] in {
      foreach imgTy = [Image2d, Image1dArray] in {
        def : Builtin<name, [VectorType<Half, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<coordTy, 2>], Attr.Pure>;
      }
      foreach imgTy = [Image3d, Image2dArray] in {
        def : Builtin<name, [VectorType<Half, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<coordTy, 4>], Attr.Pure>;
      }
      foreach imgTy = [Image1d] in {
        def : Builtin<name, [VectorType<Half, 4>, ImageType<imgTy, aQual>, Sampler, coordTy], Attr.Pure>;
      }
    }
  }
```

- **L1376**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1377**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1378**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1379**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1380**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1381**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1382**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1383**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1384**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1385**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1386**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1387**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1388**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1389**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1390**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1391**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1392**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1393**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1394**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1395**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1396**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1397**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1398**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1399**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1400**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 1401-1425 / 第 1401-1425 行

```tablegen
}
// OpenCL extension v2.0 s5.1.10: Built-in Image Sampler-less Read Functions
// --- Table 9 ---
let MinVersion = CL12 in {
  multiclass ImageReadHalf<string aQual> {
    foreach name = ["read_imageh"] in {
      foreach imgTy = [Image2d, Image1dArray] in {
        def : Builtin<name, [VectorType<Half, 4>, ImageType<imgTy, aQual>, VectorType<Int, 2>], Attr.Pure>;
      }
      foreach imgTy = [Image3d, Image2dArray] in {
        def : Builtin<name, [VectorType<Half, 4>, ImageType<imgTy, aQual>, VectorType<Int, 4>], Attr.Pure>;
      }
      foreach imgTy = [Image1d, Image1dBuffer] in {
        def : Builtin<name, [VectorType<Half, 4>, ImageType<imgTy, aQual>, Int], Attr.Pure>;
      }
    }
  }
  defm : ImageReadHalf<"RO">;
  let Extension = FuncExtOpenCLCReadWriteImages in {
    defm : ImageReadHalf<"RW">;
  }
}
// OpenCL extension v2.0 s5.1.11: Built-in Image Write Functions
// --- Table 10 ---
multiclass ImageWriteHalf<string aQual> {
```

- **L1401**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1402**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1403**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1404**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1405**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1406**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1407**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1408**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1409**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1410**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1411**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1412**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1413**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1414**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1415**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1416**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1417**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1418**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1419**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1420**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1421**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1422**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1423**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1424**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1425**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1426-1450 / 第 1426-1450 行

```tablegen
  foreach name = ["write_imageh"] in {
    def : Builtin<name, [Void, ImageType<Image2d, aQual>, VectorType<Int, 2>, VectorType<Half, 4>]>;
    def : Builtin<name, [Void, ImageType<Image2dArray, aQual>, VectorType<Int, 4>, VectorType<Half, 4>]>;
    def : Builtin<name, [Void, ImageType<Image1d, aQual>, Int, VectorType<Half, 4>]>;
    def : Builtin<name, [Void, ImageType<Image1dBuffer, aQual>, Int, VectorType<Half, 4>]>;
    def : Builtin<name, [Void, ImageType<Image1dArray, aQual>, VectorType<Int, 2>, VectorType<Half, 4>]>;
    def : Builtin<name, [Void, ImageType<Image3d, aQual>, VectorType<Int, 4>, VectorType<Half, 4>]>;
  }
}

defm : ImageWriteHalf<"WO">;
let Extension = FuncExtOpenCLCReadWriteImages in {
  defm : ImageWriteHalf<"RW">;
}



//--------------------------------------------------------------------
// OpenCL v2.0 s6.13.15 - Work-group Functions
// --- Table 26 ---
let Extension = FuncExtOpenCLCWGCollectiveFunctions in {
  foreach name = ["work_group_all", "work_group_any"] in {
    def : Builtin<name, [Int, Int], Attr.Convergent>;
  }
  foreach name = ["work_group_broadcast"] in {
```

- **L1426**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1427**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1428**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1429**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1430**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1431**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1432**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1433**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1434**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1435**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1436**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1437**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1438**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1439**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1440**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1441**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1442**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1443**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1444**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1445**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1446**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1447**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1448**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1449**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1450**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1451-1475 / 第 1451-1475 行

```tablegen
    def : Builtin<name, [IntLongFloatGenType1, IntLongFloatGenType1, Size], Attr.Convergent>;
    def : Builtin<name, [IntLongFloatGenType1, IntLongFloatGenType1, Size, Size], Attr.Convergent>;
    def : Builtin<name, [IntLongFloatGenType1, IntLongFloatGenType1, Size, Size, Size], Attr.Convergent>;
  }
  foreach op = ["add", "min", "max"] in {
    foreach name = ["work_group_reduce_", "work_group_scan_exclusive_",
                    "work_group_scan_inclusive_"] in {
      def : Builtin<name # op, [IntLongFloatGenType1, IntLongFloatGenType1], Attr.Convergent>;
    }
  }
}


//--------------------------------------------------------------------
// OpenCL2.0 : 6.13.16 : Pipe Functions
// --- Table 27 ---
// Defined in Builtins.td

// --- Table 28 ---
// Builtins taking pipe arguments are defined in Builtins.td
let Extension = FuncExtOpenCLCPipes in {
  def : Builtin<"is_valid_reserve_id", [Bool, ReserveId]>;
}

// --- Table 29 ---
```

- **L1451**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1452**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1453**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1454**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1455**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1456**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1457**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1458**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1459**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1460**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1461**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1462**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1463**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1464**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1465**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1466**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1467**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1468**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1469**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1470**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1471**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1472**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1473**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1474**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1475**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。

### Lines 1476-1500 / 第 1476-1500 行

```tablegen
// Defined in Builtins.td


//--------------------------------------------------------------------
// OpenCL2.0 : 6.13.17 : Enqueuing Kernels
// --- Table 30 ---
// Defined in Builtins.td

// --- Table 32 ---
// Defined in Builtins.td

// --- Table 33 ---
let Extension = FuncExtOpenCLCDeviceEnqueue in {
  def : Builtin<"enqueue_marker",
      [Int, Queue, UInt, PointerType<ConstType<ClkEvent>, GenericAS>, PointerType<ClkEvent, GenericAS>]>;

  // --- Table 34 ---
  def : Builtin<"retain_event", [Void, ClkEvent]>;
  def : Builtin<"release_event", [Void, ClkEvent]>;
  def : Builtin<"create_user_event", [ClkEvent]>;
  def : Builtin<"is_valid_event", [Bool, ClkEvent]>;
  def : Builtin<"set_user_event_status", [Void, ClkEvent, Int]>;
  def : Builtin<"capture_event_profiling_info",
      [Void, ClkEvent, ClkProfilingInfo, PointerType<Void, GlobalAS>]>;

```

- **L1476**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1477**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1478**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1479**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1480**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1481**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1482**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1483**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1484**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1485**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1486**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1487**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1488**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1489**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1490**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1491**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1492**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1493**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1494**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1495**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1496**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1497**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1498**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1499**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1500**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 1501-1525 / 第 1501-1525 行

```tablegen
  // --- Table 35 ---
  def : Builtin<"get_default_queue", [Queue]>;

  def : Builtin<"ndrange_1D", [NDRange, Size]>;
  def : Builtin<"ndrange_1D", [NDRange, Size, Size]>;
  def : Builtin<"ndrange_1D", [NDRange, Size, Size, Size]>;
  def : Builtin<"ndrange_2D", [NDRange, PointerType<ConstType<Size>, PrivateAS>]>;
  def : Builtin<"ndrange_2D", [NDRange, PointerType<ConstType<Size>, PrivateAS>,
                                        PointerType<ConstType<Size>, PrivateAS>]>;
  def : Builtin<"ndrange_2D", [NDRange, PointerType<ConstType<Size>, PrivateAS>,
                                        PointerType<ConstType<Size>, PrivateAS>,
                                        PointerType<ConstType<Size>, PrivateAS>]>;
  def : Builtin<"ndrange_3D", [NDRange, PointerType<ConstType<Size>, PrivateAS>]>;
  def : Builtin<"ndrange_3D", [NDRange, PointerType<ConstType<Size>, PrivateAS>,
                                        PointerType<ConstType<Size>, PrivateAS>]>;
  def : Builtin<"ndrange_3D", [NDRange, PointerType<ConstType<Size>, PrivateAS>,
                                        PointerType<ConstType<Size>, PrivateAS>,
                                        PointerType<ConstType<Size>, PrivateAS>]>;
}


//--------------------------------------------------------------------
// End of the builtin functions defined in the OpenCL C specification.
// Builtin functions defined in the OpenCL C Extension are below.
//--------------------------------------------------------------------
```

- **L1501**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1502**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1503**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1504**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1505**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1506**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1507**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1508**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1509**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1510**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1511**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1512**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1513**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1514**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1515**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1516**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1517**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1518**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1519**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1520**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1521**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1522**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1523**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1524**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1525**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。

### Lines 1526-1550 / 第 1526-1550 行

```tablegen


// OpenCL Extension v2.0 s9.18 - Mipmaps
let Extension = FuncExtKhrMipmapImage in {
  // Added to section 6.13.14.2.
  foreach aQual = ["RO", "RW"] in {
    foreach imgTy = [Image2d] in {
      foreach name = ["read_imagef"] in {
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, VectorType<Float, 2>, VectorType<Float, 2>], Attr.Pure>;
      }
      foreach name = ["read_imagei"] in {
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, VectorType<Float, 2>, VectorType<Float, 2>], Attr.Pure>;
      }
      foreach name = ["read_imageui"] in {
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, VectorType<Float, 2>, VectorType<Float, 2>], Attr.Pure>;
      }
    }
    foreach imgTy = [Image2dDepth] in {
      foreach name = ["read_imagef"] in {
        def : Builtin<name, [Float, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float], Attr.Pure>;
        def : Builtin<name, [Float, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, VectorType<Float, 2>, VectorType<Float, 2>], Attr.Pure>;
      }
```

- **L1526**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1527**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1528**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1529**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1530**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1531**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1532**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1533**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1534**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1535**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1536**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1537**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1538**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1539**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1540**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1541**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1542**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1543**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1544**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1545**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1546**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1547**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1548**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1549**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1550**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 1551-1575 / 第 1551-1575 行

```tablegen
    }
    foreach imgTy = [Image1d] in {
      foreach name = ["read_imagef"] in {
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, Float, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, Float, Float, Float], Attr.Pure>;
      }
      foreach name = ["read_imagei"] in {
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, Float, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, Float, Float, Float], Attr.Pure>;
      }
      foreach name = ["read_imageui"] in {
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, Float, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, Float, Float, Float], Attr.Pure>;
      }
    }
    foreach imgTy = [Image3d] in {
      foreach name = ["read_imagef"] in {
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, VectorType<Float, 4>, VectorType<Float, 4>], Attr.Pure>;
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, Float], Attr.Pure>;
      }
      foreach name = ["read_imagei"] in {
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, VectorType<Float, 4>, VectorType<Float, 4>], Attr.Pure>;
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, Float], Attr.Pure>;
      }
      foreach name = ["read_imageui"] in {
```

- **L1551**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1552**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1553**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1554**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1555**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1556**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1557**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1558**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1559**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1560**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1561**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1562**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1563**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1564**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1565**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1566**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1567**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1568**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1569**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1570**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1571**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1572**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1573**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1574**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1575**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1576-1600 / 第 1576-1600 行

```tablegen
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, VectorType<Float, 4>, VectorType<Float, 4>], Attr.Pure>;
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, Float], Attr.Pure>;
      }
    }
    foreach imgTy = [Image1dArray] in {
      foreach name = ["read_imagef"] in {
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float, Float], Attr.Pure>;
      }
      foreach name = ["read_imagei"] in {
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float, Float], Attr.Pure>;
      }
      foreach name = ["read_imageui"] in {
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 2>, Float, Float], Attr.Pure>;
      }
    }
    foreach imgTy = [Image2dArray] in {
      foreach name = ["read_imagef"] in {
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<Float, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, VectorType<Float, 2>, VectorType<Float, 2>], Attr.Pure>;
      }
      foreach name = ["read_imagei"] in {
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, Float], Attr.Pure>;
```

- **L1576**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1577**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1578**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1579**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1580**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1581**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1582**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1583**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1584**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1585**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1586**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1587**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1588**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1589**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1590**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1591**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1592**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1593**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1594**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1595**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1596**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1597**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1598**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1599**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1600**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1601-1625 / 第 1601-1625 行

```tablegen
        def : Builtin<name, [VectorType<Int, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, VectorType<Float, 2>, VectorType<Float, 2>], Attr.Pure>;
      }
      foreach name = ["read_imageui"] in {
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, Float], Attr.Pure>;
        def : Builtin<name, [VectorType<UInt, 4>, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, VectorType<Float, 2>, VectorType<Float, 2>], Attr.Pure>;
      }
    }
    foreach imgTy = [Image2dArrayDepth] in {
      foreach name = ["read_imagef"] in {
        def : Builtin<name, [Float, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, Float], Attr.Pure>;
        def : Builtin<name, [Float, ImageType<imgTy, aQual>, Sampler, VectorType<Float, 4>, VectorType<Float, 2>, VectorType<Float, 2>], Attr.Pure>;
      }
    }
  }
}

// Added to section 6.13.14.5
multiclass ImageQueryNumMipLevels<string aQual> {
  foreach imgTy = [Image1d, Image2d, Image3d, Image1dArray, Image2dArray, Image2dDepth, Image2dArrayDepth] in {
    def : Builtin<"get_image_num_mip_levels", [Int, ImageType<imgTy, aQual>]>;
  }
}

let Extension = FuncExtKhrMipmapImage in {
  defm : ImageQueryNumMipLevels<"RO">;
```

- **L1601**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1602**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1603**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1604**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1605**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1606**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1607**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1608**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1609**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1610**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1611**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1612**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1613**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1614**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1615**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1616**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1617**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1618**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1619**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1620**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1621**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1622**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1623**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1624**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1625**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1626-1650 / 第 1626-1650 行

```tablegen
  defm : ImageQueryNumMipLevels<"WO">;
  defm : ImageQueryNumMipLevels<"RW">;
}

// Write functions are enabled using a separate extension.
let Extension = FuncExtKhrMipmapImageWrites in {
  // Added to section 6.13.14.4.
  foreach aQual = ["WO", "RW"] in {
    foreach imgTy = [Image2d] in {
      def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, Int, VectorType<Float, 4>]>;
      def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, Int, VectorType<Int, 4>]>;
      def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, Int, VectorType<UInt, 4>]>;
    }
    def : Builtin<"write_imagef", [Void, ImageType<Image2dDepth, aQual>, VectorType<Int, 2>, Int, Float]>;
    foreach imgTy = [Image1d] in {
      def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, Int, Int, VectorType<Float, 4>]>;
      def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, Int, Int, VectorType<Int, 4>]>;
      def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, Int, Int, VectorType<UInt, 4>]>;
    }
    foreach imgTy = [Image1dArray] in {
      def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, Int, VectorType<Float, 4>]>;
      def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, Int, VectorType<Int, 4>]>;
      def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, VectorType<Int, 2>, Int, VectorType<UInt, 4>]>;
    }
    foreach imgTy = [Image2dArray] in {
```

- **L1626**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1627**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1628**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1629**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1630**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1631**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1632**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1633**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1634**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1635**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1636**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1637**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1638**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1639**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1640**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1641**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1642**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1643**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1644**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1645**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1646**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1647**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1648**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1649**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1650**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1651-1675 / 第 1651-1675 行

```tablegen
      def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, Int, VectorType<Float, 4>]>;
      def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, Int, VectorType<Int, 4>]>;
      def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, Int, VectorType<UInt, 4>]>;
    }
    def : Builtin<"write_imagef", [Void, ImageType<Image2dArrayDepth, aQual>, VectorType<Int, 4>, Int, Float]>;
    foreach imgTy = [Image3d] in {
      def : Builtin<"write_imagef", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, Int, VectorType<Float, 4>]>;
      def : Builtin<"write_imagei", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, Int, VectorType<Int, 4>]>;
      def : Builtin<"write_imageui", [Void, ImageType<imgTy, aQual>, VectorType<Int, 4>, Int, VectorType<UInt, 4>]>;
    }
  }
}

//--------------------------------------------------------------------
// OpenCL Extension v2.0 s18.3 - Creating OpenCL Memory Objects from OpenGL MSAA Textures
// --- Table 6.13.14.3 ---
multiclass ImageReadMsaa<string aQual> {
  foreach imgTy = [Image2dMsaa] in {
    def : Builtin<"read_imagef", [VectorType<Float, 4>, ImageType<imgTy, aQual>, VectorType<Int, 2>, Int], Attr.Pure>;
    def : Builtin<"read_imagei", [VectorType<Int, 4>, ImageType<imgTy, aQual>, VectorType<Int, 2>, Int], Attr.Pure>;
    def : Builtin<"read_imageui", [VectorType<UInt, 4>, ImageType<imgTy, aQual>, VectorType<Int, 2>, Int], Attr.Pure>;
  }
  foreach imgTy = [Image2dArrayMsaa] in {
    def : Builtin<"read_imagef", [VectorType<Float, 4>, ImageType<imgTy, aQual>, VectorType<Int, 4>, Int], Attr.Pure>;
    def : Builtin<"read_imagei", [VectorType<Int, 4>, ImageType<imgTy, aQual>, VectorType<Int, 4>, Int], Attr.Pure>;
```

- **L1651**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1652**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1653**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1654**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1655**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1656**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1657**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1658**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1659**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1660**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1661**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1662**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1663**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1664**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1665**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1666**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1667**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1668**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1669**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1670**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1671**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1672**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1673**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1674**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1675**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1676-1700 / 第 1676-1700 行

```tablegen
    def : Builtin<"read_imageui", [VectorType<UInt, 4>, ImageType<imgTy, aQual>, VectorType<Int, 4>, Int], Attr.Pure>;
  }
  foreach name = ["read_imagef"] in {
    def : Builtin<name, [Float, ImageType<Image2dMsaaDepth, aQual>, VectorType<Int, 2>, Int], Attr.Pure>;
    def : Builtin<name, [Float, ImageType<Image2dArrayMsaaDepth, aQual>, VectorType<Int, 4>, Int], Attr.Pure>;
  }
}

// --- Table 6.13.14.5 ---
multiclass ImageQueryMsaa<string aQual> {
  foreach imgTy = [Image2dMsaa, Image2dArrayMsaa, Image2dMsaaDepth, Image2dArrayMsaaDepth] in {
    foreach name = ["get_image_width", "get_image_height",
                    "get_image_channel_data_type", "get_image_channel_order",
                    "get_image_num_samples"] in {
      def : Builtin<name, [Int, ImageType<imgTy, aQual>], Attr.Const>;
    }
    def : Builtin<"get_image_dim", [VectorType<Int, 2>, ImageType<imgTy, aQual>], Attr.Const>;
  }
  foreach imgTy = [Image2dArrayMsaa, Image2dArrayMsaaDepth] in {
    def : Builtin<"get_image_array_size", [Size, ImageType<imgTy, aQual>], Attr.Const>;
  }
}

let Extension = FuncExtKhrGlMsaaSharing in {
  defm : ImageReadMsaa<"RO">;
```

- **L1676**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1677**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1678**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1679**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1680**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1681**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1682**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1683**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1684**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1685**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1686**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1687**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1688**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1689**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1690**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1691**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1692**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1693**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1694**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1695**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1696**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1697**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1698**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1699**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1700**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1701-1725 / 第 1701-1725 行

```tablegen
  defm : ImageQueryMsaa<"RO">;
  defm : ImageQueryMsaa<"WO">;
  defm : ImageReadMsaa<"RW">;
  defm : ImageQueryMsaa<"RW">;
}

//--------------------------------------------------------------------
// OpenCL Extension v2.0 s28 - Subgroups
// --- Table 28.2.1 ---
let Extension = FuncExtKhrSubgroups in {
  foreach name = ["get_sub_group_size", "get_max_sub_group_size",
                  "get_num_sub_groups", "get_sub_group_id",
                  "get_sub_group_local_id"] in {
    def : Builtin<name, [UInt]>;
  }
  let MinVersion = CL20 in {
    foreach name = ["get_enqueued_num_sub_groups"] in {
      def : Builtin<name, [UInt]>;
    }
  }
}

// --- Table 28.2.2 ---
let Extension = FuncExtKhrSubgroups in {
  def : Builtin<"sub_group_barrier", [Void, MemFenceFlags], Attr.Convergent>;
```

- **L1701**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1702**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1703**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1704**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1705**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1706**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1707**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1708**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1709**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1710**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1711**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1712**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1713**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1714**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1715**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1716**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1717**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1718**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1719**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1720**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1721**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1722**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1723**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1724**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1725**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1726-1750 / 第 1726-1750 行

```tablegen
  let MinVersion = CL20 in {
    def : Builtin<"sub_group_barrier", [Void, MemFenceFlags, MemoryScope], Attr.Convergent>;
  }
}

// --- Table 28.2.4 ---
let Extension = FuncExtKhrSubgroups in {
  foreach name = ["sub_group_all", "sub_group_any"] in {
    def : Builtin<name, [Int, Int], Attr.Convergent>;
  }
  foreach name = ["sub_group_broadcast"] in {
    def : Builtin<name, [IntLongFloatGenType1, IntLongFloatGenType1, UInt], Attr.Convergent>;
  }
  foreach name = ["sub_group_reduce_", "sub_group_scan_exclusive_",
                  "sub_group_scan_inclusive_"] in {
    foreach op = ["add", "min", "max"] in {
      def : Builtin<name # op, [IntLongFloatGenType1, IntLongFloatGenType1], Attr.Convergent>;
    }
  }
}

// OpenCL Extension v3.0 s38 - Extended Subgroup Functions

// Section 38.4.1 - cl_khr_subgroup_extended_types
let Extension = FuncExtKhrSubgroupExtendedTypes in {
```

- **L1726**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1727**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1728**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1729**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1730**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1731**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1732**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1733**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1734**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1735**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1736**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1737**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1738**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1739**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1740**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1741**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1742**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1743**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1744**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1745**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1746**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1747**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1748**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1749**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1750**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1751-1775 / 第 1751-1775 行

```tablegen
  // For sub_group_broadcast, add scalar char, uchar, short, and ushort support,
  def : Builtin<"sub_group_broadcast", [CharShortGenType1, CharShortGenType1, UInt], Attr.Convergent>;
  // gentype may additionally be one of the supported built-in vector data types.
  def : Builtin<"sub_group_broadcast", [AGenTypeNNoScalar, AGenTypeNNoScalar, UInt], Attr.Convergent>;

  foreach name = ["sub_group_reduce_", "sub_group_scan_exclusive_",
                  "sub_group_scan_inclusive_"] in {
    foreach op = ["add", "min", "max"] in {
      def : Builtin<name # op, [CharShortGenType1, CharShortGenType1], Attr.Convergent>;
    }
  }
}

// Section 38.5.1 - cl_khr_subgroup_non_uniform_vote
let Extension = FuncExtKhrSubgroupNonUniformVote in {
  def : Builtin<"sub_group_elect", [Int]>;
  def : Builtin<"sub_group_non_uniform_all", [Int, Int]>;
  def : Builtin<"sub_group_non_uniform_any", [Int, Int]>;
  def : Builtin<"sub_group_non_uniform_all_equal", [Int, AGenType1]>;
}

// Section 38.6.1 - cl_khr_subgroup_ballot
let Extension = FuncExtKhrSubgroupBallot in {
  def : Builtin<"sub_group_non_uniform_broadcast", [AGenTypeN, AGenTypeN, UInt]>;
  def : Builtin<"sub_group_broadcast_first", [AGenType1, AGenType1]>;
```

- **L1751**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1752**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1753**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1754**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1755**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1756**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L1757**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1758**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1759**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1760**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1761**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1762**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1763**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1764**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1765**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1766**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1767**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1768**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1769**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1770**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1771**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1772**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1773**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1774**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1775**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1776-1800 / 第 1776-1800 行

```tablegen
  def : Builtin<"sub_group_ballot", [VectorType<UInt, 4>, Int]>;
  def : Builtin<"sub_group_inverse_ballot", [Int, VectorType<UInt, 4>], Attr.Const>;
  def : Builtin<"sub_group_ballot_bit_extract", [Int, VectorType<UInt, 4>, UInt], Attr.Const>;
  def : Builtin<"sub_group_ballot_bit_count", [UInt, VectorType<UInt, 4>], Attr.Const>;
  def : Builtin<"sub_group_ballot_inclusive_scan", [UInt, VectorType<UInt, 4>]>;
  def : Builtin<"sub_group_ballot_exclusive_scan", [UInt, VectorType<UInt, 4>]>;
  def : Builtin<"sub_group_ballot_find_lsb", [UInt, VectorType<UInt, 4>]>;
  def : Builtin<"sub_group_ballot_find_msb", [UInt, VectorType<UInt, 4>]>;

  foreach op = ["eq", "ge", "gt", "le", "lt"] in {
    def : Builtin<"get_sub_group_" # op # "_mask", [VectorType<UInt, 4>], Attr.Const>;
  }
}

// Section 38.7.1 - cl_khr_subgroup_non_uniform_arithmetic
let Extension = FuncExtKhrSubgroupNonUniformArithmetic in {
  foreach name = ["reduce_", "scan_exclusive_", "scan_inclusive_"] in {
    foreach op = ["add", "min", "max", "mul"] in {
      def : Builtin<"sub_group_non_uniform_" # name # op, [AGenType1, AGenType1]>;
    }
    foreach op = ["and", "or", "xor"] in {
      def : Builtin<"sub_group_non_uniform_" # name # op, [AIGenType1, AIGenType1]>;
    }
    foreach op = ["and", "or", "xor"] in {
      def : Builtin<"sub_group_non_uniform_" # name # "logical_" # op, [Int, Int]>;
```

- **L1776**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1777**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1778**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1779**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1780**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1781**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1782**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1783**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1784**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1785**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1786**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1787**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1788**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1789**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1790**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1791**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1792**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1793**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1794**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1795**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1796**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1797**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1798**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1799**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1800**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1801-1825 / 第 1801-1825 行

```tablegen
    }
  }
}

// Section 38.8.1 - cl_khr_subgroup_shuffle
let Extension = FuncExtKhrSubgroupShuffle in {
  def : Builtin<"sub_group_shuffle", [AGenType1, AGenType1, UInt]>;
  def : Builtin<"sub_group_shuffle_xor", [AGenType1, AGenType1, UInt]>;
}

// Section 38.9.1 - cl_khr_subgroup_shuffle_relative
let Extension = FuncExtKhrSubgroupShuffleRelative in {
  def : Builtin<"sub_group_shuffle_up", [AGenType1, AGenType1, UInt]>;
  def : Builtin<"sub_group_shuffle_down", [AGenType1, AGenType1, UInt]>;
}

// Section 38.10.1 - cl_khr_subgroup_clustered_reduce
let Extension = FuncExtKhrSubgroupClusteredReduce in {
  foreach op = ["add", "min", "max", "mul"] in {
    def : Builtin<"sub_group_clustered_reduce_" # op, [AGenType1, AGenType1, UInt]>;
  }
  foreach op = ["and", "or", "xor"] in {
    def : Builtin<"sub_group_clustered_reduce_" # op, [AIGenType1, AIGenType1, UInt]>;
  }
  foreach op = ["and", "or", "xor"] in {
```

- **L1801**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1802**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1803**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1804**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1805**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1806**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1807**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1808**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1809**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1810**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1811**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1812**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1813**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1814**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1815**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1816**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1817**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1818**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1819**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1820**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1821**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1822**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1823**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1824**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1825**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1826-1850 / 第 1826-1850 行

```tablegen
    def : Builtin<"sub_group_clustered_reduce_logical_" # op, [Int, Int, UInt]>;
  }
}

// Section 40.3.1 - cl_khr_extended_bit_ops
let Extension = FuncExtKhrExtendedBitOps in {
  def : Builtin<"bitfield_insert", [AIGenTypeN, AIGenTypeN, AIGenTypeN, UInt, UInt], Attr.Const>;
  def : Builtin<"bitfield_extract_signed", [SGenTypeN, SGenTypeN, UInt, UInt], Attr.Const>;
  def : Builtin<"bitfield_extract_signed", [SGenTypeN, UGenTypeN, UInt, UInt], Attr.Const>;
  def : Builtin<"bitfield_extract_unsigned", [UGenTypeN, SGenTypeN, UInt, UInt], Attr.Const>;
  def : Builtin<"bitfield_extract_unsigned", [UGenTypeN, UGenTypeN, UInt, UInt], Attr.Const>;
  def : Builtin<"bit_reverse", [AIGenTypeN, AIGenTypeN], Attr.Const>;
}

// Section 42.3 - cl_khr_integer_dot_product
let Extension = FunctionExtension<"__opencl_c_integer_dot_product_input_4x8bit"> in {
  def : Builtin<"dot", [UInt, VectorType<UChar, 4>, VectorType<UChar, 4>], Attr.Const>;
  def : Builtin<"dot", [Int, VectorType<Char, 4>, VectorType<Char, 4>], Attr.Const>;
  def : Builtin<"dot", [Int, VectorType<UChar, 4>, VectorType<Char, 4>], Attr.Const>;
  def : Builtin<"dot", [Int, VectorType<Char, 4>, VectorType<UChar, 4>], Attr.Const>;

  def : Builtin<"dot_acc_sat", [UInt, VectorType<UChar, 4>, VectorType<UChar, 4>, UInt], Attr.Const>;
  def : Builtin<"dot_acc_sat", [Int, VectorType<Char, 4>, VectorType<Char, 4>, Int], Attr.Const>;
  def : Builtin<"dot_acc_sat", [Int, VectorType<UChar, 4>, VectorType<Char, 4>, Int], Attr.Const>;
  def : Builtin<"dot_acc_sat", [Int, VectorType<Char, 4>, VectorType<UChar, 4>, Int], Attr.Const>;
```

- **L1826**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1827**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1828**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1829**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1830**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1831**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1832**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1833**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1834**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1835**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1836**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1837**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1838**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1839**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1840**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1841**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1842**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1843**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1844**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1845**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1846**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1847**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1848**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1849**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1850**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 1851-1875 / 第 1851-1875 行

```tablegen
}

let Extension = FunctionExtension<"__opencl_c_integer_dot_product_input_4x8bit_packed"> in {
  def : Builtin<"dot_4x8packed_uu_uint", [UInt, UInt, UInt], Attr.Const>;
  def : Builtin<"dot_4x8packed_ss_int", [Int, UInt, UInt], Attr.Const>;
  def : Builtin<"dot_4x8packed_us_int", [Int, UInt, UInt], Attr.Const>;
  def : Builtin<"dot_4x8packed_su_int", [Int, UInt, UInt], Attr.Const>;

  def : Builtin<"dot_acc_sat_4x8packed_uu_uint", [UInt, UInt, UInt, UInt], Attr.Const>;
  def : Builtin<"dot_acc_sat_4x8packed_ss_int", [Int, UInt, UInt, Int], Attr.Const>;
  def : Builtin<"dot_acc_sat_4x8packed_us_int", [Int, UInt, UInt, Int], Attr.Const>;
  def : Builtin<"dot_acc_sat_4x8packed_su_int", [Int, UInt, UInt, Int], Attr.Const>;
}

// Section 48.3 - cl_khr_subgroup_rotate
let Extension = FunctionExtension<"cl_khr_subgroup_rotate"> in {
  def : Builtin<"sub_group_rotate", [AGenType1, AGenType1, Int], Attr.Convergent>;
  def : Builtin<"sub_group_clustered_rotate", [AGenType1, AGenType1, Int, UInt], Attr.Convergent>;
}

// cl_khr_kernel_clock
let Extension = FunctionExtension<"cl_khr_kernel_clock __opencl_c_kernel_clock_scope_device"> in {
  def : Builtin<"clock_read_device", [ULong]>;
  def : Builtin<"clock_read_hilo_device", [VectorType<UInt, 2>]>;
}
```

- **L1851**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1852**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1853**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1854**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1855**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1856**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1857**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1858**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1859**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1860**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1861**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1862**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1863**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1864**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1865**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1866**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1867**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1868**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1869**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1870**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1871**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1872**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1873**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1874**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1875**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 1876-1900 / 第 1876-1900 行

```tablegen
let Extension = FunctionExtension<"cl_khr_kernel_clock __opencl_c_kernel_clock_scope_work_group"> in {
  def : Builtin<"clock_read_work_group", [ULong]>;
  def : Builtin<"clock_read_hilo_work_group", [VectorType<UInt, 2>]>;
}
let Extension = FunctionExtension<"cl_khr_kernel_clock __opencl_c_kernel_clock_scope_sub_group"> in {
  def : Builtin<"clock_read_sub_group", [ULong]>;
  def : Builtin<"clock_read_hilo_sub_group", [VectorType<UInt, 2>]>;
}

//--------------------------------------------------------------------
// Arm extensions.
let Extension = ArmIntegerDotProductInt8 in {
  foreach name = ["arm_dot"] in {
    def : Builtin<name, [UInt, VectorType<UChar, 4>, VectorType<UChar, 4>]>;
    def : Builtin<name, [Int, VectorType<Char, 4>, VectorType<Char, 4>]>;
  }
}
let Extension = ArmIntegerDotProductAccumulateInt8 in {
  foreach name = ["arm_dot_acc"] in {
    def : Builtin<name, [UInt, VectorType<UChar, 4>, VectorType<UChar, 4>, UInt]>;
    def : Builtin<name, [Int, VectorType<Char, 4>, VectorType<Char, 4>, Int]>;
  }
}
let Extension = ArmIntegerDotProductAccumulateInt16 in {
  foreach name = ["arm_dot_acc"] in {
```

- **L1876**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1877**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1878**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1879**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1880**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1881**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1882**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1883**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1884**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1885**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1886**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1887**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1888**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1889**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1890**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1891**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1892**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1893**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1894**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1895**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1896**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1897**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1898**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1899**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1900**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1901-1925 / 第 1901-1925 行

```tablegen
    def : Builtin<name, [UInt, VectorType<UShort, 2>, VectorType<UShort, 2>, UInt]>;
    def : Builtin<name, [Int, VectorType<Short, 2>, VectorType<Short, 2>, Int]>;
  }
}
let Extension = ArmIntegerDotProductAccumulateSaturateInt8 in {
  foreach name = ["arm_dot_acc_sat"] in {
    def : Builtin<name, [UInt, VectorType<UChar, 4>, VectorType<UChar, 4>, UInt]>;
    def : Builtin<name, [Int, VectorType<Char, 4>, VectorType<Char, 4>, Int]>;
  }
}

//--------------------------------------------------------------------
// AMD extensions.
let Extension = AMDMediaOps in {
  foreach name = ["amd_bitalign", "amd_bytealign", "amd_lerp", "amd_sadhi", "amd_sad"] in {
    def : Builtin<name, [GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar]>;
  }
  foreach name = ["amd_unpack0", "amd_unpack1", "amd_unpack2", "amd_unpack3"] in {
    def : Builtin<name, [GenTypeFloatVecAndScalar, GenTypeUIntVecAndScalar]>;
  }
  def : Builtin<"amd_pack", [UInt, VectorType<Float, 4>]>;
  def : Builtin<"amd_sad4", [UInt, VectorType<UInt, 4>, VectorType<UInt, 4>, UInt]>;
}

let Extension = AMDMediaOps2 in {
```

- **L1901**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1902**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1903**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1904**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1905**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1906**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1907**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1908**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1909**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1910**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1911**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1912**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1913**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L1914**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1915**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1916**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1917**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1918**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1919**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1920**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1921**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1922**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1923**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1924**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L1925**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。

### Lines 1926-1938 / 第 1926-1938 行

```tablegen
  def : Builtin<"amd_bfe", [GenTypeIntVecAndScalar, GenTypeIntVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar]>;
  foreach name = ["amd_bfe", "amd_msad", "amd_sadd", "amd_sadw"] in {
    def : Builtin<name, [GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar]>;
  }
  def : Builtin<"amd_bfm", [GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar, GenTypeUIntVecAndScalar]>;
  foreach name = ["amd_max3", "amd_median3", "amd_min3"] in {
    foreach gentype = [GenTypeFloatVecAndScalar, GenTypeIntVecAndScalar, GenTypeUIntVecAndScalar] in {
      def : Builtin<name, [gentype, gentype, gentype, gentype]>;
    }
  }
  def : Builtin<"amd_mqsad", [GenTypeULongVecAndScalar, GenTypeULongVecAndScalar, GenTypeUIntVecAndScalar, GenTypeULongVecAndScalar]>;
  def : Builtin<"amd_qsad", [GenTypeULongVecAndScalar, GenTypeULongVecAndScalar, GenTypeUIntVecAndScalar, GenTypeULongVecAndScalar]>;
}
```

- **L1926**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1927**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1928**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1929**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1930**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1931**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1932**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L1933**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1934**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1935**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L1936**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1937**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L1938**: Closes the current scope or body. / 关闭当前作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: This file is a declaration unit in Clang's **Sema** subsystem. / 该文件是 Clang **Sema** 子系统中的声明单元。
- **Scale / 规模**: 1938 lines and 0 direct includes. / 共 1938 行，并直接包含 0 个头文件。
- **Subsystem focus / 子系统关注点**: type checking, name lookup, semantic constraint enforcement. / 类型检查、名字查找、语义约束执行。
- **Primary types / 主要类型**: `Version`, `AddressSpace`, `AbstractExtension`, `FunctionExtension`, `TypeExtension`, `concatExtension`, `QualType`, `IntList`. / 主要类型包括 `Version`、`AddressSpace`、`AbstractExtension`、`FunctionExtension`、`TypeExtension`、`concatExtension`、`QualType`、`IntList`。
- **Visible entry points / 关键入口**: `cast<FunctionExtension>`. / 可见的关键入口包括 `cast<FunctionExtension>`。

## Dependencies / 依赖关系

- **Core types / 核心类型**: `Version`, `AddressSpace`, `AbstractExtension`, `FunctionExtension`, `TypeExtension`, `concatExtension`, `QualType`, `IntList`, `Type`, `VectorType`.
- **Referenced routines / 关键例程**: `cast<FunctionExtension>`.
