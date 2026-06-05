# GPUOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/GPU/IR/GPUOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines some operations of the GPU dialect.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/GPU/IR`，围绕 GPU 方言公开 `GPUOps` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- GPUOps.td - GPU dialect operation definitions ------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Defines some operations of the GPU dialect.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 12-31
```tablegen

#ifndef GPU_OPS
#define GPU_OPS

include "mlir/Dialect/DLTI/DLTIBase.td"
include "mlir/Dialect/GPU/IR/GPUBase.td"
include "mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td"
include "mlir/Dialect/GPU/IR/CompilationAttrs.td"
include "mlir/Dialect/GPU/IR/GPUDeviceMappingAttr.td"
include "mlir/Dialect/GPU/IR/ParallelLoopMapperAttr.td"
include "mlir/IR/CommonTypeConstraints.td"
include "mlir/IR/EnumAttr.td"
include "mlir/IR/RegionKindInterface.td"
include "mlir/IR/SymbolInterfaces.td"
include "mlir/Interfaces/ControlFlowInterfaces.td"
include "mlir/Interfaces/DataLayoutInterfaces.td"
include "mlir/IR/OpAsmInterface.td"
include "mlir/Interfaces/FunctionInterfaces.td"
include "mlir/Interfaces/InferIntRangeInterface.td"
include "mlir/Interfaces/InferTypeOpInterface.td"
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 32-41
```tablegen
include "mlir/Interfaces/SideEffectInterfaces.td"

//===----------------------------------------------------------------------===//
// GPU Dialect operations.
//===----------------------------------------------------------------------===//

class GPU_Op<string mnemonic, list<Trait> traits = []> :
    Op<GPU_Dialect, mnemonic, traits>;

class GPU_IndexOp<string mnemonic, list<Trait> traits = []> :
```
- **EN**: Introduces declarations for `GPU_Op`, `GPU_IndexOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_Op`, `GPU_IndexOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 42-61
```tablegen
    GPU_Op<mnemonic, !listconcat(traits, [
        Pure,
        DeclareOpInterfaceMethods<InferIntRangeInterface, ["inferResultRanges"]>,
        DeclareOpInterfaceMethods<OpAsmOpInterface, ["getAsmResultNames"]>])>,
    Arguments<(ins GPU_DimensionAttr:$dimension,
                   OptionalAttr<IndexAttr>:$upper_bound)>, Results<(outs Index)> {
  let assemblyFormat = "$dimension (`upper_bound` $upper_bound^)? attr-dict";
  let extraClassDefinition = [{
    void $cppClass::getAsmResultNames(
        llvm::function_ref<void(mlir::Value, mlir::StringRef)> setNameFn) {
      auto dimStr = stringifyDimension(getDimensionAttr().getValue());
      auto opName = getOperationName();
      opName.consume_front("gpu.");
      SmallString<8> resultName({opName, "_", dimStr});
      setNameFn(getResult(),resultName);
    }
  }];
  let builders = [
    OpBuilder<(ins "::mlir::gpu::Dimension":$dimension), [{
      build($_builder, $_state, dimension, /*upperBound=*/nullptr);
```
- **EN**: Implements logic around `listconcat`, `Arguments`, `Results`, `dimension`, and 9 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `listconcat`, `Arguments`, `Results`, `dimension`, and 9 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 62-73
```tablegen
    }]>,
    OpBuilder<(ins "::mlir::Type":$resultType, "::mlir::gpu::Dimension":$dimension), [{
      build($_builder, $_state, resultType, dimension, /*upperBound=*/nullptr);
    }]>
  ];
}

def GPU_ClusterDimOp : GPU_IndexOp<"cluster_dim"> {
  let description = [{
    Returns the number of cluster identifiers per grid along
    the x, y, or z `dimension`.

```
- **EN**: Introduces declarations for `GPU_ClusterDimOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ClusterDimOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 74-83
```tablegen
    Example:

    ```mlir
    %cDimX = gpu.cluster_dim x
    ```

    If `upper_bound` is set, then executing (a lowering of) this operation in an
    environment where the clusters per grid is greater than `upper_bound` causes
    undefined behavior.

```
- **EN**: Implements logic around `executing`; this block coordinates dialect conversion or lowering decisions; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `executing` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并涉及目标平台或加速器专用语义。

### Lines 84-94
```tablegen
    There is an implicit upper bound of `kMaxDim` (currently uint32_t::max).
  }];
}

def GPU_ClusterDimBlocksOp : GPU_IndexOp<"cluster_dim_blocks"> {
  let description = [{
    Returns the number of thread blocks in the cluster along
    the x, y, or z `dimension`.

    Example:

```
- **EN**: Introduces declarations for `GPU_ClusterDimBlocksOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ClusterDimBlocksOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 95-106
```tablegen
    ```mlir
    %cDimBlocksX = gpu.cluster_dim_blocks x
    ```

    If `upper_bound` is set, then executing (a lowering of) this operation in an
    environment where the thread blocks per cluster  is greater than `upper_bound`
    causes undefined behavior.

    There is an implicit upper bound of `kMaxClusterDim` (currently 8).
  }];
}

```
- **EN**: Declares APIs or declarative rules around `executing`; this block coordinates dialect conversion or lowering decisions; touches target- or accelerator-specific semantics.
- **CN**: 声明与 `executing` 相关的 API 或声明式规则；该代码块协调方言转换或 lowering 决策，并涉及目标平台或加速器专用语义。

### Lines 107-117
```tablegen
def GPU_ClusterIdOp : GPU_IndexOp<"cluster_id"> {
  let description = [{
    Returns the cluster id, i.e. the index of the current cluster within the
    grid along the x, y, or z `dimension`.

    Example:

    ```mlir
    %cIdY = gpu.cluster_id y
    ```

```
- **EN**: Introduces declarations for `GPU_ClusterIdOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ClusterIdOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 118-129
```tablegen
    If `upper_bound` is set, then executing (a lowering of) this operation in an
    environment where the number of clusters in the grid along `dimension` is
    greater than `upper_bound` causes undefined behavior.

    There is an implicit upper bound of `kMaxDim` (currently uint32_t::max).
  }];
}

def GPU_ClusterBlockIdOp : GPU_IndexOp<"cluster_block_id"> {
  let description = [{
    Returns the block id within the cluster along the x, y, or z `dimension`.

```
- **EN**: Introduces declarations for `GPU_ClusterBlockIdOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ClusterBlockIdOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 130-139
```tablegen
    Example:

    ```mlir
    %cBlockIdY = gpu.cluster_block_id y
    ```

    If `upper_bound` is set, then executing (a lowering of) this operation in an
    environment where the number of thread blocks per cluster along `dimension`
    is greater than `upper_bound` causes undefined behavior.

```
- **EN**: Implements logic around `executing`; this block coordinates dialect conversion or lowering decisions; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `executing` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并涉及目标平台或加速器专用语义。

### Lines 140-150
```tablegen
    There is an implicit upper bound of `kMaxClusterDim` (currently 8).
  }];
}

def GPU_BlockDimOp : GPU_IndexOp<"block_dim"> {
  let description = [{
    Returns the number of threads in the thread block (aka the block size) along
    the x, y, or z `dimension`.

    Example:

```
- **EN**: Introduces declarations for `GPU_BlockDimOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_BlockDimOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 151-162
```tablegen
    ```mlir
    %bDimX = gpu.block_dim x
    ```

    If `known_block_size` is set on an this operation's enclosing `gpu.func`,
    or `gpu.known_block_size` is set on an enclosing `FunctionOpInterface`
    implementor, or if the enclosing `gpu.launch` specifies a constant size for
    `dimension`'s blocks, these contextual facts may be used to infer that this
    operation has a constant value, though such a transformation will not be
    performed by canonicalization or the default constant folder. Executions which
    cause that constant-value assumption to be false incur undefined behavior.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 163-173
```tablegen
    If `upper_bound` is set, executions where the bblock size along `dimension`
    exceeds `upper_bound` cause undefined behavior.

    There is an implicit upper bound of `kMaxDim` (currently uint32_t::max).
  }];
}
def GPU_BlockIdOp : GPU_IndexOp<"block_id"> {
  let description = [{
    Returns the block id, i.e. the index of the current block within the grid
    along the x, y, or z `dimension`.

```
- **EN**: Introduces declarations for `GPU_BlockIdOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_BlockIdOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 174-184
```tablegen
    Example:

    ```mlir
    %bIdY = gpu.block_id y
    ```

    If `upper_bound` is set, or if one can be inferred from `known_grid_size`-type
    annotations in context, executions where the block index in `dimension` would
    be greater than or equal to that bound cause undefined behavior. `upper_bound`
    takes priority over bounds inferrable from context.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 185-194
```tablegen
    There is an implicit upper bound of `kMaxDim` (currently uint32_t::max).
  }];
}
def GPU_GridDimOp : GPU_IndexOp<"grid_dim"> {
  let description = [{
    Returns the number of thread blocks in the grid along the x, y, or z
    `dimension`.

    Example:

```
- **EN**: Introduces declarations for `GPU_GridDimOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_GridDimOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 195-207
```tablegen
    ```mlir
    %gDimZ = gpu.grid_dim z
    ```


    If `known_grid_size` is set on an this operation's enclosing `gpu.func`,
    or `gpu.known_grid_size` is set on an enclosing `FunctionOpInterface`
    implementor, or if the enclosing `gpu.launch` specifies a constant size for
    `dimension`'s grid length, these contextual facts may be used to infer that this
    operation has a constant value, though such a transformation will not be
    performed by canonicalization or the default constant folder. Executions which
    cause that constant-value assumption to be false incur undefined behavior.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 208-218
```tablegen
    If `upper_bound` is set, executions where the grid size in `dimension` would
    exceed `upper_bound` cause undefined behavior.

    There is an implicit upper bound of `kMaxDim` (currently uint32_t::max).
  }];
}
def GPU_ThreadIdOp : GPU_IndexOp<"thread_id"> {
  let description = [{
    Returns the thread id, i.e. the index of the current thread within the block
    along the x, y, or z `dimension`.

```
- **EN**: Introduces declarations for `GPU_ThreadIdOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ThreadIdOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 219-228
```tablegen
    Example:

    ```mlir
    %tIdX = gpu.thread_id x
    ```

    If `upper_bound` is set, or if one can be inferred from `known_block_size`-type
    annotations in context, executions where the thread index would be greater
    than or equal to that bound cause undefined behavior.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 229-242
```tablegen
    There is an implicit upper bound of `kMaxDim` (currently uint32_t::max).
  }];
}

def GPU_LaneIdOp : GPU_Op<"lane_id", [
      Pure, DeclareOpInterfaceMethods<InferIntRangeInterface, ["inferResultRanges"]>]> {
  let description = [{
    Returns the lane id within the subgroup (warp/wave).

    Example:
    ```mlir
    %laneId = gpu.lane_id
    ```

```
- **EN**: Introduces declarations for `GPU_LaneIdOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_LaneIdOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 243-252
```tablegen
    If `upper_bound` is set, executions with more than `upper_bound` lanes per
    subgroup cause undefined behavior. In the abscence of `upper_bound`,
    the lane id is still assumed to be non-negative and less than the
    target-independent `kMaxSubgroupSize` (currently 128).
  }];
  let arguments = (ins OptionalAttr<IndexAttr>:$upper_bound);
  let results = (outs Index:$result);
  let assemblyFormat = "(`upper_bound` $upper_bound^)? attr-dict";
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 253-262
```tablegen
def GPU_SubgroupIdOp : GPU_Op<"subgroup_id", [
      Pure, DeclareOpInterfaceMethods<InferIntRangeInterface, ["inferResultRanges"]>]>,
    Arguments<(ins OptionalAttr<IndexAttr>:$upper_bound)>,
    Results<(outs Index:$result)> {
  let description = [{
    Returns the subgroup id, i.e., the index of the current subgroup within the
    workgroup.

    Example:

```
- **EN**: Introduces declarations for `GPU_SubgroupIdOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupIdOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 263-274
```tablegen
    ```mlir
    %sgId = gpu.subgroup_id : index
    ```

    Executions where there are more than `upper_bound` subgroups per workgroup
    cause undefined behavior. There is an implicit upper bound of `kMaxDim`
    (currently uint32_t::max).
  }];

  let assemblyFormat = "(`upper_bound` $upper_bound^)? attr-dict `:` type($result)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 275-287
```tablegen
def GPU_GlobalIdOp : GPU_IndexOp<"global_id"> {
  let description = [{
    Returns the unique global workitem/thread id, i.e., the unique index of the
    current workitem/thread within all workgroups / grid along the x, y, or z
    `dimension`.

    Example:

    ```mlir
    %gidX = gpu.global_id x
    %gidX = gpu.global_id x upper_bound 65536
    ```

```
- **EN**: Introduces declarations for `GPU_GlobalIdOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_GlobalIdOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 288-301
```tablegen
    The `upper_bound` attribute defines an upper bound analogously to the ones on
    `thread_id` and `block_id`. If one is not set, the bound may be inferred from
    a combination of `known_block_size` and `known_grid_size`-type annotations.
  }];
}


def GPU_NumSubgroupsOp : GPU_Op<"num_subgroups", [
      Pure, DeclareOpInterfaceMethods<InferIntRangeInterface, ["inferResultRanges"]>]>,
    Arguments<(ins OptionalAttr<IndexAttr>:$upper_bound)>,
    Results<(outs Index:$result)> {
  let description = [{
    Returns the number of subgroups within a workgroup.

```
- **EN**: Introduces declarations for `GPU_NumSubgroupsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_NumSubgroupsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 302-312
```tablegen
    Example:

    ```mlir
    %numSg = gpu.num_subgroups : index
    ```

    If `upper_bound` is set, executions with more than `upper_bound` subgroups
    per workgroup cause undefined behavior. There is a default upper bound of
    `kMaxDim` (currently uint32_t::max).
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 313-322
```tablegen
  let assemblyFormat = "(`upper_bound` $upper_bound^)? attr-dict `:` type($result)";
}

def GPU_SubgroupSizeOp : GPU_Op<"subgroup_size", [
      Pure, DeclareOpInterfaceMethods<InferIntRangeInterface, ["inferResultRanges"]>]>,
    Arguments<(ins OptionalAttr<IndexAttr>:$upper_bound)>,
    Results<(outs Index:$result)> {
  let description = [{
    Returns the number of threads within a subgroup.

```
- **EN**: Introduces declarations for `GPU_SubgroupSizeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupSizeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 323-334
```tablegen
    Example:

    ```mlir
    %sgSz = gpu.subgroup_size : index
    ```

    Executions where the number of threads per subgroup exceed `upper_bound` cause
    undefined behavior. When no `upper_bound` is specified, range analyses and
    similar machinery assume the default bound of `kMaxSubgroupSize`, currently
    128.
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 335-347
```tablegen
  let assemblyFormat = "(`upper_bound` $upper_bound^)? attr-dict `:` type($result)";
}

def GPU_OptionalDimSizeHintAttr : ConfinedAttr<OptionalAttr<DenseI32ArrayAttr>,
  [AttrConstraint<Or<[IsNullAttr.predicate, And<[DenseArrayCount<3>.predicate,
      CPred<"([](mlir::Attribute a) { "
            "  auto arr = ::llvm::cast<::mlir::DenseI32ArrayAttr>(a).asArrayRef();"
            "  return ::llvm::all_of(arr, [](int32_t v) { return v >= 1; });"
            "})($_self)">]>]>,
    "with 3 elements (if present) and all elements >= 1"
  >]
>;

```
- **EN**: Introduces declarations for `GPU_OptionalDimSizeHintAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_OptionalDimSizeHintAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 348-357
```tablegen
def GPU_GPUFuncOp : GPU_Op<"func", [
    HasParent<"GPUModuleOp">, AutomaticAllocationScope, FunctionOpInterface,
    IsolatedFromAbove, AffineScope
  ]> {
  let summary = "Function executable on a GPU";

  let description = [{
    Defines a function that can be executed on a GPU. This supports memory
    attribution and its body has a particular execution model.

```
- **EN**: Introduces declarations for `GPU_GPUFuncOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_GPUFuncOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 358-374
```tablegen
    GPU functions are either kernels (as indicated by the `kernel` attribute) or
    regular functions. The former can be launched from the host side, while the
    latter are device side only.

    The memory attribution defines SSA values that correspond to memory buffers
    allocated in the memory hierarchy of the GPU (see below).

    The operation has one attached region that corresponds to the body of the
    function. The region arguments consist of the function arguments without
    modification, followed by buffers defined in memory annotations. The body of
    a GPU function, when launched, is executed by multiple work items. There are
    no guarantees on the order in which work items execute, or on the connection
    between them. In particular, work items are not necessarily executed in
    lock-step. Synchronization ops such as "gpu.barrier" should be used to
    coordinate work items. Declarations of GPU functions, i.e. not having the
    body region, are not supported.

```
- **EN**: Implements logic around `kernels`, `GPU`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `kernels`, `GPU` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 375-384
```tablegen
    A function may optionally be annotated with the block and/or grid sizes
    that will be used when it is launched using the `known_block_size` and
    `known_grid_size` attributes, respectively. If set, these attributes must
    be arrays of three 32-bit integers giving the x, y, and z launch dimensions.
    Launching a kernel that has these annotations, or that calls a function with
    these annotations, using a block size or grid size other than what is specified
    is undefined behavior. These attributes may be set on non-`gpu.func` functions
    by using `gpu.known_block_size` or `gpu.known_grid_size`, but this carries
    the risk that they will de discarded.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 385-395
```tablegen
    Syntax:

    ```
    op ::= `gpu.func` symbol-ref-id `(` argument-list `)` (`->`
    function-result-list)?
           memory-attribution `kernel`? function-attributes? region

    memory-attribution ::= (`workgroup` `(` ssa-id-and-type-list `)`)?
                           (`private` `(` ssa-id-and-type-list `)`)?
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 396-407
```tablegen
    Example:

    ```mlir
    gpu.func @foo(%arg0: index)
        workgroup(%workgroup: memref<32xf32, 3>)
        private(%private: memref<1xf32, 5>)
        kernel
        attributes {qux: "quux"} {
      gpu.return
    }
    ```

```
- **EN**: Implements logic around `foo`, `workgroup`, `private`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `foo`, `workgroup`, `private` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 408-417
```tablegen
    The generic form illustrates the concept

    ```mlir
    "gpu.func"(%arg: index) {sym_name: "foo", kernel, qux: "quux"} ({
    ^bb0(%arg0: index, %workgroup: memref<32xf32, 3>,
         %private: memref<1xf32, 5>):
      "gpu.return"() : () -> ()
    }) : (index) -> ()
    ```

```
- **EN**: Implements logic around `bb0`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bb0` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 418-433
```tablegen
    Note the non-default memory spaces used in memref types in memory
    attribution.
  }];

  let arguments = (ins TypeAttrOf<FunctionType>:$function_type,
                       OptionalAttr<DictArrayAttr>:$arg_attrs,
                       OptionalAttr<DictArrayAttr>:$res_attrs,
                       OptionalAttr<DictArrayAttr>:$workgroup_attrib_attrs,
                       OptionalAttr<DictArrayAttr>:$private_attrib_attrs,
                       GPU_OptionalDimSizeHintAttr:$known_block_size,
                       GPU_OptionalDimSizeHintAttr:$known_grid_size,
                       GPU_OptionalDimSizeHintAttr:$known_cluster_size,
                       OptionalAttr<ConfinedAttr<I64Attr, [IntNonNegative]>>:$workgroup_attributions,
                       UnitAttr:$kernel);
  let regions = (region AnyRegion:$body);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 434-444
```tablegen
  let skipDefaultBuilders = 1;

  let builders = [
    OpBuilder<(ins "StringRef":$name, "FunctionType":$type,
      CArg<"TypeRange", "{}">:$workgroupAttributions,
      CArg<"TypeRange", "{}">:$privateAttributions,
      CArg<"ArrayRef<NamedAttribute>", "{}">:$attrs)>
  ];

  let extraClassDeclaration = [{
    /// Returns `true` if the GPU function defined by this Op is a kernel, i.e.
```
- **EN**: Implements logic around `OpBuilder`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 445-454
```tablegen
    /// it is intended to be launched from host. Also accepts legacy discardable
    /// `gpu.kernel` for compatibility with older generic IR.
    bool isKernel() {
      if (getKernel())
        return true;
      return (*this)->getAttrOfType<UnitAttr>(
          GPUDialect::getKernelFuncAttrName()) != nullptr;
    }

    /// Returns the number of buffers located in the workgroup memory.
```
- **EN**: Implements logic around `isKernel`, `getKernel`, `getAttrOfType`, `getKernelFuncAttrName`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `isKernel`, `getKernel`, `getAttrOfType`, `getKernelFuncAttrName` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 455-464
```tablegen
    unsigned getNumWorkgroupAttributions() {
      std::optional<int64_t> v = getWorkgroupAttributions();
      if (!v)
        return 0;
      int64_t value = *v;
      assert(value < std::numeric_limits<uint32_t>::max() &&
             "invalid workgroup attribution count");
      return static_cast<unsigned>(value);
    }

```
- **EN**: Implements logic around `getNumWorkgroupAttributions`, `getWorkgroupAttributions`, `assert`, `static_cast`.
- **CN**: 围绕 `getNumWorkgroupAttributions`, `getWorkgroupAttributions`, `assert`, `static_cast` 实现具体逻辑。

### Lines 465-479
```tablegen
    /// Return the index of the first workgroup attribution in the block argument
    /// list.
    unsigned getFirstWorkgroupAttributionIndex() {
      return getFunctionType().getNumInputs();
    }

    /// Returns a list of block arguments that correspond to buffers located in
    /// the workgroup memory
    ArrayRef<BlockArgument> getWorkgroupAttributionBBArgs() {
      auto begin =
          std::next(getBody().args_begin(), getFirstWorkgroupAttributionIndex());
      auto end = std::next(begin, getNumWorkgroupAttributions());
      return {begin, end};
    }

```
- **EN**: Implements logic around `getFirstWorkgroupAttributionIndex`, `getFunctionType`, `getWorkgroupAttributionBBArgs`, `next`.
- **CN**: 围绕 `getFirstWorkgroupAttributionIndex`, `getFunctionType`, `getWorkgroupAttributionBBArgs`, `next` 实现具体逻辑。

### Lines 480-489
```tablegen
    /// Adds a new block argument that corresponds to buffers located in
    /// workgroup memory.
    BlockArgument addWorkgroupAttribution(Type type, Location loc);

    /// Get the workgroup attribution attribute dictionary for the attribution
    /// at index `index`, counted from the start of the workgroup attributions.
    DictionaryAttr getworkgroupAttributionAttrs(unsigned index);

    /// Set the workgroup attribution attribute dictionary for the attribution
    /// at index `index`, counted from the start of the workgroup attributions.
```
- **EN**: Declares APIs or declarative rules around `addWorkgroupAttribution`, `getworkgroupAttributionAttrs`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `addWorkgroupAttribution`, `getworkgroupAttributionAttrs` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 490-499
```tablegen
    void setworkgroupAttributionAttrs(unsigned index, DictionaryAttr value);

    /// Get an attribute for a workgroup attribution. `index` is counted
    /// from the start of the workgroup attributions, not the start of the block.
    Attribute getWorkgroupAttributionAttr(unsigned index, StringAttr name);
    Attribute getWorkgroupAttributionAttr(unsigned index, StringRef name) {
      return getWorkgroupAttributionAttr(index, StringAttr::get((*this)->getContext(), name));
    }

    /// Set an attribute for a workgroup attribution. `index` is counted
```
- **EN**: Implements logic around `setworkgroupAttributionAttrs`, `getWorkgroupAttributionAttr`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setworkgroupAttributionAttrs`, `getWorkgroupAttributionAttr` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 500-512
```tablegen
    /// from the start of the workgroup attributions, not the start of the block.
    /// A null `value` removes an attributino attribute.
    void setWorkgroupAttributionAttr(unsigned index, StringAttr name, Attribute value);
    void setWorkgroupAttributionAttr(unsigned index, StringRef name, Attribute value) {
      return setWorkgroupAttributionAttr(index, StringAttr::get((*this)->getContext(), name), value);
    }

    /// Returns the number of buffers located in the private memory.
    unsigned getNumPrivateAttributions() {
      return getBody().getNumArguments() - getFunctionType().getNumInputs() -
          getNumWorkgroupAttributions();
    }

```
- **EN**: Implements logic around `setWorkgroupAttributionAttr`, `getNumPrivateAttributions`, `getBody`, `getNumWorkgroupAttributions`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setWorkgroupAttributionAttr`, `getNumPrivateAttributions`, `getBody`, `getNumWorkgroupAttributions` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 513-527
```tablegen
    /// Returns the index of the first private buffer in the block argument list.
    unsigned getFirstPrivateAttributionIndex() {
      // Buffers on the private memory always come after buffers on the workgroup
      // memory.
      return getFunctionType().getNumInputs() + getNumWorkgroupAttributions();
    }

    /// Returns a list of block arguments that correspond to buffers located in
    /// the private memory.
    ArrayRef<BlockArgument> getPrivateAttributions() {
      auto begin =
          std::next(getBody().args_begin(), getFirstPrivateAttributionIndex());
      return {begin, getBody().args_end()};
    }

```
- **EN**: Implements logic around `getFirstPrivateAttributionIndex`, `getFunctionType`, `getPrivateAttributions`, `next`, and 1 more symbols.
- **CN**: 围绕 `getFirstPrivateAttributionIndex`, `getFunctionType`, `getPrivateAttributions`, `next`, and 1 more symbols 实现具体逻辑。

### Lines 528-537
```tablegen
    /// Adds a new block argument that corresponds to buffers located in
    /// private memory.
    BlockArgument addPrivateAttribution(Type type, Location loc);

    /// Get the private attribution attribute dictionary for the attribution
    /// at index `index`, counted from the start of the private attributions.
    DictionaryAttr getPrivateAttributionAttrs(unsigned index);

    /// Set the private attribution attribute dictionary for the attribution
    /// at index `index`, counted from the start of the private attributions.
```
- **EN**: Declares APIs or declarative rules around `addPrivateAttribution`, `getPrivateAttributionAttrs`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `addPrivateAttribution`, `getPrivateAttributionAttrs` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 538-547
```tablegen
    void setPrivateAttributionAttrs(unsigned index, DictionaryAttr value);

    /// Get an attribute for a private attribution. `index` is counted
    /// from the start of the private attributions, not the start of the block.
    Attribute getPrivateAttributionAttr(unsigned index, StringAttr name);
    Attribute getPrivateAttributionAttr(unsigned index, StringRef name) {
      return getPrivateAttributionAttr(index, StringAttr::get((*this)->getContext(), name));
    }

    /// Set an attribute for a private attribution. `index` is counted
```
- **EN**: Implements logic around `setPrivateAttributionAttrs`, `getPrivateAttributionAttr`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setPrivateAttributionAttrs`, `getPrivateAttributionAttr` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 548-557
```tablegen
    /// from the start of the private attributions, not the start of the block.
    /// A null `value` removes an attribute.
    void setPrivateAttributionAttr(unsigned index, StringAttr name, Attribute value);
    void setPrivateAttributionAttr(unsigned index, StringRef name, Attribute value) {
      return setPrivateAttributionAttr(index, StringAttr::get((*this)->getContext(), name), value);
    }

    /// Returns the argument types of this function.
    ArrayRef<Type> getArgumentTypes() { return getFunctionType().getInputs(); }

```
- **EN**: Implements logic around `setPrivateAttributionAttr`, `getArgumentTypes`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setPrivateAttributionAttr`, `getArgumentTypes` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 558-567
```tablegen
    /// Returns the result types of this function.
    ArrayRef<Type> getResultTypes() { return getFunctionType().getResults(); }

    Region *getCallableRegion() { return &getBody(); }

    /// Returns the keywords used in the custom syntax for this Op.
    static StringRef getWorkgroupKeyword() { return "workgroup"; }
    static StringRef getPrivateKeyword() { return "private"; }
    static StringRef getKernelKeyword() { return "kernel"; }

```
- **EN**: Implements logic around `getResultTypes`, `getCallableRegion`, `getWorkgroupKeyword`, `getPrivateKeyword`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getResultTypes`, `getCallableRegion`, `getWorkgroupKeyword`, `getPrivateKeyword`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 568-577
```tablegen
    /// Hook for FunctionOpInterface verifier.
    LogicalResult verifyType();

    /// Verifies the body of the function.
    LogicalResult verifyBody();
  }];
  let hasCustomAssemblyFormat = 1;
}

def GPU_DynamicSharedMemoryOp : GPU_Op<"dynamic_shared_memory", [Pure]>
```
- **EN**: Introduces declarations for `GPU_DynamicSharedMemoryOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_DynamicSharedMemoryOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 578-587
```tablegen
{
  let summary = "Get the memref for dynamic shared memory";

  let description = [{
    This operation provides a memref pointer to the start of dynamic shared
    memory, often referred to as workgroup memory. It's important to note that
    this dynamic shared memory needs to be allocated at kernel launch. One can
    conveniently utilize the `dynamic_shared_memory_size` parameter of
    `gpu.launch` for this purpose.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 588-602
```tablegen
    Examples:
    ```mlir
    %0 = gpu.dynamic.shared.memory : memref<?xi8, #gpu.address_space<workgroup>>
    %1 = memref.view %0[%c8192][] : memref<?xi8, #gpu.address_space<workgroup>>
                            to memref<32x64xf32, #gpu.address_space<workgroup>>
    %2 = memref.view %0[%c16384][] : memref<?xi8, #gpu.address_space<workgroup>>
                            to memref<32x64xf32, #gpu.address_space<workgroup>>
    ```
  }];
  let arguments = (ins);
  let results = (outs Arg<MemRefRankOf<[I8], [1]>>:$resultMemref);
  let assemblyFormat = [{ attr-dict `:` type($resultMemref) }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 603-622
```tablegen
def LaunchIndx : AnyTypeOf<[Index, I32, I64]>;

def GPU_LaunchFuncOp :GPU_Op<"launch_func", [
      GPU_AsyncOpInterface, AttrSizedOperandSegments,
      DeclareOpInterfaceMethods<SymbolUserOpInterface>,
      AllTypesMatch<["gridSizeX", "gridSizeY", "gridSizeZ", "blockSizeX",
                     "blockSizeY", "blockSizeZ"]>]>,
    Arguments<(ins Variadic<GPU_AsyncToken>:$asyncDependencies,
               SymbolRefAttr:$kernel,
               LaunchIndx:$gridSizeX,
               LaunchIndx:$gridSizeY,
               LaunchIndx:$gridSizeZ,
               LaunchIndx:$blockSizeX,
               LaunchIndx:$blockSizeY,
               LaunchIndx:$blockSizeZ,
               Optional<LaunchIndx>:$clusterSizeX,
               Optional<LaunchIndx>:$clusterSizeY,
               Optional<LaunchIndx>:$clusterSizeZ,
               Optional<I32>:$dynamicSharedMemorySize,
               UnitAttr:$cooperative,
```
- **EN**: Introduces declarations for `LaunchIndx`, `GPU_LaunchFuncOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LaunchIndx`, `GPU_LaunchFuncOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 623-639
```tablegen
               Variadic<AnyType>:$kernelOperands,
               Optional<AnyType>:$asyncObject)>,
    Results<(outs Optional<GPU_AsyncToken>:$asyncToken)> {
  let summary = "Launches a function as a GPU kernel";

  let description = [{
    Launch a kernel function on the specified grid of thread blocks.
    `gpu.launch` operations are lowered to `gpu.launch_func` operations by
    outlining the kernel body into a function in a dedicated module, which
    reflects the separate compilation process. The kernel function is required
    to have the `gpu.kernel` attribute. The module containing the kernel
    function is required to be a gpu.module. And finally, the module containing
    the kernel module (which thus cannot be the top-level module) is required
    to have the `gpu.container_module` attribute. The `gpu.launch_func`
    operation has a symbol attribute named `kernel` to identify the fully
    specified kernel function to launch (both the gpu.module and func).

```
- **EN**: Implements logic around `Results`, `module`, `launch`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `Results`, `module`, `launch` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 640-651
```tablegen
    The `gpu.launch_func` supports async dependencies: the kernel does not start
    executing until the ops producing those async dependencies have completed.

    By the default, the host implicitly blocks until kernel execution has
    completed. If the `async` keyword is present, the host does not block but
    instead a `!gpu.async.token` is returned. Other async GPU ops can take this
    token as dependency.

    The operation requires at least the grid and block sizes along the x,y,z
    dimensions as arguments. When a lower-dimensional kernel is required,
    unused sizes must be explicitly set to `1`.

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 652-664
```tablegen
    The remaining operands are optional. The first optional operand corresponds
    to the amount of dynamic shared memory a kernel's workgroup should be
    allocated; when this operand is not present, a zero size is assumed.

    The remaining operands if present are passed as arguments to the kernel
    function.

    The `gpu.launch_func` also supports kernel launching with clusters if
    supported by the target architecture. The cluster size can be set by
    `clusterSizeX`, `clusterSizeY`, and `clusterSizeZ` arguments. When these
    arguments are present, the Op launches a kernel that clusters the given
    thread blocks. This feature is exclusive to certain architectures.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 665-674
```tablegen
    The `cooperative` attribute indicates that the kernel should be launched
    cooperatively, guaranteeing that all thread blocks in the grid are
    co-resident on the GPU simultaneously. This enables grid-wide
    synchronization patterns.

    Example:

    ```mlir
    module attributes {gpu.container_module} {

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 675-686
```tablegen
      // This module creates a separate compilation unit for the GPU compiler.
      gpu.module @kernels {
        func.func @kernel_1(%arg0 : f32, %arg1 : memref<?xf32, 1>)
            attributes { nvvm.kernel = true } {

          // Operations that produce block/thread IDs and dimensions are
          // injected when outlining the `gpu.launch` body to a function called
          // by `gpu.launch_func`.
          %tIdX = gpu.thread_id x
          %tIdY = gpu.thread_id y
          %tIdZ = gpu.thread_id z

```
- **EN**: Implements logic around `kernel_1`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `kernel_1` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 687-698
```tablegen
          %bDimX = gpu.block_dim x
          %bDimY = gpu.block_dim y
          %bDimZ = gpu.block_dim z

          %bIdX = gpu.block_id x
          %bIdY = gpu.block_id y
          %bIdZ = gpu.block_id z

          %gDimX = gpu.grid_dim x
          %gDimY = gpu.grid_dim y
          %gDimZ = gpu.grid_dim z

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 699-712
```tablegen
          // (Optional)  Cluster size only for support architectures
          %cIdX = gpu.cluster_id x
          %cIdY = gpu.cluster_id y
          %cIdZ = gpu.cluster_id z

          %cDimX = gpu.cluster_dim x
          %cDimY = gpu.cluster_dim y
          %cDimZ = gpu.cluster_dim z

          "some_op"(%bx, %tx) : (index, index) -> ()
          %42 = load %arg1[%bx] : memref<?xf32, 1>
        }
      }

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 713-728
```tablegen
      %t0 = gpu.wait async
      gpu.launch_func
          async                           // (Optional) Don't block host, return token.
          [%t0]                           // (Optional) Execute only after %t0 has completed.
          @kernels::@kernel_1             // Kernel function.
          clusters in (%cst, %cst, %cst)  // (Optional) Cluster size only for support architectures.
          blocks in (%cst, %cst, %cst)    // Grid size.
          threads in (%cst, %cst, %cst)   // Block size.
          dynamic_shared_memory_size %s   // (Optional) Amount of dynamic shared
                                          // memory to allocate for a workgroup.
          args(%arg0 : f32,               // (Optional) Kernel arguments.
               %arg1 : memref<?xf32, 1>)
    }
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `in`, `args`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `in`, `args` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 729-748
```tablegen
  let skipDefaultBuilders = 1;

  let builders = [
    OpBuilder<(ins "GPUFuncOp":$kernelFunc, "KernelDim3":$gridSize,
      "KernelDim3":$blockSize, "Value":$dynamicSharedMemorySize,
      "ValueRange":$kernelOperands,
      CArg<"Type", "nullptr">:$asyncTokenType,
      CArg<"ValueRange", "{}">:$asyncDependencies,
      CArg<"std::optional<KernelDim3>", "std::nullopt">:$clusterSize)>,
    OpBuilder<(ins "SymbolRefAttr":$kernel, "KernelDim3":$gridSize,
      "KernelDim3":$blockSize, "Value":$dynamicSharedMemorySize,
      "ValueRange":$kernelOperands,
      "Type":$asyncTokenType,
      CArg<"ValueRange", "{}">:$asyncDependencies,
      CArg<"std::optional<KernelDim3>", "std::nullopt">:$clusterSize)>,
    OpBuilder<(ins "SymbolRefAttr":$kernel, "KernelDim3":$gridSize,
      "KernelDim3":$blockSize, "Value":$dynamicSharedMemorySize,
      "ValueRange":$kernelOperands,
      CArg<"Value", "nullptr">:$asyncObject,
      CArg<"std::optional<KernelDim3>", "std::nullopt">:$clusterSize)>
```
- **EN**: Implements logic around `OpBuilder`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 749-758
```tablegen
  ];

  let extraClassDeclaration = [{
    /// The name of the kernel's containing module.
    StringAttr getKernelModuleName();

    /// The name of the kernel.
    StringAttr getKernelName();

    /// Returns true if cluster size is specified.
```
- **EN**: Implements logic around `getKernelModuleName`, `getKernelName`.
- **CN**: 围绕 `getKernelModuleName`, `getKernelName` 实现具体逻辑。

### Lines 759-768
```tablegen
    bool hasClusterSize() {
      if (getClusterSizeX() && getClusterSizeY() && getClusterSizeZ())
        return true;
      return false;
    }

    /// The number of operands passed to the kernel function.
    unsigned getNumKernelOperands();

    /// The i-th operand passed to the kernel function.
```
- **EN**: Implements logic around `hasClusterSize`, `getClusterSizeX`, `getNumKernelOperands`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `hasClusterSize`, `getClusterSizeX`, `getNumKernelOperands` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 769-778
```tablegen
    Value getKernelOperand(unsigned i);

    /// Get the SSA values passed as operands to specify the cluster size.
    /// When the cluster sizes are not specified, it asserts.
    KernelDim3 getClusterSizeOperandValues();

    /// Get the SSA values passed as operands to specify the grid size.
    KernelDim3 getGridSizeOperandValues();

    /// Get the SSA values passed as operands to specify the block size.
```
- **EN**: Declares APIs or declarative rules around `getKernelOperand`, `getClusterSizeOperandValues`, `getGridSizeOperandValues`.
- **CN**: 声明与 `getKernelOperand`, `getClusterSizeOperandValues`, `getGridSizeOperandValues` 相关的 API 或声明式规则。

### Lines 779-798
```tablegen
    KernelDim3 getBlockSizeOperandValues();

    // This needs to quietly verify if attributes with names defined below are
    // present since it is run before the verifier of this op.
    friend LogicalResult GPUDialect::verifyOperationAttribute(Operation *,
                                                              NamedAttribute);
  }];

  let assemblyFormat = [{
      custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
      (`<` $asyncObject^ `:` type($asyncObject) `>`)?
      $kernel
      ( `clusters` `in` ` ` `(` $clusterSizeX^ `,` $clusterSizeY `,` $clusterSizeZ `)` )?
      `blocks` `in` ` ` `(` $gridSizeX `,` $gridSizeY `,` $gridSizeZ `)`
      `threads` `in` ` ` `(` $blockSizeX `,` $blockSizeY `,` $blockSizeZ `)`
      custom<LaunchDimType>(type($gridSizeX), ref($clusterSizeX), type($clusterSizeX), type($clusterSizeY), type($clusterSizeZ))
      (`dynamic_shared_memory_size` $dynamicSharedMemorySize^)?
      (`cooperative` $cooperative^)?
      custom<LaunchFuncOperands>($kernelOperands, type($kernelOperands)) attr-dict
  }];
```
- **EN**: Implements logic around `getBlockSizeOperandValues`, `verifyOperationAttribute`, `custom`, `type`; this block handles textual assembly parsing or printing concerns; checks structural or semantic invariants; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getBlockSizeOperandValues`, `verifyOperationAttribute`, `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并检查结构或语义不变式，并处理方言 IR 实体，如操作、类型或属性。

### Lines 799-818
```tablegen
  let hasVerifier = 1;
}

def GPU_LaunchOp : GPU_Op<"launch", [
      AffineScope, AutomaticAllocationScope, AttrSizedOperandSegments,
      DeclareOpInterfaceMethods<InferIntRangeInterface, ["inferResultRanges"]>,
      GPU_AsyncOpInterface, RecursiveMemoryEffects]>,
    Arguments<(ins Variadic<GPU_AsyncToken>:$asyncDependencies,
               Index:$gridSizeX, Index:$gridSizeY, Index:$gridSizeZ,
               Index:$blockSizeX, Index:$blockSizeY, Index:$blockSizeZ,
               Optional<Index>:$clusterSizeX,
               Optional<Index>:$clusterSizeY,
               Optional<Index>:$clusterSizeZ,
               Optional<I32>:$dynamicSharedMemorySize,
               UnitAttr:$cooperative,
               OptionalAttr<FlatSymbolRefAttr>:$module,
               OptionalAttr<FlatSymbolRefAttr>:$function,
               OptionalAttr<ConfinedAttr<I64Attr, [IntNonNegative]>>:$workgroup_attributions)>,
    Results<(outs Optional<GPU_AsyncToken>:$asyncToken)> {
  let summary = "GPU kernel launch operation";
```
- **EN**: Introduces declarations for `GPU_LaunchOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_LaunchOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 819-830
```tablegen

  let description = [{
    Launch a kernel on the specified grid of thread blocks. The body of the
    kernel is defined by the single region that this operation contains. The
    operation takes an optional list of async dependencies followed by six
    operands and an optional operand.

    The `async` keyword indicates the kernel should be launched asynchronously;
    the operation returns a new !gpu.async.token when the keyword is specified.
    The kernel launched does not start executing until the ops producing its
    async dependencies (optional operands) have completed.

```
- **EN**: Implements logic around `dependencies`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `dependencies` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 831-840
```tablegen
    The first three operands (following any async dependencies) are grid sizes
    along the x,y,z dimensions and the following three are block sizes along the
    x,y,z dimensions. When a lower-dimensional kernel is required, unused sizes
    must be explicitly set to `1`.  The last operand is optional and corresponds
    to the amount of dynamic shared memory a kernel's workgroup should be
    allocated; when this operand is not present, a zero size is assumed.

    The body region has at least _twelve_ arguments, or _eighteen_ if cluster
    dimensions are present, grouped as follows:

```
- **EN**: Declares APIs or declarative rules around `operands`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `operands` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 841-852
```tablegen
    -   three optional arguments that contain cluster identifiers along x,y,z
        dimensions;
    -   three arguments that contain block identifiers along x,y,z dimensions;
    -   three arguments that contain thread identifiers along x,y,z dimensions;
    -   operands of the `gpu.launch` operation as is (i.e. the operands for
        grid and block sizes).
    -   a variadic number of Workgroup memory attributions.
    -   a variadic number of Private memory attributions.

    The `function` and `module` attributes are optional and specifies
    the kernel name and a module in which the kernel should be outlined.

```
- **EN**: Declares APIs or declarative rules around `is`; this block touches target- or accelerator-specific semantics.
- **CN**: 声明与 `is` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义。

### Lines 853-869
```tablegen
    Syntax:

    ```
    operation ::= `gpu.launch` (`async` (`[` ssa-id-list `]`)? )?
                             ( `clusters` `(` ssa-id-list `)` `in` ssa-reassignment )?
                             `blocks` `(` ssa-id-list `)` `in` ssa-reassignment
                             `threads` `(` ssa-id-list `)` `in` ssa-reassignment
                             (dynamic_shared_memory_size ssa-use)?
                             (`module(` symbol-ref-id `)`)?
                             (`function(` symbol-ref-id `)`)?
                             memory-attribution
                             region attr-dict?
    ssa-reassignment ::= `(` ssa-id `=` ssa-use (`,` ssa-id `=` ssa-use)* `)`
    memory-attribution ::= (`workgroup` `(` ssa-id-and-type-list `)`)?
                           (`private` `(` ssa-id-and-type-list `)`)?
    ```

```
- **EN**: Implements logic around `module`, `function`, `use`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `module`, `function`, `use` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 870-881
```tablegen
    Example:

    ```mlir
    gpu.launch blocks(%bx, %by, %bz) in (%sz_bx = %0, %sz_by = %1, %sz_bz = %2)
               threads(%tx, %ty, %tz) in (%sz_tx = %3, %sz_ty = %4, %sz_tz = %5) {
      // Block and thread identifiers, as well as block/grid sizes are
      // immediately usable inside body region.
      "some_op"(%bx, %tx) : (index, index) -> ()
      // Assuming %val1 is defined outside the gpu.launch region.
      %42 = load %val1[%bx] : memref<?xf32, 1>
    }

```
- **EN**: Implements logic around `blocks`, `threads`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `blocks`, `threads` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 882-898
```tablegen
    // Generic syntax explains how the pretty syntax maps to the IR structure.
    "gpu.launch"(%cst, %cst, %c1,  // Grid sizes.
                 %cst, %c1, %c1)   // Block sizes.

        {/*attributes*/}
        // All sizes and identifiers have "index" size.
        : (index, index, index, index, index, index) -> () {
    // The operation passes block and thread identifiers, followed by grid and
    // block sizes.
    ^bb0(%bx : index, %by : index, %bz : index,
         %tx : index, %ty : index, %tz : index,
         %num_bx : index, %num_by : index, %num_bz : index,
         %num_tx : index, %num_ty : index, %num_tz : index)
      "some_op"(%bx, %tx) : (index, index) -> ()
      %3 = "memref.load"(%val1, %bx) : (memref<?xf32, 1>, index) -> f32
    }

```
- **EN**: Implements logic around `bb0`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bb0` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 899-910
```tablegen
    // Launch with memory attributions.
    gpu.launch blocks(%bx, %by, %bz) in (%sz_bx = %0, %sz_by = %1, %sz_bz = %2)
               threads(%tx, %ty, %tz) in (%sz_tx = %3, %sz_ty = %4, %sz_tz = %5)
               workgroup(%workgroup: memref<32xf32, 3>)
               private(%private: memref<1xf32, 5>) {
      // Block and thread identifiers, as well as block/grid sizes are
      // immediately usable inside body region.
      "some_op"(%bx, %tx) : (index, index) -> ()
      // Assuming %val1 is defined outside the gpu.launch region.
      %42 = load %workgroup[%bx] : memref<32xf32, 3>
    }

```
- **EN**: Implements logic around `blocks`, `threads`, `workgroup`, `private`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `blocks`, `threads`, `workgroup`, `private` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 911-920
```tablegen
    // Launch with clusters.
    gpu.launch clusters(%cx, %cy, %cz) in (%sz_cx = %0, %sz_cy = %1, %sz_cz = %2)
               blocks(%bx, %by, %bz) in (%sz_bx = %3, %sz_by = %4, %sz_bz = %5)
               threads(%tx, %ty, %tz) in (%sz_tx = %6, %sz_ty = %7, %sz_tz = %8)
    {
      // Cluster, block and thread identifiers, as well as cluster/block/grid
      // sizes are immediately usable inside body region.
      "some_op"(%cx, %bx, %tx) : (index, index, index) -> ()
    }

```
- **EN**: Implements logic around `clusters`, `blocks`, `threads`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `clusters`, `blocks`, `threads` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 921-936
```tablegen
    // Launch with module and function attributes.
    gpu.launch blocks(%bx, %by, %bz) in (%sz_bx = %0, %sz_by = %1, %sz_bz = %2)
               threads(%tx, %ty, %tz) in (%sz_tx = %3, %sz_ty = %4, %sz_tz = %5)
               module(@kernel_module) function(@kernel_func) {
      "some_op"(%bx, %tx) : (index, index) -> ()
      %42 = load %val1[%bx] : memref<?xf32, 1>
    }
    ```

    Rationale: using operation/block arguments gives analyses a clear way of
    understanding that a value has additional semantics (e.g., we will need to
    know what value corresponds to threadIdx.x for coalescing). We can recover
    these properties by analyzing the operations producing values, but it is
    easier just to have that information by construction.
  }];

```
- **EN**: Implements logic around `blocks`, `threads`, `module`, `semantics`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `blocks`, `threads`, `module`, `semantics` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 937-956
```tablegen
  let regions = (region AnyRegion:$body);

  let skipDefaultBuilders = 1;

  let builders = [
    OpBuilder<(ins "Value":$gridSizeX, "Value":$gridSizeY,
      "Value":$gridSizeZ, "Value":$blockSizeX, "Value":$blockSizeY,
      "Value":$blockSizeZ,
      CArg<"Value", "nullptr">:$dynamicSharedMemorySize,
      CArg<"Type", "nullptr">:$asyncTokenType,
      CArg<"ValueRange", "{}">:$asyncDependencies,
      CArg<"TypeRange", "{}">:$workgroupAttributions,
      CArg<"TypeRange", "{}">:$privateAttributions,
      CArg<"Value", "nullptr">:$clusterSizeX,
      CArg<"Value", "nullptr">:$clusterSizeY,
      CArg<"Value", "nullptr">:$clusterSizeZ,
      CArg<"FlatSymbolRefAttr", "nullptr">:$module,
      CArg<"FlatSymbolRefAttr", "nullptr">:$function)>,
  ];

```
- **EN**: Implements logic around `OpBuilder`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 957-966
```tablegen
  let extraClassDeclaration = [{
    /// Get the SSA values corresponding to kernel block identifiers.
    KernelDim3 getBlockIds();
    /// Get the SSA values corresponding to kernel thread identifiers.
    KernelDim3 getThreadIds();
    /// Get the SSA values corresponding to kernel cluster identifiers.
    std::optional<KernelDim3> getClusterIds();
    /// Get the SSA values corresponding to kernel grid size.
    KernelDim3 getGridSize();
    /// Get the SSA values corresponding to kernel block size.
```
- **EN**: Implements logic around `getBlockIds`, `getThreadIds`, `getClusterIds`, `getGridSize`.
- **CN**: 围绕 `getBlockIds`, `getThreadIds`, `getClusterIds`, `getGridSize` 实现具体逻辑。

### Lines 967-977
```tablegen
    KernelDim3 getBlockSize();
    /// Get the SSA values corresponding to kernel cluster size.
    std::optional<KernelDim3> getClusterSize();

    /// Get the SSA values passed as operands to specify the grid size.
    KernelDim3 getGridSizeOperandValues();
    /// Get the SSA values passed as operands to specify the block size.
    KernelDim3 getBlockSizeOperandValues();
    /// Get the SSA values passed as operands to specify the cluster size.
    std::optional<KernelDim3> getClusterSizeOperandValues();

```
- **EN**: Declares APIs or declarative rules around `getBlockSize`, `getClusterSize`, `getGridSizeOperandValues`, `getBlockSizeOperandValues`, and 1 more symbols.
- **CN**: 声明与 `getBlockSize`, `getClusterSize`, `getGridSizeOperandValues`, `getBlockSizeOperandValues`, and 1 more symbols 相关的 API 或声明式规则。

### Lines 978-988
```tablegen
    static StringRef getBlocksKeyword() { return "blocks"; }
    static StringRef getClustersKeyword() { return "clusters"; }
    static StringRef getThreadsKeyword() { return "threads"; }
    static StringRef getDynamicSharedMemorySizeKeyword() {
      return "dynamic_shared_memory_size";
    }

    /// The number of launch configuration operands, placed at the leading
    /// positions of the operand list.
    static constexpr unsigned kNumConfigOperands = 6;

```
- **EN**: Implements logic around `getBlocksKeyword`, `getClustersKeyword`, `getThreadsKeyword`, `getDynamicSharedMemorySizeKeyword`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getBlocksKeyword`, `getClustersKeyword`, `getThreadsKeyword`, `getDynamicSharedMemorySizeKeyword` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 989-999
```tablegen
    /// The number of region attributes containing the launch configuration,
    /// placed in the leading positions of the argument list.
    static constexpr unsigned kNumConfigRegionAttributes = 12;

    /// Returns true if cluster size is specified.
    bool hasClusterSize() {
      if (getClusterSizeX() && getClusterSizeY() && getClusterSizeZ())
        return true;
      return false;
    }
    /// Returns the number of operands including cluster size
```
- **EN**: Implements logic around `hasClusterSize`, `getClusterSizeX`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `hasClusterSize`, `getClusterSizeX` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1000-1011
```tablegen
    unsigned getNumConfigOperands() {
      return kNumConfigOperands + (hasClusterSize() ? 3 : 0);
    }
    /// Returns the number of region attributes including cluster size
    unsigned getNumConfigRegionAttributes() {
      return kNumConfigRegionAttributes + (hasClusterSize() ? 6 : 0);
    }

    /// Returns the keywords used in the custom syntax for this Op.
    static StringRef getWorkgroupKeyword() { return "workgroup"; }
    static StringRef getPrivateKeyword() { return "private"; }

```
- **EN**: Implements logic around `getNumConfigOperands`, `hasClusterSize`, `getNumConfigRegionAttributes`, `getWorkgroupKeyword`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getNumConfigOperands`, `hasClusterSize`, `getNumConfigRegionAttributes`, `getWorkgroupKeyword`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1012-1022
```tablegen
    /// Returns the number of buffers located in the workgroup memory.
    unsigned getNumWorkgroupAttributions() {
      std::optional<int64_t> v = getWorkgroupAttributions();
      if (!v)
        return 0;
      int64_t value = *v;
      assert(value < std::numeric_limits<uint32_t>::max() &&
             "invalid workgroup attribution count");
      return static_cast<unsigned>(value);
    }

```
- **EN**: Implements logic around `getNumWorkgroupAttributions`, `getWorkgroupAttributions`, `assert`, `static_cast`.
- **CN**: 围绕 `getNumWorkgroupAttributions`, `getWorkgroupAttributions`, `assert`, `static_cast` 实现具体逻辑。

### Lines 1023-1032
```tablegen
    /// Block arguments for workgroup memory buffers
    ArrayRef<BlockArgument> getWorkgroupAttributionBBArgs() {
      auto begin =
          std::next(getBody().args_begin(), getNumConfigRegionAttributes());
      auto end = std::next(begin, getNumWorkgroupAttributions());
      return {begin, end};
    }

    /// Adds a new block argument that corresponds to buffers located in
    /// workgroup memory.
```
- **EN**: Implements logic around `getWorkgroupAttributionBBArgs`, `next`.
- **CN**: 围绕 `getWorkgroupAttributionBBArgs`, `next` 实现具体逻辑。

### Lines 1033-1042
```tablegen
    BlockArgument addWorkgroupAttribution(Type type, Location loc);

    /// Returns the number of buffers located in the private memory.
    unsigned getNumPrivateAttributions() {
      return getBody().getNumArguments() - getNumConfigRegionAttributes() -
          getNumWorkgroupAttributions();
    }

    /// Returns a list of block arguments that correspond to buffers located in
    /// the private memory.
```
- **EN**: Implements logic around `addWorkgroupAttribution`, `getNumPrivateAttributions`, `getBody`, `getNumWorkgroupAttributions`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `addWorkgroupAttribution`, `getNumPrivateAttributions`, `getBody`, `getNumWorkgroupAttributions` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1043-1052
```tablegen
    ArrayRef<BlockArgument> getPrivateAttributions() {
      // Buffers on the private memory always come after buffers on the workgroup
      // memory.
      auto begin =
          std::next(getBody().args_begin(),
                    getNumConfigRegionAttributes() + getNumWorkgroupAttributions());
      return {begin, getBody().args_end()};
    }

    /// Adds a new block argument that corresponds to buffers located in
```
- **EN**: Implements logic around `getPrivateAttributions`, `next`, `getNumConfigRegionAttributes`, `getBody`.
- **CN**: 围绕 `getPrivateAttributions`, `next`, `getNumConfigRegionAttributes`, `getBody` 实现具体逻辑。

### Lines 1053-1062
```tablegen
    /// private memory.
    BlockArgument addPrivateAttribution(Type type, Location loc);
  }];

  let hasCanonicalizer = 1;
  let hasCustomAssemblyFormat = 1;
  let hasRegionVerifier = 1;
  let hasVerifier = 1;
}

```
- **EN**: Declares APIs or declarative rules around `addPrivateAttribution`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `addPrivateAttribution` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1063-1078
```tablegen
def GPU_PrintfOp : GPU_Op<"printf", [MemoryEffects<[MemWrite]>]>,
  Arguments<(ins StrAttr:$format,
                Variadic<AnyTypeOf<[AnyInteger, Index, AnyFloat]>>:$args)> {
  let summary = "Device-side printf, as in CUDA or OpenCL, for debugging";
  let description = [{
    `gpu.printf` takes a literal format string `format` and an arbitrary number of
    scalar arguments that should be printed.

    The format string is a C-style printf string, subject to any restrictions
    imposed by one's target platform.
  }];
  let assemblyFormat = [{
    $format attr-dict (`,` $args^ `:` type($args))?
  }];
}

```
- **EN**: Introduces declarations for `GPU_PrintfOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_PrintfOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1079-1088
```tablegen
def GPU_ReturnOp : GPU_Op<"return", [HasParent<"GPUFuncOp">, Pure,
                                     Terminator]>,
    Arguments<(ins Variadic<AnyType>:$operands)>, Results<(outs)> {
  let summary = "Terminator for GPU functions.";
  let description = [{
    A terminator operation for regions that appear in the body of `gpu.func`
    functions. The operands to the `gpu.return` are the result values returned
    by an invocation of the `gpu.func`.
  }];

```
- **EN**: Introduces declarations for `GPU_ReturnOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ReturnOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1089-1104
```tablegen
  let builders = [OpBuilder<(ins), [{ // empty}]>];

  let assemblyFormat = "attr-dict ($operands^ `:` type($operands))?";
  let hasVerifier = 1;
}

def GPU_TerminatorOp : GPU_Op<"terminator", [HasParent<"LaunchOp">,
                                             Pure, Terminator]>,
    Arguments<(ins)>, Results<(outs)> {
  let summary = "Terminator for GPU launch regions.";
  let description = [{
    A terminator operation for regions that appear in the body of `gpu.launch`
    operation.  These regions are not expected to return any value so the
    terminator takes no operands.
  }];

```
- **EN**: Introduces declarations for `GPU_TerminatorOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_TerminatorOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1105-1114
```tablegen
  let assemblyFormat = "attr-dict";
}

def GPU_YieldOp : GPU_Op<"yield", [Pure, ReturnLike, Terminator]>,
    Arguments<(ins Variadic<AnyType>:$values)> {
  let summary = "GPU yield operation";
  let description = [{
    `gpu.yield` is a special terminator operation for blocks inside regions
    in gpu ops. It returns values to the immediately enclosing gpu op.

```
- **EN**: Introduces declarations for `GPU_YieldOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_YieldOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1115-1125
```tablegen
    Example:

    ```mlir
    gpu.yield %f0, %f1 : f32, f32
    ```
  }];

  let builders = [
    OpBuilder<(ins), [{ /* nothing to do */ }]>
  ];

```
- **EN**: Implements logic around `OpBuilder`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 1126-1135
```tablegen
  let assemblyFormat = "attr-dict ($values^ `:` type($values))?";
}

// These mirror the reduction combining kinds from the vector dialect.
def GPU_AllReduceOpAdd : I32EnumAttrCase<"ADD", 0, "add">;
def GPU_AllReduceOpMul : I32EnumAttrCase<"MUL", 1, "mul">;
def GPU_AllReduceOpMinUI : I32EnumAttrCase<"MINUI", 2, "minui">;
def GPU_AllReduceOpMinSI : I32EnumAttrCase<"MINSI", 3, "minsi">;
// Follows the `arith.minnumf` semantics.
def GPU_AllReduceOpMinnumF : I32EnumAttrCase<"MINNUMF", 4, "minnumf">;
```
- **EN**: Introduces declarations for `GPU_AllReduceOpAdd`, `GPU_AllReduceOpMul`, `GPU_AllReduceOpMinUI`, `GPU_AllReduceOpMinSI`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_AllReduceOpAdd`, `GPU_AllReduceOpMul`, `GPU_AllReduceOpMinUI`, `GPU_AllReduceOpMinSI`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1136-1146
```tablegen
def GPU_AllReduceOpMaxUI : I32EnumAttrCase<"MAXUI", 5, "maxui">;
def GPU_AllReduceOpMaxSI : I32EnumAttrCase<"MAXSI", 6, "maxsi">;
// Follows the `arith.maxnumf` semantics.
def GPU_AllReduceOpMaxnumF : I32EnumAttrCase<"MAXNUMF", 7, "maxnumf">;
def GPU_AllReduceOpAnd : I32EnumAttrCase<"AND", 8, "and">;
def GPU_AllReduceOpOr  : I32EnumAttrCase<"OR",  9, "or">;
def GPU_AllReduceOpXor : I32EnumAttrCase<"XOR", 10, "xor">;
// Follows the `arith.minimumf` semantics.
def GPU_AllReduceOpMinimumF : I32EnumAttrCase<"MINIMUMF", 11, "minimumf">;
// Follows the `arith.maximumf` semantics.
def GPU_AllReduceOpMaximumF : I32EnumAttrCase<"MAXIMUMF", 12, "maximumf">;
```
- **EN**: Introduces declarations for `GPU_AllReduceOpMaxUI`, `GPU_AllReduceOpMaxSI`, `GPU_AllReduceOpMaxnumF`, `GPU_AllReduceOpAnd`, and 4 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_AllReduceOpMaxUI`, `GPU_AllReduceOpMaxSI`, `GPU_AllReduceOpMaxnumF`, `GPU_AllReduceOpAnd`, and 4 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1147-1166
```tablegen

def GPU_AllReduceOperation : I32EnumAttr<"AllReduceOperation",
    "built-in reduction operations supported by gpu.allreduce.",
    [
      GPU_AllReduceOpAdd,
      GPU_AllReduceOpMul,
      GPU_AllReduceOpMinUI,
      GPU_AllReduceOpMinSI,
      GPU_AllReduceOpMinnumF,
      GPU_AllReduceOpMaxUI,
      GPU_AllReduceOpMaxSI,
      GPU_AllReduceOpMaxnumF,
      GPU_AllReduceOpAnd,
      GPU_AllReduceOpOr,
      GPU_AllReduceOpXor,
      GPU_AllReduceOpMinimumF,
      GPU_AllReduceOpMaximumF
    ]>{
  let genSpecializedAttr = 0;
  let cppNamespace = "::mlir::gpu";
```
- **EN**: Introduces declarations for `GPU_AllReduceOperation`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_AllReduceOperation` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1167-1180
```tablegen
}

def AnyIntegerOrFloat : AnyTypeOf<[AnySignlessInteger, AnyFloat], "Integer or Float">;

def GPU_AllReduceOperationAttr : EnumAttr<GPU_Dialect, GPU_AllReduceOperation,
                                          "all_reduce_op">;

def GPU_AllReduceOp : GPU_Op<"all_reduce",
    [SameOperandsAndResultType, IsolatedFromAbove]> {
  let summary = "Reduce values among workgroup.";
  let description = [{
    The `all_reduce` op reduces the value of every work item across a local
    workgroup. The result is equal for all work items of a workgroup.

```
- **EN**: Introduces declarations for `AnyIntegerOrFloat`, `GPU_AllReduceOperationAttr`, `GPU_AllReduceOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AnyIntegerOrFloat`, `GPU_AllReduceOperationAttr`, `GPU_AllReduceOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1181-1191
```tablegen
    For example, both

    ```mlir
    %1 = gpu.all_reduce add %0 {} : (f32) -> (f32)
    %2 = gpu.all_reduce %0 {
    ^bb(%lhs : f32, %rhs : f32):
      %sum = arith.addf %lhs, %rhs : f32
      "gpu.yield"(%sum) : (f32) -> ()
    } : (f32) -> (f32)
    ```

```
- **EN**: Implements logic around `bb`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `bb` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 1192-1203
```tablegen
    compute the sum of each work item's %0 value. The first version specifies
    the accumulation as operation, whereas the second version specifies the
    accumulation as code region. The reduction operation must be one of:
    *  Integer types: `add`, `mul`, `minui`, `minsi`, `maxui`, `maxsi`, `and`,
       `or`, `xor`
    *  Floating point types: `add`, `mul`, `minnumf`, `maxnumf`, `minimumf`,
       `maximumf`

    If `uniform` flag is set either none or all work items of a workgroup
    need to execute this op in convergence.
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1204-1215
```tablegen
  let arguments = (ins
    AnyIntegerOrFloat:$value,
    OptionalAttr<GPU_AllReduceOperationAttr>:$op,
    UnitAttr:$uniform
  );
  let results = (outs AnyIntegerOrFloat:$result);

  let regions = (region AnyRegion:$body);
  let assemblyFormat = [{ custom<AllReduceOperation>($op) $value
                          (`uniform` $uniform^)? $body attr-dict
                          `:` functional-type(operands, results) }];

```
- **EN**: Implements logic around `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1216-1228
```tablegen
  let hasFolder = 1;
  let hasRegionVerifier = 1;
}

def AnyIntegerOrFloatOr1DVector :
  AnyTypeOf<[AnyIntegerOrFloat, FixedVectorOfRankAndType<[1], [AnyIntegerOrFloat]>]>;

def GPU_SubgroupReduceOp : GPU_Op<"subgroup_reduce", [SameOperandsAndResultType, NoMemoryEffect]> {
  let summary = "Reduce values among subgroup.";
  let description = [{
    The `subgroup_reduce` op reduces the values of lanes (work items) across a
    subgroup.

```
- **EN**: Introduces declarations for `AnyIntegerOrFloatOr1DVector`, `GPU_SubgroupReduceOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AnyIntegerOrFloatOr1DVector`, `GPU_SubgroupReduceOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1229-1238
```tablegen
    The subgroup is divided into clusters starting at lane index 0. Within each
    cluster, there are `size` lanes, and the lane index advances by `stride`.
    A reduction is done for each cluster in parallel: every lane in the cluster
    is reduced, and the result is equal for all lanes in the cluster. If `size`
    is omitted, there is a single cluster covering the entire subgroup. If
    `stride` is omitted, the stride is 1 (the cluster's lanes are contiguous).

    When the reduced value is of a vector type, each vector element is reduced
    independently. Only 1-d vector types are allowed.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1239-1250
```tablegen
    Example:

    ```mlir
    %1 = gpu.subgroup_reduce add %a : (f32) -> f32
    %2 = gpu.subgroup_reduce add %b : (vector<4xf16>) -> vector<4xf16>
    %3 = gpu.subgroup_reduce add %c cluster(size = 4) : (f32) -> f32
    %3 = gpu.subgroup_reduce add %c cluster(size = 4, stride = 2) : (f32) -> f32
    ```

    If `uniform` flag is set either none or all lanes of a subgroup need to execute
    this op in convergence.

```
- **EN**: Implements logic around `cluster`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `cluster` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1251-1266
```tablegen
    The reduction operation must be one of:
    *  Integer types: `add`, `mul`, `minui`, `minsi`, `maxui`, `maxsi`, `and`,
       `or`, `xor`
    *  Floating point types: `add`, `mul`, `minnumf`, `maxnumf`, `minimumf`,
       `maximumf`
  }];

  let arguments = (ins
    AnyIntegerOrFloatOr1DVector:$value,
    GPU_AllReduceOperationAttr:$op,
    UnitAttr:$uniform,
    OptionalAttr<I32Attr>:$cluster_size,
    DefaultValuedAttr<I32Attr,"1">:$cluster_stride
  );
  let results = (outs AnyIntegerOrFloatOr1DVector:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1267-1286
```tablegen
  let builders = [
    OpBuilder<(ins "Value":$value,
               "::mlir::gpu::AllReduceOperation":$op,
               "bool":$uniform), [{
      build($_builder, $_state, value, op, uniform, std::nullopt);
    }]>,
    OpBuilder<(ins "Value":$value,
               "::mlir::gpu::AllReduceOperation":$op,
               "bool":$uniform,
               "std::optional<uint32_t>":$cluster_size), [{
      build($_builder, $_state, value, op, uniform,
            cluster_size ? $_builder.getI32IntegerAttr(*cluster_size) : nullptr);
    }]>,
    OpBuilder<(ins "Value":$value,
               "::mlir::gpu::AllReduceOperation":$op,
               "bool":$uniform,
               "std::optional<uint32_t>":$cluster_size,
               "uint32_t":$cluster_stride), [{
      build($_builder, $_state, value, op, uniform,
            cluster_size ? $_builder.getI32IntegerAttr(*cluster_size) : nullptr,
```
- **EN**: Implements logic around `OpBuilder`, `build`, `getI32IntegerAttr`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder`, `build`, `getI32IntegerAttr` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 1287-1296
```tablegen
            cluster_stride);
    }]>
  ];

  let assemblyFormat = [{ custom<AllReduceOperation>($op) $value
                          (`uniform` $uniform^)?
                          (`cluster` `(` `size` `=` $cluster_size^ (`,` `stride` `=` $cluster_stride^)? `)`)?
                          attr-dict
                          `:` functional-type(operands, results) }];

```
- **EN**: Implements logic around `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1297-1306
```tablegen
  let hasFolder = 1;
  let hasVerifier = 1;
}

def GPU_ShuffleOpXor  : I32EnumAttrCase<"XOR",  0, "xor">;
def GPU_ShuffleOpDown : I32EnumAttrCase<"DOWN", 1, "down">;
def GPU_ShuffleOpUp   : I32EnumAttrCase<"UP",   2, "up">;
def GPU_ShuffleOpIdx  : I32EnumAttrCase<"IDX",  3, "idx">;

def GPU_ShuffleMode : I32EnumAttr<"ShuffleMode",
```
- **EN**: Introduces declarations for `GPU_ShuffleOpXor`, `GPU_ShuffleOpDown`, `GPU_ShuffleOpUp`, `GPU_ShuffleOpIdx`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ShuffleOpXor`, `GPU_ShuffleOpDown`, `GPU_ShuffleOpUp`, `GPU_ShuffleOpIdx`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1307-1316
```tablegen
    "Indexing modes supported by gpu.shuffle.",
    [
      GPU_ShuffleOpXor, GPU_ShuffleOpUp, GPU_ShuffleOpDown, GPU_ShuffleOpIdx,
    ]> {
  let genSpecializedAttr = 0;
  let cppNamespace = "::mlir::gpu";
}
def GPU_ShuffleModeAttr : EnumAttr<GPU_Dialect, GPU_ShuffleMode,
                                   "shuffle_mode">;

```
- **EN**: Introduces declarations for `GPU_ShuffleModeAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ShuffleModeAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1317-1329
```tablegen
def GPU_ShuffleOp : GPU_Op<
    "shuffle", [Pure, AllTypesMatch<["value", "shuffleResult"]>]>,
    Arguments<(ins AnyIntegerOrFloatOr1DVector:$value, I32:$offset, I32:$width,
               GPU_ShuffleModeAttr:$mode)>,
    Results<(outs AnyIntegerOrFloatOr1DVector:$shuffleResult, I1:$valid)> {
  let summary = "Shuffles values within a subgroup.";
  let description = [{
    The "shuffle" op moves values across lanes in a subgroup (a.k.a., local
    invocation) within the same subgroup. The `width` argument specifies the
    number of lanes that participate in the shuffle, and must be uniform
    across all lanes. Further, the first `width` lanes of the subgroup must
    be active.

```
- **EN**: Introduces declarations for `GPU_ShuffleOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ShuffleOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1330-1341
```tablegen
    The intepretation of the `offset` arguments depends on the selected
    `mode`.

    Returns the `shuffleResult` and `true` if the current lane id is smaller
    than `width`, and an unspecified value and `false` otherwise.

    `xor` example:

    ```mlir
    %1, %2 = gpu.shuffle xor %0, %offset, %width : f32
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1342-1351
```tablegen
    For lane `k`, returns the value `%0` from lane `k ^ offset`. Every lane
    trades value with exactly one other lane.

    `down` example:

    ```mlir
    %cst1 = arith.constant 1 : i32
    %3, %4 = gpu.shuffle down %0, %cst1, %width : f32
    ```

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1352-1361
```tablegen
    For lane `k`, returns the value from lane `(k + cst1)`. If `(k + cst1)` is
    bigger than or equal to `width`, the value is poison and `valid` is `false`.

    `up` example:

    ```mlir
    %cst1 = arith.constant 1 : i32
    %5, %6 = gpu.shuffle up %0, %cst1, %width : f32
    ```

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1362-1371
```tablegen
    For lane `k`, returns the value from lane `(k - cst1)`. If `(k - cst1)` is
    smaller than `0`, the value is poison and `valid` is `false`.

    `idx` example:

    ```mlir
    %cst0 = arith.constant 0 : i32
    %7, %8 = gpu.shuffle idx %0, %cst0, %width : f32
    ```

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1372-1385
```tablegen
    Broadcasts the value from lane 0 to all lanes.
  }];

  let assemblyFormat = [{
    $mode $value `,` $offset `,` $width attr-dict `:` type($value)
  }];

  let builders = [
    // Helper function that creates a shuffle with constant offset/width.
    OpBuilder<(ins "Value":$value, "int32_t":$offset, "int32_t":$width,
                   "ShuffleMode":$mode)>
  ];
}

```
- **EN**: Implements logic around `type`, `OpBuilder`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type`, `OpBuilder` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1386-1399
```tablegen
def GPU_RotateOp : GPU_Op<
    "rotate", [Pure, AllTypesMatch<["value", "rotateResult"]>]>,
    Arguments<(ins AnyIntegerOrFloatOr1DVector:$value,
                   ConfinedAttr<I32Attr, [IntMinValue<0>]>:$offset,
                   ConfinedAttr<I32Attr, [IntPowerOf2]>:$width)>,
    Results<(outs AnyIntegerOrFloatOr1DVector:$rotateResult, I1:$valid)> {
  let summary = "Rotate values within a subgroup.";
  let description = [{
    The "rotate" op moves values across lanes in a subgroup (a.k.a., local
    invocations) within the same subgroup. The `width` attribute specifies the
    number of lanes that participate in the rotation, and must be uniform across
    all participating lanes. Further, the first `width` lanes of the subgroup
    must be active.

```
- **EN**: Introduces declarations for `GPU_RotateOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_RotateOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1400-1409
```tablegen
    `width` must be a power of two, and `offset` must be in the range
    `[0, width)`.

    Return the `rotateResult` of the invocation whose id within the group is
    calculated as follows:

    ```mlir
    Invocation ID = ((LaneId + offset) & (width - 1)) + (LaneId & ~(width - 1))
    ```

```
- **EN**: Implements logic around `~`.
- **CN**: 围绕 `~` 实现具体逻辑。

### Lines 1410-1421
```tablegen
    Returns the `rotateResult` and `true` if the current lane id is smaller than
    `width`, and poison value and `false` otherwise.

    example:

    ```mlir
    %1, %2 = gpu.rotate %0, 1, 16 : f32
    ```

    For lane `k`, returns the value from lane `(k + cst1) % width`.
  }];

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1422-1435
```tablegen
  let assemblyFormat = [{
    $value `,` $offset `,` $width attr-dict `:` type($value)
  }];

  let hasVerifier = 1;
}

def GPU_BarrierOp : GPU_Op<"barrier">,
    Arguments<(ins OptionalAttr<GPU_AddressSpaceAttrArray> :$address_spaces)> {
  let summary = "Synchronizes all work items of a workgroup.";
  let description = [{
    The `barrier` op synchronizes all work items of a workgroup. It is used
    to coordinate communication between the work items of the workgroup.

```
- **EN**: Introduces declarations for `GPU_BarrierOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_BarrierOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1436-1445
```tablegen
    ```mlir
    gpu.barrier
    ```

    waits until all work items in the workgroup have reached the operation
    and all memory accesses made by these work items prior to the op are
    visible to all work items in the workgroup. Data hazards between work items
    accessing the same memory can be avoided by synchronizing work items
    in-between these accesses.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1446-1463
```tablegen
    If the `memfence` attribute is specified, the set of memory accesses that must
    by completed after the barrier resolves is limited to only those accesses that
    read from or write to the specified address spaces (though accesses to other
    address spaces may be completed as well, especially if a particular combination
    of address spaces is not supported on a given backend). In particular,
    specifying `memfence []` creates a barrier that is not required to affect
    the visibility of any memory operations and is purely used for synchronizing
    work items.

    ```mlir
    // Only workgroup address spaces accesses required to be visible.
    gpu.barrier memfence [#gpu.address_space<workgroup>]
    // No memory accesses required to be visible.
    gpu.barrier memfence []
    // All memory accesses required to be visible.
    gpu.barrier
    ```

```
- **EN**: Implements logic around `spaces`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `spaces` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1464-1474
```tablegen
    Either none or all work items of a workgroup need to execute this op
    in convergence.
  }];
  let assemblyFormat = "(`memfence` $address_spaces^)? attr-dict";
  let hasCanonicalizer = 1;
  let builders = [OpBuilder<(
                      ins CArg<"std::optional<::mlir::gpu::AddressSpace>",
                               "std::nullopt">:$addressSpace)>,
                  OpBuilder<(ins "Value":$memrefToFence)>];
}

```
- **EN**: Declares APIs or declarative rules around `OpBuilder`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 声明与 `OpBuilder` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1475-1484
```tablegen
def GPU_GPUModuleOp : GPU_Op<"module", [
      IsolatedFromAbove, DataLayoutOpInterface, HasDefaultDLTIDataLayout,
      NoRegionArguments, SymbolTable, Symbol] # GraphRegionNoTerminator.traits> {
  let summary = "A top level compilation unit containing code to be run on a GPU.";
  let description = [{
    GPU module contains code that is intended to be run on a GPU. A host device
    can launch this code through a gpu.launc_func that creates a fully
    qualified symbol through the gpu.module's symbol and a gpu.func symbol
    contained in the gpu.module.

```
- **EN**: Introduces declarations for `GPU_GPUModuleOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_GPUModuleOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1485-1497
```tablegen
    The module's top-level scope is modeled by a single region with a single
    block. GPU modules are required to have a name that is used for symbol
    resolution by the gpu.launch_func operation.

    Using an op with a region to define a GPU module enables "embedding" GPU
    modules with SIMT execution models in other dialects in a clean manner and
    allows filtering of code regions to execute passes on only code intended to
    or not intended to be run on the separate device.

    Modules can contain zero or more target attributes. These attributes encode
    how to transform modules into binary strings and are used by the
    `gpu-module-to-binary` pass to transform modules into GPU binaries.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1498-1517
```tablegen
    Modules can contain an optional `OffloadingTranslationAttr` attribute. This
    attribute will be used during the `gpu-module-to-binary` pass to specify the
    `OffloadingTranslationAttr` used when creating the `gpu.binary` operation.

    ```
    gpu.module @symbol_name {
      gpu.func {}
        ...
    }
    // Module with offloading handler and target attributes.
    gpu.module @symbol_name2 <#gpu.select_object<1>> [
        #nvvm.target,
        #rocdl.target<chip = "gfx90a">] {
      gpu.func {}
        ...
    }
    ```
  }];
  let builders = [
    OpBuilder<(ins "StringRef":$name,
```
- **EN**: Implements logic around `OpBuilder`; this block packages logic as an MLIR pass or pass-related API; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1518-1536
```tablegen
                   CArg<"ArrayAttr", "{}">:$targets,
                   CArg<"Attribute", "{}">:$handler)>,
    OpBuilder<(ins "StringRef":$name,
                   "ArrayRef<Attribute>":$targets,
                   CArg<"Attribute", "{}">:$handler)>
  ];

  let arguments = (ins
      SymbolNameAttr:$sym_name,
      OptionalAttr<GPUNonEmptyTargetArrayAttr>:$targets,
      OptionalAttr<OffloadingTranslationAttr>:$offloadingHandler);
  let regions = (region SizedRegion<1>:$bodyRegion);
  let assemblyFormat = [{
    $sym_name
    (`<` $offloadingHandler^ `>`)?
    ($targets^)?
    attr-dict-with-keyword $bodyRegion
  }];

```
- **EN**: Implements logic around `OpBuilder`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1537-1548
```tablegen
  // We need to ensure the block inside the region is properly terminated;
  // the auto-generated builders do not guarantee that.
  let skipDefaultBuilders = 1;

  let extraClassDeclaration = [{
    /// Checks if `target` is in the `targets` list.
    bool hasTarget(Attribute target);

    /// Sets the targets of the module.
    void setTargets(ArrayRef<TargetAttrInterface> targets);
  }];

```
- **EN**: Implements logic around `hasTarget`, `setTargets`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `hasTarget`, `setTargets` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1549-1561
```tablegen
  let hasVerifier = 1;
}

def GPU_BinaryOp : GPU_Op<"binary", [Symbol]>, Arguments<(ins
      SymbolNameAttr:$sym_name,
      OptionalAttr<OffloadingTranslationAttr>:$offloadingHandler,
      ConfinedAttr<GPUObjectArrayAttr, [ArrayMinCount<1>]>:$objects)
    > {
  let summary = "An Op for storing serialized GPU binary objects.";
  let description = [{
    GPU binaries provide a semantic mechanism for storing GPU objects,
    e.g. the result of compiling a GPU module to an object file.

```
- **EN**: Introduces declarations for `GPU_BinaryOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_BinaryOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1562-1571
```tablegen
    This operation has 3 arguments:
     - The name of the binary.
     - An optional attribute implementing the offloading LLVM translation interface.
     - An array of GPU object attributes.

    During translation, the offloading attribute will be called for translating
    GPU `binary` and `launch_func` operations. The default offloading handler is:
    `#gpu.select_object`, this handler selects the first object from the array
    and embeds it as a string.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1572-1591
```tablegen
    Examples:
    ```
      // Selects the first object.
      gpu.binary @myobject [#gpu.object<...>, #gpu.object<...>]
      // Uses the `#foo.my_handler` for handling the binary during translation.
      gpu.binary @myobject <#foo.my_handler> [#gpu.object<...>, #gpu.object<...>]
      // Selects the object with the `#rocdl.target` target attribute.
      gpu.binary @myobject <#gpu.select_object<#rocdl.target>> [#gpu.object<...>, #gpu.object<#rocdl.target, ...>]
    ```
  }];
  let builders = [
    OpBuilder<(ins "StringRef":$name,
                   "Attribute":$offloadingHandler,
                   "ArrayAttr":$objects)>,
    OpBuilder<(ins "StringRef":$name,
                   "Attribute":$offloadingHandler,
                   "ArrayRef<Attribute>":$objects)>
  ];
  let skipDefaultBuilders = 1;
  let assemblyFormat = [{
```
- **EN**: Implements logic around `OpBuilder`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1592-1601
```tablegen
    $sym_name custom<OffloadingHandler>($offloadingHandler) attr-dict $objects
  }];
}

def GPU_HostRegisterOp : GPU_Op<"host_register">,
    Arguments<(ins AnyUnrankedMemRef:$value)> {
  let summary = "Registers a memref for access from device.";
  let description = [{
    This op maps the provided host buffer into the device address space.

```
- **EN**: Introduces declarations for `GPU_HostRegisterOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_HostRegisterOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1602-1612
```tablegen
    This operation may not be supported in every environment, there is not yet a
    way to check at runtime whether this feature is supported.

    Writes from the host are guaranteed to be visible to device kernels that are
    launched afterwards. Writes from the device are guaranteed to be visible on
    the host after synchronizing with the device kernel completion.
  }];

  let assemblyFormat = "$value attr-dict `:` type($value)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1613-1622
```tablegen
def GPU_HostUnregisterOp : GPU_Op<"host_unregister">,
    Arguments<(ins AnyUnrankedMemRef:$value)> {
  let summary = "Unregisters a memref for access from device.";
  let description = [{
      This op unmaps the provided host buffer from the device address space.

      This operation may not be supported in every environment, there is not yet a
          way to check at runtime whether this feature is supported.
  }];

```
- **EN**: Introduces declarations for `GPU_HostUnregisterOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_HostUnregisterOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1623-1637
```tablegen
  let assemblyFormat = "$value attr-dict `:` type($value)";
}

def GPU_WaitOp : GPU_Op<"wait", [GPU_AsyncOpInterface]> {
  let summary = "Wait for async gpu ops to complete.";
  let description = [{
    This op synchronizes the host or the device with a list of dependent ops.

    If the op contains the `async` keyword, it returns a new async token which
    is synchronized with the op arguments. This new token is merely a shortcut
    to the argument list, and one could replace the uses of the result with the
    arguments for the same effect. The async version of this op is primarily
    used to make each async token have a single use during lowering and
    thereby make forks in async execution explicit. Example usage:

```
- **EN**: Introduces declarations for `GPU_WaitOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_WaitOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1638-1651
```tablegen
    ```mlir
    %t0 = gpu.foo async : !gpu.async.token
    %t1 = gpu.bar async : !gpu.async.token
    %t2 = gpu.wait async [%t0, %t1]
    // gpu.baz doesn't run until gpu.foo and gpu.bar have both completed, just
    // as if the async dependencies were [%t0, %t1].
    %t3 = gpu.baz async [%t2]
    ```

    If the op does not contain the `async` keyword, it does not return a new
    async token but blocks until all ops producing the async dependency tokens
    finished execution. All dependent memory operations are visible to the host
    once this op completes. Example usage:

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1652-1662
```tablegen
    ```mlir
    %t0 = gpu.foo async : !gpu.async.token
    %t1 = gpu.bar async : !gpu.async.token
    // The gpu.wait op blocks until gpu.foo and gpu.bar have completed.
    gpu.wait [%t0, %t1]
    ```
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1663-1674
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies) attr-dict
  }];

  let hasCanonicalizer = 1;
}

def GPU_AllocOp : GPU_Op<"alloc", [
    GPU_AsyncOpInterface,
    AttrSizedOperandSegments
  ]> {

```
- **EN**: Introduces declarations for `GPU_AllocOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_AllocOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1675-1686
```tablegen
  let summary = "GPU memory allocation operation.";
  let description = [{
    The `gpu.alloc` operation allocates a region of memory on the GPU. It is
    similar to the `memref.alloc` op, but supports asynchronous GPU execution.

    The op does not execute before all async dependencies have finished
    executing.

    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it also returns a !gpu.async.token.

```
- **EN**: Implements logic around `asynchronously`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `asynchronously` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1687-1696
```tablegen
    If the `host_shared` keyword is present, the memory will be allocated in a
    memory accessible both on host and on device.

    Example:

    ```mlir
    %memref, %token = gpu.alloc async [%dep] host_shared (%width) : memref<64x?xf32, 1>
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `host_shared`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `host_shared` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1697-1706
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   Variadic<Index>:$dynamicSizes, Variadic<Index>:$symbolOperands,
                   UnitAttr:$hostShared);
  let results = (outs Res<AnyMemRef, "", [MemAllocAt<0, FullEffect>]>:$memref,
                 Optional<GPU_AsyncToken>:$asyncToken);

  let extraClassDeclaration = [{
    MemRefType getType() { return ::llvm::cast<MemRefType>(getMemref().getType()); }
  }];

```
- **EN**: Implements logic around `getType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1707-1716
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies) (` ` `host_shared` $hostShared^)? ` `
    `(` $dynamicSizes `)` (`` `[` $symbolOperands^ `]`)? attr-dict `:` type($memref)
  }];

  let hasVerifier = 1;
  let hasCanonicalizer = 1;
}

def GPU_DeallocOp : GPU_Op<"dealloc", [GPU_AsyncOpInterface]> {
```
- **EN**: Introduces declarations for `GPU_DeallocOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_DeallocOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1717-1727
```tablegen

  let summary = "GPU memory deallocation operation";

  let description = [{
    The `gpu.dealloc` operation frees the region of memory referenced by a
    memref which was originally created by the `gpu.alloc` operation. It is
    similar to the `memref.dealloc` op, but supports asynchronous GPU execution.

    The op does not execute before all async dependencies have finished
    executing.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1728-1738
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token.

    Example:

    ```mlir
    %token = gpu.dealloc async [%dep] %memref : memref<8x64xf32, 1>
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `asynchronously`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `asynchronously` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1739-1748
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   Arg<AnyMemRef, "", [MemFreeAt<0, FullEffect>]>:$memref);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $memref attr-dict `:` type($memref)
  }];
}

```
- **EN**: Implements logic around `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1749-1758
```tablegen
def GPU_MemcpyOp : GPU_Op<"memcpy", [GPU_AsyncOpInterface]> {

  let summary = "GPU memcpy operation";

  let description = [{
    The `gpu.memcpy` operation copies the content of one memref to another.

    The op does not execute before all async dependencies have finished
    executing.

```
- **EN**: Introduces declarations for `GPU_MemcpyOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_MemcpyOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1759-1769
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token.

    Example:

    ```mlir
    %token = gpu.memcpy async [%dep] %dst, %src : memref<?xf32, 1>, memref<?xf32>
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `asynchronously`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `asynchronously` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1770-1783
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   Arg<AnyMemRef, "", [MemWriteAt<0, FullEffect>]>:$dst,
                   Arg<AnyMemRef, "", [MemReadAt<0, FullEffect>]>:$src);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $dst`,` $src `:` type($dst)`,` type($src) attr-dict
  }];
  let hasFolder = 1;
  let hasVerifier = 1;
  let hasCanonicalizer = 1;
}

```
- **EN**: Implements logic around `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1784-1794
```tablegen
def GPU_MemsetOp : GPU_Op<"memset",
  [GPU_AsyncOpInterface, AllElementTypesMatch<["dst", "value"]>]> {

  let summary = "GPU memset operation";

  let description = [{
    The `gpu.memset` operation sets the content of memref to a scalar value.

    The op does not execute before all async dependencies have finished
    executing.

```
- **EN**: Introduces declarations for `GPU_MemsetOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_MemsetOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1795-1805
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token.

    Example:

    ```mlir
    %token = gpu.memset async [%dep] %dst, %value : memref<?xf32, 1>, f32
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `asynchronously`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `asynchronously` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1806-1817
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   Arg<AnyMemRef, "", [MemWriteAt<0, FullEffect>]>:$dst,
                   Arg<AnyType, "">:$value);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $dst`,` $value `:` type($dst)`,` type($value) attr-dict
  }];
  let hasFolder = 1;
}

```
- **EN**: Implements logic around `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1818-1829
```tablegen
def GPU_SetDefaultDeviceOp : GPU_Op<"set_default_device",
                                    [MemoryEffects<[MemWrite]>]>,
    Arguments<(ins I32:$devIndex)> {
  let summary = "Set default GPU for operations after this by index";
  let description = [{
    Operation that sets the current default GPU, using a zero-based index
    into the set of GPUs on the system. The default GPU setting may be
    thread-local.
  }];
  let assemblyFormat = "attr-dict $devIndex";
}

```
- **EN**: Introduces declarations for `GPU_SetDefaultDeviceOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SetDefaultDeviceOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1830-1839
```tablegen
// Promises IndexedAccessOpInterface.
def GPU_SubgroupMmaLoadMatrixOp : GPU_Op<"subgroup_mma_load_matrix",
    [MemoryEffects<[MemRead]>]>{

  let summary = "GPU warp synchronous matrix load";

  let description = [{
    The `gpu.subgroup_mma_load_matrix` operation loads a matrix collectively
    using all the threads in a subgroup.

```
- **EN**: Introduces declarations for `GPU_SubgroupMmaLoadMatrixOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupMmaLoadMatrixOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1840-1850
```tablegen
    This operation takes a memref as its first operand: it is the source matrix
    from which data is to be loaded. The op returns a `!gpu.mma_matrix`. The
    source memref can be in global memory or shared memory. The load address is
    determined using `indices`. The matrix being loaded into is the result.  The
    `leadDimension` attribute specifies the leading dimension size of the source
    matrix which eventually allows the lowering to determine the size of each
    row.  If the `transpose` attribute is present then the op does a transposed load.

    The memory indices along each dimension must be in-bounds for that dimension
    as with an ordinary `memref.load`.

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1851-1865
```tablegen
    For integer types, the resulting `!gpu.mma_matrix` type needs to specify the
    signedness of the data if the matrix type is an `A` or `B` operand for
    `gpu.subgroup_mma_compute`.

    This op is often meant to be used along with `gpu.subgroup_mma_store_matrix` and
    `gpu.subgroup_mma_compute`.

    Example:

    ```mlir
     %0 = gpu.subgroup_mma_load_matrix src[%i,%j] : {leadDimension = 32 : i32}
          : memref<32x32xf16, 3>, !gpu.mma_matrix<16x16xf16, "AOp">
    ```
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1866-1879
```tablegen
  let arguments = (ins Arg<GPU_MMAMemRef, "",
                          [MemReadAt<0, FullEffect>]>:$srcMemref,
                  Variadic<Index>:$indices,
                  IndexAttr:$leadDimension,
                  OptionalAttr<UnitAttr>:$transpose);

  let results = (outs GPU_MMAMatrix:$res);

  let assemblyFormat = [{
    $srcMemref`[`$indices`]` attr-dict `:` type($srcMemref) `->` type($res)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1880-1889
```tablegen
// Promises IndexedAccessOpInterface.
def GPU_SubgroupMmaStoreMatrixOp : GPU_Op<"subgroup_mma_store_matrix",
    [MemoryEffects<[MemWrite]>]>{

  let summary = "GPU warp synchronous matrix store";

  let description = [{
    The `gpu.subgroup_mma_store_matrix` operation stores a matrix collectively
    using all the threads in a subgroup.

```
- **EN**: Introduces declarations for `GPU_SubgroupMmaStoreMatrixOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupMmaStoreMatrixOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1890-1899
```tablegen
    This operation takes a `!gpu.mma_matrix` and a memref as operands.
    `!gpu.mma_matrix` is the source value containing the data to be stored into the
    destination memref which can be in global or shared memory.  The store address
    is determined using the indices provided. The `leadDimension` attribute
    specifies the leading dimension of the destination matrix. If the
    `transpose` attribute is present then the op does a transposed store.

    This op is often meant to be used along with `gpu.subgroup_mma_load_matrix` and
    `gpu.subgroup_mma_compute`.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1900-1910
```tablegen
    The memory indices along each dimension must be in-bounds for that dimension
    as with an ordinary `memref.load`.

    Example:

    ```mlir
    gpu.subgroup_mma_store_matrix %D, %sg[%i,%j] : { leadDimension = 32 : i32}
                    : !gpu.mma_matrix<16x16xf16, "COp">, memref<32x32xf16, 3>
    ```
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1911-1922
```tablegen
  let arguments = (ins Arg<MMAMatrixOf<[SI8, UI8, I32, F16, F32, F64]>>:$src,
                  Arg<GPU_MMAMemRef, "",[MemWriteAt<0, FullEffect>]>:$dstMemref,
                  Variadic<Index>:$indices,
                  IndexAttr:$leadDimension,
                  OptionalAttr<UnitAttr>:$transpose);

  let assemblyFormat = [{
    $src`,` $dstMemref`[`$indices`]` attr-dict `:` type($src)`,` type($dstMemref)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1923-1941
```tablegen
def GPU_SubgroupMmaComputeOp
    : GPU_Op<"subgroup_mma_compute", [Pure, AllTypesMatch<["opC", "res"]>]> {

  let summary = "GPU warp synchronous matrix multiply accumulate";

  let description = [{
    The `gpu.subgroup_mma_compute` operation performs a matrix-multiply accumulate (mma)
    operation using all the threads in a subgroup.

    This operation takes three `!gpu.mma_matrix`s as arguments: these hold `A`,
    `B` and `C`operands for the mma operation. The operation performed is represented
    as `C += A * B`. The op returns a `!gpu.mma_matrix` which contains the result of
    the operation held by all threads in a subgroup. `a_transpose` or
    `b_transpose` if present, signify that the respective operand was loaded in a
    transposed manner. The transpose operands are required to map to correct
    underlying intrisics but they currently do not seem to affect correctness
    even if they are absent given that the operands were loaded correctly using
    the `transpose` attribute in `gpu.subgroup_mma_load_matrix` op.

```
- **EN**: Introduces declarations for `GPU_SubgroupMmaComputeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupMmaComputeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1942-1957
```tablegen
    For integer types, the `A` and `B` matrices carry their signedness with their
    types. The accumulator type is expected to be signless and imply a signed integer
    with a greater width than the other two operands.

    This op is meant to be used along with `gpu.subgroup_mma_store_matrix` and
    `gpu.subgroup_mma_load_matrix` ops.

    Example:

    ```mlir
    %D = gpu.subgroup_mma_compute_matrix %A, %B, %C :
      !gpu.mma_matrix<16x16xf16, "AOp">, !gpu.mma_matrix<16x16xf16, "BOp">>
      -> !gpu.mma_matrix<16x16xf16, "COp">
    ```
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1958-1971
```tablegen
  let arguments = (ins Arg<MMAMatrixOf<[SI8, UI8, F16, F32, F64]>>:$opA,
                  Arg<MMAMatrixOf<[SI8, UI8, F16, F32, F64]>>:$opB,
                  Arg<MMAMatrixOf<[I32, F16, F32, F64]>>:$opC,
                  OptionalAttr<UnitAttr>:$a_transpose,
                  OptionalAttr<UnitAttr>:$b_transpose);

  let results = (outs GPU_MMAMatrix : $res);

  let assemblyFormat = [{
    $opA`,` $opB`,` $opC attr-dict `:` type($opA)`,` type($opB) `->` type($res)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1972-1983
```tablegen
def GPU_SubgroupMmaConstantMatrixOp : GPU_Op<"subgroup_mma_constant_matrix",
    [Pure,
     TypesMatchWith<"value type matches element type of mma_matrix",
                    "res", "value",
                    "::llvm::cast<gpu::MMAMatrixType>($_self).getElementType()">]>{

  let summary = "GPU warp synchronous constant matrix";

  let description = [{
    The `gpu.subgroup_mma_constant_matrix` creates a `!gpu.mma_matrix` with
    constant elements.

```
- **EN**: Introduces declarations for `GPU_SubgroupMmaConstantMatrixOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupMmaConstantMatrixOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1984-1993
```tablegen
    The operation takes a scalar input and return a `!gpu.mma_matrix` where
    each element of is equal to the operand constant. The destination
    mma_matrix type must have elememt type equal to the constant type. Since
    the layout of `!gpu.mma_matrix` is opaque this only support setting all the
    elements to the same value.

    This op is meant to be used along with `gpu.subgroup_mma_compute`.

    Example:

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1994-2003
```tablegen
    ```mlir
     %0 = gpu.subgroup_mma_constant_matrix %a :
       !gpu.mma_matrix<16x16xf16, "AOp">
     %1 = gpu.subgroup_mma_constant_matrix %b :
       !gpu.mma_matrix<16x16xf32, "COp">
    ```
  }];

  let arguments = (ins AnyTypeOf<[SI8, UI8, I32, F16, F32]>:$value);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2004-2016
```tablegen
  let results = (outs GPU_MMAMatrix:$res);

  let extraClassDeclaration = [{
    gpu::MMAMatrixType getType() {
      return ::llvm::cast<gpu::MMAMatrixType>(getRes().getType());
    }
  }];

  let assemblyFormat = [{
    $value attr-dict `:` type($res)
  }];
}

```
- **EN**: Implements logic around `getType`, `MMAMatrixType>`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getType`, `MMAMatrixType>`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2017-2028
```tablegen
def GPU_SubgroupMmaExtractThreadLocalOp : GPU_Op<"subgroup_mma_extract_thread_local",
    [Pure,
     TypesMatchWith<"value type matches element type of mma_matrix",
                    "matrix", "res",
                    "::llvm::cast<gpu::MMAMatrixType>($_self).getElementType()">]>{

  let summary = "Extract a value from GPU warp by invocation and indices";

  let description = [{
    The `gpu.subgroup_mma_extract_thread_local` operation extracts a value from `!gpu.mma_matrix`
    that is stored at subgroup level.

```
- **EN**: Introduces declarations for `GPU_SubgroupMmaExtractThreadLocalOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupMmaExtractThreadLocalOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2029-2042
```tablegen
    This operation takes `!gpu.mma_matrix` as its first operand. It is the source
    matrix across a subgroup. The op returns a scalar value stored in the invocation
    in the subgroup.

    Since `matrix` is packed into the the threads within a subgroup, `indices` are
    the indices into the values stored by each thread. That is, an index of 0 (or [0, 0])
    does not necessarily refer to the first element of the matrix, but the first element
    that a particular thread holds.

    The mapping of matrix elements to threads is not defined by this operation and may
    not be defined by some lowerings (such as the lowering to SPIR-V). However, if the
    size of the subgroup is S, then `subgroup_mma_extract_thread_local` at each index in
    `[0, (M * N) / S)` will have the entire matrix extracted across the subgroup.

```
- **EN**: Implements logic around `lowerings`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `lowerings` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2043-2052
```tablegen
    Example:

    ```mlir
    %c0 = arith.constant 0 : index
    %val = gpu.subgroup_mma_extract_thread_local %m[%c0] : !gpu.mma_matrix<16x16xf32, "AOp"> -> f32
    ```
  }];

  let arguments = (ins GPU_MMAMatrix:$matrix, Variadic<Index>:$indices);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2053-2065
```tablegen
  let results = (outs AnyIntegerOrFloat:$res);

  let assemblyFormat = [{
    $matrix`[`$indices`]` attr-dict `:` type($matrix) `->` type($res)
  }];
}

def GPU_SubgroupMmaInsertThreadLocalOp : GPU_Op<"subgroup_mma_insert_thread_local",
    [Pure,
     TypesMatchWith<"value type matches element type of mma_matrix",
                    "matrix", "value",
                    "::llvm::cast<gpu::MMAMatrixType>($_self).getElementType()"> ]>{

```
- **EN**: Introduces declarations for `GPU_SubgroupMmaInsertThreadLocalOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupMmaInsertThreadLocalOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2066-2079
```tablegen
  let summary = "Insert a value into GPU warp by invocation and indices";

  let description = [{
    The `gpu.subgroup_mma_insert_thread_local` operation inserts a value to `!gpu.mma_matrix`
    that is stored at subgroup level.

    This operation takes scalar value as its first operand and `!gpu.mma_matrix`
    as its second operand. The op inserts the scalar value to the matrix.

    Since `matrix` is packed into the the threads within a subgroup, `indices` are
    the indices into the values stored by each thread. That is, an index of 0 (or [0, 0])
    does not necessarily refer to the first element of the matrix, but the first element
    that a particular thread holds.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2080-2095
```tablegen
    The mapping of matrix elements to threads is not defined by this operation and may
    not be defined by some lowerings (such as the lowering to SPIR-V). However, if the
    size of the subgroup is S, then `subgroup_mma_insert_thread_local` at each index in
    `[0, (M * N) / S)` will have the entire matrix inserted across the subgroup.

    The op returns `!gpu.mma_matrix` with the updated value.

    Example:

    ```mlir
    %c0 = arith.constant 0 : index
    %s0 = gpu.subgroup_mma_insert_thread_local %val, %m[%c0] : f16, !gpu.mma_matrix<16x16xf16, "COp">
            -> !gpu.mma_matrix<16x16xf16, "COp">
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `lowerings`; this block coordinates dialect conversion or lowering decisions; touches target- or accelerator-specific semantics.
- **CN**: 声明与 `lowerings` 相关的 API 或声明式规则；该代码块协调方言转换或 lowering 决策，并涉及目标平台或加速器专用语义。

### Lines 2096-2105
```tablegen
  let arguments = (ins AnyIntegerOrFloat:$value, GPU_MMAMatrix:$matrix,
                       Variadic<Index>:$indices);

  let results = (outs GPU_MMAMatrix:$res);

  let assemblyFormat = [{
    $value`,` $matrix`[`$indices`]` attr-dict `:` type($value)`,` type($matrix) `->` type($res)
  }];
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 2106-2115
```tablegen
def GPU_ElementwiseOpAddF  : I32EnumAttrCase<"ADDF", 0, "addf">;
def GPU_ElementwiseOpMulF  : I32EnumAttrCase<"MULF", 1, "mulf">;
def GPU_ElementwiseOpSUBF  : I32EnumAttrCase<"SUBF", 2, "subf">;
def GPU_ElementwiseOpMaxF : I32EnumAttrCase<"MAXF", 3, "maxf">;
def GPU_ElementwiseOpMinF : I32EnumAttrCase<"MINF", 4, "minf">;
def GPU_ElementwiseOpDivF : I32EnumAttrCase<"DIVF", 5, "divf">;
def GPU_ElementwiseOpAddI  : I32EnumAttrCase<"ADDI", 6, "addi">;
def GPU_ElementwiseOpMulI  : I32EnumAttrCase<"MULI", 7, "muli">;
def GPU_ElementwiseOpSUBI  : I32EnumAttrCase<"SUBI", 8, "subi">;
def GPU_ElementwiseOpDivS : I32EnumAttrCase<"DIVS", 9, "divs">;
```
- **EN**: Introduces declarations for `GPU_ElementwiseOpAddF`, `GPU_ElementwiseOpMulF`, `GPU_ElementwiseOpSUBF`, `GPU_ElementwiseOpMaxF`, and 6 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ElementwiseOpAddF`, `GPU_ElementwiseOpMulF`, `GPU_ElementwiseOpSUBF`, `GPU_ElementwiseOpMaxF`, and 6 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2116-2135
```tablegen
def GPU_ElementwiseOpDivU : I32EnumAttrCase<"DIVU", 10, "divu">;
def GPU_ElementwiseOpNEGF : I32EnumAttrCase<"NEGATEF", 11, "negatef">;
def GPU_ElementwiseOpNEGS : I32EnumAttrCase<"NEGATES", 12, "negates">;
def GPU_ElementwiseOpEXTF : I32EnumAttrCase<"EXTF", 13, "extf">;
def GPU_ElementwiseOpTRUNCF : I32EnumAttrCase<"TRUNCF", 14, "truncf">;

def MMAElementWise : I32EnumAttr<"MMAElementwiseOp",
  "elementwise operation to apply to mma matrix", [
    GPU_ElementwiseOpAddF,
    GPU_ElementwiseOpMulF,
    GPU_ElementwiseOpSUBF,
    GPU_ElementwiseOpMaxF,
    GPU_ElementwiseOpMinF,
    GPU_ElementwiseOpDivF,
    GPU_ElementwiseOpAddI,
    GPU_ElementwiseOpMulI,
    GPU_ElementwiseOpSUBI,
    GPU_ElementwiseOpDivS,
    GPU_ElementwiseOpDivU,
    GPU_ElementwiseOpNEGF,
```
- **EN**: Introduces declarations for `GPU_ElementwiseOpDivU`, `GPU_ElementwiseOpNEGF`, `GPU_ElementwiseOpNEGS`, `GPU_ElementwiseOpEXTF`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_ElementwiseOpDivU`, `GPU_ElementwiseOpNEGF`, `GPU_ElementwiseOpNEGS`, `GPU_ElementwiseOpEXTF`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2136-2145
```tablegen
    GPU_ElementwiseOpNEGS,
    GPU_ElementwiseOpEXTF,
    GPU_ElementwiseOpTRUNCF
  ]> {
  let genSpecializedAttr = 0;
  let cppNamespace = "::mlir::gpu";
}
def MMAElementWiseAttr : EnumAttr<GPU_Dialect, MMAElementWise,
                                  "mma_element_wise">;

```
- **EN**: Introduces declarations for `MMAElementWiseAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MMAElementWiseAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2146-2156
```tablegen
def GPU_SubgroupMmaElementwiseOp : GPU_Op<"subgroup_mma_elementwise",
    [Pure,
     AllTypesMatch<["args"]>]>{

  let summary = "GPU warp elementwise operation on a matrix";

  let description = [{
    The `gpu.subgroup_mma_elementwise` takes `!gpu.mma_matrix` inputs and
    compute a new `!gpu.mma_matrix` by applying an elementwise operation to each
    element.

```
- **EN**: Introduces declarations for `GPU_SubgroupMmaElementwiseOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupMmaElementwiseOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2157-2170
```tablegen
    Since the operation is elementwise and the matrix type must match, the
    matrix elements are processed independently of the matrix layout.

    This op is meant to be used along with `gpu.subgroup_mma_compute`.

    Example:

    ```mlir
     %0 =  %A, %B { opType = "ADD" } :
      (!gpu.mma_matrix<16x16xf16, "COp">, !gpu.mma_matrix<16x16xf16, "COp">)
      -> !gpu.mma_matrix<16x16xf16, "COp">
    ```
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2171-2181
```tablegen
  let arguments = (ins Variadic<GPU_MMAMatrix>:$args,
                       MMAElementWiseAttr:$opType);

  let results = (outs GPU_MMAMatrix:$res);

  let extraClassDeclaration = [{
    gpu::MMAMatrixType getType() {
      return ::llvm::cast<gpu::MMAMatrixType>(getRes().getType());
    }
  }];

```
- **EN**: Implements logic around `getType`, `MMAMatrixType>`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getType`, `MMAMatrixType>` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 2182-2191
```tablegen
  let assemblyFormat = [{
    $opType $args attr-dict `:` functional-type($args, $res)
  }];
}

//
// Operation on sparse matrices, called from the host
// (currently lowers to cuSparse for CUDA only, no ROCM lowering).
//

```
- **EN**: Implements logic around `type`; this block coordinates dialect conversion or lowering decisions; handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 2192-2203
```tablegen
def GPU_CreateDnTensorOp : GPU_Op<"create_dn_tensor", [GPU_AsyncOpInterface, AttrSizedOperandSegments]> {
  let summary = "Create dense tensor operation";
  let description = [{
    The `gpu.create_dn_tensor` operation initializes a dense tensor from
    the given values buffer and sizes. The buffer must already be copied
    from the host to the device prior to using this operation. The
    operation returns a handle to the dense tensor descriptor.

    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

```
- **EN**: Introduces declarations for `GPU_CreateDnTensorOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_CreateDnTensorOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2204-2215
```tablegen
    Example:

    ```mlir
    %dmat, %token = gpu.create_dn_tensor async [%dep] %mem, %dims : index, index into memref<?xf64>
    ```
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       AnyMemRef:$memref,
                       Variadic<Index>:$dims);
  let results = (outs Res<GPU_SparseDnTensorHandle>:$dnTensor, Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2216-2228
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $memref `,` $dims attr-dict `:` type($dims) `into` type($memref)
  }];
}

def GPU_DestroyDnTensorOp : GPU_Op<"destroy_dn_tensor", [GPU_AsyncOpInterface]> {
  let summary = "Destroy dense tensor operation";
  let description = [{
    The `gpu.destroy_dn_tensor` operation releases all resources of a dense
    tensor represented by a handle that was previously created by a
    `gpu.create_dn_tensor` operation.

```
- **EN**: Introduces declarations for `GPU_DestroyDnTensorOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_DestroyDnTensorOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2229-2239
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    Example:

    ```mlir
    %token = gpu.destroy_dn_tensor async [%dep] %dnTensor
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 声明与 `asynchronously` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义。

### Lines 2240-2249
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       Arg<GPU_SparseDnTensorHandle>:$dnTensor);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $dnTensor attr-dict
  }];
}

```
- **EN**: Implements logic around `custom`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 2250-2262
```tablegen
def GPU_CreateCooOp : GPU_Op<"create_coo", [GPU_AsyncOpInterface]> {
  let summary = "Create sparse matrix in COO format operation";
  let description = [{
    The `gpu.create_coo` operation initializes a sparse matrix in COO format
    with the given sizes from the given index and values buffers. The buffers
    must already be copied from the host to the device prior to using this
    operation. The operation returns a handle to the sparse matrix descriptor.
    Note that this operation builds the COO in SoA format.

    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

```
- **EN**: Introduces declarations for `GPU_CreateCooOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_CreateCooOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2263-2280
```tablegen
    Example:

    ```mlir
    %spmat, %token = gpu.create_coo async [%dep] %rows, %cols, %nnz, %rowIdx,
        %colIdx, %values : memref<?xindex>, memref<?xindex>, memref<?xf64>
    ```
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       Index:$rows,
                       Index:$cols,
                       Index:$nnz,
                       AnyMemRef:$rowIdxs,
                       AnyMemRef:$colIdxs,
                       AnyMemRef:$values);
  let results = (outs Res<GPU_SparseSpMatHandle>:$spmat,
                      Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2281-2298
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $rows `,` $cols `,` $nnz `,` $rowIdxs `,` $colIdxs `,` $values attr-dict
    `:` type($rowIdxs) `,` type($colIdxs) `,` type($values)
  }];
}

def GPU_CreateCooAoSOp : GPU_Op<"create_coo_aos", [GPU_AsyncOpInterface]> {
  let summary = "Create sparse matrix in COO format operation (AoS)";
  let description = [{
    The `gpu.create_coo_aos` operation initializes a sparse matrix in COO format
    with the given sizes from the given index and values buffers. The buffers
    must already be copied from the host to the device prior to using this
    operation. The operation returns a handle to the sparse matrix descriptor.
    Unlike the default `gpu.create_coo` operation, this operation builds the
    COO format from a single index buffer in AoS format (note that this
    feature has been deprecated in cuSparse 11.2).

```
- **EN**: Introduces declarations for `GPU_CreateCooAoSOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_CreateCooAoSOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2299-2310
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    Example:

    ```mlir
    %spmat, %token = gpu.create_coo_aos async [%dep] %rows, %cols, %nnz, %idxs,
        %values : memref<?xindex>, memref<?xf64>
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `asynchronously`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `asynchronously` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2311-2326
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   Index:$rows,
                   Index:$cols,
                   Index:$nnz,
                   AnyMemRef:$idxs,
                   AnyMemRef:$values);
  let results = (outs Res<GPU_SparseSpMatHandle>:$spmat,
                      Optional<GPU_AsyncToken>:$asyncToken);

  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $rows `,` $cols `,` $nnz `,` $idxs `,` $values attr-dict
    `:` type($idxs) `,` type($values)
  }];
}

```
- **EN**: Implements logic around `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 2327-2338
```tablegen
def GPU_CreateCsrOp : GPU_Op<"create_csr", [GPU_AsyncOpInterface]> {
  let summary = "Create sparse matrix in CSR format operation";
  let description = [{
    The `gpu.create_csr` operation initializes a sparse matrix in CSR format
    with the given sizes from the given position, index, and values buffers.
    The buffers must already be copied from the host to the device prior to
    using this operation. The operation returns a handle to the sparse
    matrix descriptor.

    The CSR format has exactly the same memory layout as its transpose
    in CSC format (and vice versa).

```
- **EN**: Introduces declarations for `GPU_CreateCsrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_CreateCsrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2339-2350
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    Example:

    ```mlir
    %spmat, %token = gpu.create_csr async [%dep] %rows, %cols, %nnz, %rowPos,
        %colIdx, %values : memref<?xindex>, memref<?xindex>, memref<?xf64>
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `asynchronously`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `asynchronously` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2351-2360
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   Index:$rows,
                   Index:$cols,
                   Index:$nnz,
                   AnyMemRef:$rowPos,
                   AnyMemRef:$colIdxs,
                   AnyMemRef:$values);
  let results = (outs Res<GPU_SparseSpMatHandle>:$spmat,
                      Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2361-2376
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $rows `,` $cols `,` $nnz `,` $rowPos `,` $colIdxs `,` $values attr-dict
    `:` type($rowPos) `,` type($colIdxs) `,` type($values)
  }];
}

def GPU_CreateCscOp : GPU_Op<"create_csc", [GPU_AsyncOpInterface]> {
  let summary = "Create sparse matrix in CSC format operation";
  let description = [{
    The `gpu.create_csc` operation initializes a sparse matrix in CSC format
    with the given sizes from the given position, index, and values buffers.
    The buffers must already be copied from the host to the device prior to
    using this operation. The operation returns a handle to the sparse
    matrix descriptor.

```
- **EN**: Introduces declarations for `GPU_CreateCscOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_CreateCscOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2377-2391
```tablegen
    The CSC format has exactly the same memory layout as its transpose
    in CSR format (and vice versa).

    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    Example:

    ```mlir
    %spmat, %token = gpu.create_csc async [%dep] %rows, %cols, %nnz, %colPos,
        %rowIdx, %values : memref<?xindex>, memref<?xindex>, memref<?xf64>
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `format`, `asynchronously`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `format`, `asynchronously` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2392-2401
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   Index:$rows,
                   Index:$cols,
                   Index:$nnz,
                   AnyMemRef:$colPos,
                   AnyMemRef:$rowIdxs,
                   AnyMemRef:$values);
  let results = (outs Res<GPU_SparseSpMatHandle>:$spmat,
                      Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2402-2417
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $rows `,` $cols `,` $nnz `,` $colPos `,` $rowIdxs `,` $values attr-dict
    `:` type($colPos) `,` type($rowIdxs) `,` type($values)
  }];
}

def GPU_CreateBsrOp : GPU_Op<"create_bsr", [GPU_AsyncOpInterface]> {
  let summary = "Create sparse matrix in BSR format operation";
  let description = [{
    The `gpu.create_bsr` operation initializes a sparse matrix in BSR format
    with the given sizes for the matrix and blocks from the given position,
    index, and values buffers. The buffers must already be copied from the
    host to the device prior to using this operation. The operation returns
    a handle to the sparse matrix descriptor.

```
- **EN**: Introduces declarations for `GPU_CreateBsrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_CreateBsrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2418-2428
```tablegen
    The BSR format is similar to CSR, where the column indices represent
    two-dimensional blocks instead of a single matrix entry. Note that this
    operation (currently) only supports storage with **square** blocks,
    i.e., `rBlockSize == cBlockSize`.

    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    Example:

```
- **EN**: Implements logic around `operation`, `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `operation`, `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 2429-2447
```tablegen
    ```mlir
    %spmat, %token = gpu.create_bsr async [%dep]
       %brows, %bcols, %bnnz, %rBlockSize, %cBlockSize,
       %bRowPos, %bColIdxs, %values : memref<?xindex>, memref<?xindex>, memref<?xf64>
    ```
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   Index:$brows,
                   Index:$bcols,
                   Index:$bnnz,
                   Index:$rBlockSize,
                   Index:$cBlockSize,
                   AnyMemRef:$bRowPos,
                   AnyMemRef:$bColIdxs,
                   AnyMemRef:$values);
  let results = (outs Res<GPU_SparseSpMatHandle>:$spmat,
                      Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2448-2466
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $brows `,` $bcols `,` $bnnz `,` $rBlockSize `,` $cBlockSize `,`
    $bRowPos `,` $bColIdxs `,` $values attr-dict
    `:` type($bRowPos) `,` type($bColIdxs) `,` type($values)
  }];
}

def GPU_Prune2To4SpMatFlag : I32EnumAttr<"Prune2To4SpMatFlag",
  "pruning strategy for 2:4 sparse matrix",
  [
    I32EnumAttrCase<"NONE", 0>,
    I32EnumAttrCase<"PRUNE_ONLY", 1>,
    I32EnumAttrCase<"PRUNE_AND_CHECK", 2>,
  ]> {
    let genSpecializedAttr = 0;
    let cppNamespace = GPU_Dialect.cppNamespace;
}

```
- **EN**: Introduces declarations for `GPU_Prune2To4SpMatFlag`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_Prune2To4SpMatFlag` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2467-2481
```tablegen
def GPU_Prune2To4SpMatFlagAttr : EnumAttr<GPU_Dialect, GPU_Prune2To4SpMatFlag,
                                   "prune_2to4_spmat_flag">{
  let defaultValue = "Prune2To4SpMatFlag::PRUNE_AND_CHECK";
}


def GPU_Create2To4SpMatOp : GPU_Op<"create_2to4_spmat", [GPU_AsyncOpInterface]> {
  let summary = "Create sparse matrix with 2:4 sparsity operation";
  let description = [{
    The `gpu.create_2to4_spmat` operation initializes a sparse matrix in dense
    format with 2:4 sparsity.
    The buffers must already be copied from the host to the device prior to
    using this operation. The operation returns a handle to the sparse
    matrix descriptor.

```
- **EN**: Introduces declarations for `GPU_Prune2To4SpMatFlagAttr`, `GPU_Create2To4SpMatOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_Prune2To4SpMatFlagAttr`, `GPU_Create2To4SpMatOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2482-2492
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    Example:

    ```mlir
    %spmat, %token = gpu.create_2to4_spmat async [%dep] {PRUNE_AND_CHECK} %rows, %cols, %mem: memref<?xf64>
    ```
  }];

```
- **EN**: Implements logic around `asynchronously`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2493-2506
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       Index:$rows,
                       Index:$cols,
                       GPU_Prune2To4SpMatFlagAttr:$pruneFlag,
                       AnyMemRef:$memref);
  let results = (outs Res<GPU_SparseSpMatHandle>:$spMat,
                      Optional<GPU_AsyncToken>:$asyncToken);

  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
     `{` $pruneFlag `}` $rows `,` $cols `,` $memref attr-dict `:` type($memref)
  }];
}

```
- **EN**: Implements logic around `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2507-2517
```tablegen
def GPU_DestroySpMatOp : GPU_Op<"destroy_sp_mat", [GPU_AsyncOpInterface]> {
  let summary = "Destroy sparse matrix operation";
  let description = [{
    The `gpu.destroy_sp_mat` operation releases all resources of a sparse
    matrix represented by a handle that was previously created by a
    one of the sparse matrix creation operations.

    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

```
- **EN**: Introduces declarations for `GPU_DestroySpMatOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_DestroySpMatOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2518-2528
```tablegen
    Example:

    ```mlir
    %token = gpu.destroy_sp_mat async [%dep] %spmat
    ```
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       Arg<GPU_SparseSpMatHandle>:$spmat);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2529-2543
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies) $spmat attr-dict
  }];
}

// To avoid coupling this dialect with cusparse.h specifics, we hardcoded magic
// literals in this enum. Note that this should be kept in sync with
// cusparseOperation_t in cusparse.h:
// typedef enum {
// CUSPARSE_OPERATION_NON_TRANSPOSE       = 0,
// CUSPARSE_OPERATION_TRANSPOSE           = 1,
// CUSPARSE_OPERATION_CONJUGATE_TRANSPOSE = 2
// } cusparseOperation_t;
// TODO: find a proper way to keep them in sync?
def GPU_TransposeMode : I32EnumAttr<"TransposeMode",
```
- **EN**: Introduces declarations for `GPU_TransposeMode`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_TransposeMode` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2544-2553
```tablegen
    "transpose mode of sparse matrix supported by sparse tensor ops",
    [
      I32EnumAttrCase<"NON_TRANSPOSE", 0>,
      I32EnumAttrCase<"TRANSPOSE", 1>,
      I32EnumAttrCase<"CONJUGATE_TRANSPOSE", 2>,
    ]> {
      let genSpecializedAttr = 0;
      let cppNamespace = GPU_Dialect.cppNamespace;
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2554-2566
```tablegen
def GPU_TransposeModeAttr : EnumAttr<GPU_Dialect, GPU_TransposeMode,
                                   "mat_transpose_mode">{
  let defaultValue = "TransposeMode::NON_TRANSPOSE";
}

def GPU_SpMVBufferSizeOp : GPU_Op<"spmv_buffer_size", [GPU_AsyncOpInterface]> {
  let summary = "Precompute buffersize for SpMV operation";
  let description = [{
    The `gpu.spmv_buffer_size` operation returns the buffer size required
    to perform the SpMV operation on the given sparse matrix and dense vectors.
    The operation expects handles returned by previous sparse operations
    to construct an environment and the operands for SpMV.

```
- **EN**: Introduces declarations for `GPU_TransposeModeAttr`, `GPU_SpMVBufferSizeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_TransposeModeAttr`, `GPU_SpMVBufferSizeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2567-2576
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    The matrix arguments can also be associated with one of the following
    operators: NON_TRANSPOSE, TRANSPOSE, CONJUGATE_TRANSPOSE. The default value
    is NON_TRANSPOSE.

    Example:

```
- **EN**: Implements logic around `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 2577-2589
```tablegen
    ```mlir
    %buffersz, %token = gpu.spmv_buffer_size async [%dep] %spmatA{TRANSPOSE}, %dnX, %dnY into f32
    ```
  }];
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       GPU_TransposeModeAttr:$modeA,
                       GPU_SparseSpMatHandle:$spmatA,
                       GPU_SparseDnTensorHandle:$dnX,
                       GPU_SparseDnTensorHandle:$dnY,
                       TypeAttr:$computeType);
  let results = (outs Res<Index>:$bufferSz,
                      Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2590-2603
```tablegen
  let builders = [OpBuilder<(ins
      "Type":$bufferSz,
      "Type":$asyncToken,
      "ValueRange":$asyncDependencies,
      "Value":$spmatA,
      "Value":$dnX,
      "Value":$dnY,
      "Type":$computeType)
      , [{
    auto modeA = gpu::TransposeMode::NON_TRANSPOSE;
    return build($_builder, $_state, bufferSz, asyncToken, asyncDependencies,
                 modeA, spmatA, dnX, dnY, computeType);}]>
  ];

```
- **EN**: Implements logic around `OpBuilder`, `build`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2604-2617
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $spmatA (`{` $modeA^ `}`)? `,` $dnX `,` $dnY attr-dict  `into` $computeType
  }];
}

def GPU_SpMVOp : GPU_Op<"spmv", [GPU_AsyncOpInterface]> {
  let summary = "SpMV operation";
  let description = [{
    The `gpu.spmv` operation performs the SpMV operation on the given sparse matrix,
    dense vectors, and buffer.  The operation expects handles returned by previous
    sparse operations to construct an environment and the operands for SpMV. The
    buffer must have been allocated on the device.

```
- **EN**: Introduces declarations for `GPU_SpMVOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SpMVOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2618-2627
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    The matrix arguments can also be associated with one of the following
    operators: NON_TRANSPOSE, TRANSPOSE, CONJUGATE_TRANSPOSE. The default value
    is NON_TRANSPOSE.

    Example:

```
- **EN**: Implements logic around `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 2628-2640
```tablegen
    ```mlir
    %token = gpu.spmv async [%dep] %spmatA{TRANSPOSE}, %dnX, %dnY : memref<?xf64> into bf16
    ```
  }];
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       GPU_TransposeModeAttr:$modeA,
                       GPU_SparseSpMatHandle:$spmatA,
                       GPU_SparseDnTensorHandle:$dnX,
                       GPU_SparseDnTensorHandle:$dnY,
                       TypeAttr:$computeType,
                       AnyMemRef:$buffer);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2641-2653
```tablegen
  let builders = [OpBuilder<(ins
      "Type":$asyncToken,
      "ValueRange":$asyncDependencies,
      "Value":$spmatA,
      "Value":$dnX,
      "Value":$dnY,
      "Type":$computeType,
      "Value":$buffer), [{
    auto modeA = gpu::TransposeMode::NON_TRANSPOSE;
    return build($_builder, $_state, asyncToken, asyncDependencies, modeA,
                 spmatA, dnX, dnY, computeType, buffer);}]>
  ];

```
- **EN**: Implements logic around `OpBuilder`, `build`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2654-2667
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $spmatA (`{` $modeA^ `}`)? `,` $dnX `,` $dnY `,` $buffer attr-dict `:` type($buffer) `into` $computeType
  }];
}

def GPU_SpMMBufferSizeOp : GPU_Op<"spmm_buffer_size", [GPU_AsyncOpInterface, AttrSizedResultSegments]> {
  let summary = "Precompute buffersize for SpMM operation";
  let description = [{
    The `gpu.spmm_buffer_size` operation returns the buffer size required
    to perform the SpMM operation on the given sparse and dense matrix.
    The operation expects handles returned by previous sparse operations
    to construct an environment and the operands for SpMM.

```
- **EN**: Introduces declarations for `GPU_SpMMBufferSizeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SpMMBufferSizeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2668-2677
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    The matrix arguments can also be associated with one of the following
    operators: NON_TRANSPOSE, TRANSPOSE, CONJUGATE_TRANSPOSE. The default value
    is NON_TRANSPOSE.

    Example:

```
- **EN**: Implements logic around `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 2678-2692
```tablegen
    ```mlir
    %bufferszs, %token = gpu.spmm_buffer_size async [%dep] %spmatA{TRANSPOSE}, %dnmatB{TRANSPOSE}, %dnmatC : i64 into f32
    ```
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       GPU_TransposeModeAttr:$modeA,
                       GPU_TransposeModeAttr:$modeB,
                       GPU_SparseSpMatHandle:$spmatA,
                       GPU_SparseDnTensorHandle:$dnmatB,
                       GPU_SparseDnTensorHandle:$dnmatC,
                       TypeAttr:$computeType);
  let results = (outs Variadic<Index>:$bufferSzs,
                      Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2693-2706
```tablegen
  let builders = [OpBuilder<(ins
      "Type":$bufferSzs,
      "Type":$asyncToken,
      "ValueRange":$asyncDependencies,
      "Value":$spmatA,
      "Value":$dnmatB,
      "Value":$dnmatC,
      "Type":$computeType), [{
    auto modeA = gpu::TransposeMode::NON_TRANSPOSE;
    auto modeB = gpu::TransposeMode::NON_TRANSPOSE;
    return build($_builder, $_state, bufferSzs, asyncToken, asyncDependencies,
                 modeA, modeB, spmatA, dnmatB, dnmatC, computeType);}]>
  ];

```
- **EN**: Implements logic around `OpBuilder`, `build`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2707-2720
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $spmatA (`{` $modeA^ `}`)? `,` $dnmatB (`{` $modeB^ `}`)? `,` $dnmatC attr-dict `:` type($bufferSzs) `into` $computeType
  }];
}

def GPU_SpMMOp : GPU_Op<"spmm", [GPU_AsyncOpInterface, AttrSizedOperandSegments]> {
  let summary = "SpMM operation";
  let description = [{
    The `gpu.spmm` operation performs the SpMM operation on the given sparse and
    dense matrix, and buffer.  The operation expects handles returned by previous
    sparse operations to construct an environment and the operands for SpMM. The
    buffer must have been allocated on the device.

```
- **EN**: Introduces declarations for `GPU_SpMMOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SpMMOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2721-2730
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    The matrix arguments can also be associated with one of the following
    operators: NON_TRANSPOSE, TRANSPOSE, CONJUGATE_TRANSPOSE. The default value
    is NON_TRANSPOSE.

    Example:

```
- **EN**: Implements logic around `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 2731-2745
```tablegen
    ```mlir
    %token = gpu.spmm async [%dep] %spmatA{TRANSPOSE}, %dnmatB{TRANSPOSE}, %dnmatC, %buffers : type($buffers) into f32
    ```
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       GPU_TransposeModeAttr:$modeA,
                       GPU_TransposeModeAttr:$modeB,
                       GPU_SparseSpMatHandle:$spmatA,
                       GPU_SparseDnTensorHandle:$dnmatB,
                       GPU_SparseDnTensorHandle:$dnmatC,
                       TypeAttr:$computeType,
                       Variadic<AnyMemRef>:$buffers);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Implements logic around `type`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2746-2759
```tablegen
  let builders = [OpBuilder<(ins
      "Type":$asyncToken,
      "ValueRange":$asyncDependencies,
      "Value":$spmatA,
      "Value":$dnmatB,
      "Value":$dnmatC,
      "Type":$computeType,
      "ValueRange":$buffers), [{
    auto modeA = gpu::TransposeMode::NON_TRANSPOSE;
    auto modeB = gpu::TransposeMode::NON_TRANSPOSE;
    return build($_builder, $_state, asyncToken, asyncDependencies, modeA,
                 modeB, spmatA, dnmatB, dnmatC, computeType, buffers);}]>
  ];

```
- **EN**: Implements logic around `OpBuilder`, `build`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2760-2773
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $spmatA (`{` $modeA^ `}`)? `,` $dnmatB (`{` $modeB^ `}`)? `,` $dnmatC `,` $buffers attr-dict `:` type($buffers) `into` $computeType
  }];
}

def GPU_SDDMMBufferSizeOp : GPU_Op<"sddmm_buffer_size", [GPU_AsyncOpInterface]> {
  let summary = "Precompute buffersize for SDDMM operation";
  let description = [{
    The `gpu.sddmm_buffer_size` operation returns the buffer size required
    to perform the SDDMM operation on the given sparse and dense matrices.
    The operation expects handles returned by previous sparse operations
    to construct an environment and the operands for SDDMM.

```
- **EN**: Introduces declarations for `GPU_SDDMMBufferSizeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SDDMMBufferSizeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2774-2783
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    Example:

    ```mlir
    %buffersz, %token = gpu.sddmm_buffer_size async [%dep] %dnmatA{TRANSPOSE}, %dnmatB{TRANSPOSE}, %spmatC into f32
    ```

```
- **EN**: Implements logic around `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 2784-2797
```tablegen
    The matrix arguments can also be associated with one of the following
    operators: NON_TRANSPOSE, TRANSPOSE, CONJUGATE_TRANSPOSE. The default value
    is NON_TRANSPOSE.
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   GPU_TransposeModeAttr:$modeA,
                   GPU_TransposeModeAttr:$modeB,
                   GPU_SparseDnTensorHandle:$dnmatA,
                   GPU_SparseDnTensorHandle:$dnmatB,
                   GPU_SparseSpMatHandle:$spmatC,
                   TypeAttr:$computeType);
  let results = (outs Res<Index>:$bufferSz, Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2798-2811
```tablegen
  let builders = [OpBuilder<(ins
      "Type":$bufferSz,
      "Type":$asyncToken,
      "ValueRange":$asyncDependencies,
      "Value":$dnmatA,
      "Value":$dnmatB,
      "Value":$spmatC,
      "Type":$computeType), [{
    auto modeA = gpu::TransposeMode::NON_TRANSPOSE;
    auto modeB = gpu::TransposeMode::NON_TRANSPOSE;
    return build($_builder, $_state, bufferSz, asyncToken, asyncDependencies,
                 modeA, modeB, dnmatA, dnmatB, spmatC, computeType);}]>
  ];

```
- **EN**: Implements logic around `OpBuilder`, `build`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2812-2825
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $dnmatA (`{` $modeA^ `}`)? `,` $dnmatB (`{` $modeB^ `}`)? `,` $spmatC attr-dict `into` $computeType
  }];
}

def GPU_SDDMMOp : GPU_Op<"sddmm", [GPU_AsyncOpInterface]> {
  let summary = "SDDMM operation";
  let description = [{
    The `gpu.sddmm` operation performs the SDDMM operation on the given sparse and
    dense matrices, and buffer.  The operation expects handles returned by previous
    sparse operations to construct an environment and the operands for SDDMM. The
    buffer must have been allocated on the device.

```
- **EN**: Introduces declarations for `GPU_SDDMMOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SDDMMOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2826-2835
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a !gpu.async.token in addition to the environment.

    Example:

    ```mlir
    %token = gpu.sddmm async [%dep] %dnmatA{TRANSPOSE}, %dnmatB{TRANSPOSE}, %spmatC, %buffer into f32
    ```

```
- **EN**: Implements logic around `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 2836-2850
```tablegen
    The matrix arguments can also be associated with one of the following
    operators: NON_TRANSPOSE, TRANSPOSE, CONJUGATE_TRANSPOSE. The default value
    is NON_TRANSPOSE.
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                   GPU_TransposeModeAttr:$modeA,
                   GPU_TransposeModeAttr:$modeB,
                   GPU_SparseDnTensorHandle:$dnmatA,
                   GPU_SparseDnTensorHandle:$dnmatB,
                   GPU_SparseSpMatHandle:$spmatC,
                   TypeAttr:$computeType,
                   AnyMemRef:$buffer);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2851-2864
```tablegen
  let builders = [OpBuilder<(ins
    "Type":$asyncToken,
    "ValueRange":$asyncDependencies,
    "Value":$dnmatA,
    "Value":$dnmatB,
    "Value":$spmatC,
    "Type":$computeType,
    "Value":$buffer), [{
  auto modeA = gpu::TransposeMode::NON_TRANSPOSE;
  auto modeB = gpu::TransposeMode::NON_TRANSPOSE;
  return build($_builder, $_state, asyncToken, asyncDependencies, modeA,
                modeB, dnmatA, dnmatB, spmatC, computeType, buffer);}]>
  ];

```
- **EN**: Implements logic around `OpBuilder`, `build`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2865-2880
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $dnmatA (`{` $modeA^ `}`)? `,` $dnmatB (`{` $modeB^ `}`)? `,` $spmatC `,` $buffer attr-dict `:` type($buffer) `into` $computeType
  }];
}

def GPU_SpGEMMWorkEstimationOrComputeKind : I32EnumAttr<"SpGEMMWorkEstimationOrComputeKind",
    "choose whether spgemm_work_estimation_or_compute does work estimation or compute",
    [
      I32EnumAttrCase<"WORK_ESTIMATION", 0>,
      I32EnumAttrCase<"COMPUTE", 1>,
    ]> {
      let genSpecializedAttr = 0;
      let cppNamespace = GPU_Dialect.cppNamespace;
}

```
- **EN**: Introduces declarations for `GPU_SpGEMMWorkEstimationOrComputeKind`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SpGEMMWorkEstimationOrComputeKind` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2881-2892
```tablegen
def GPU_SpGEMMWorkEstimationOrComputeKindAttr : EnumAttr<GPU_Dialect,
    GPU_SpGEMMWorkEstimationOrComputeKind,
    "spgemm_work_estimation_or_compute_kind"> {}

def GPU_SpGEMMCreateDescrOp : GPU_Op<"spgemm_create_descr", [GPU_AsyncOpInterface]> {
  let summary = "SpGEMM Create Descr operation";
  let description = [{
    The `gpu.spgemm_create_descr` creates a descriptor for the SpGEMM operation.
    The descriptor describes the SpGEMM operation and stores the internal data
    throughout the computation. It needs to be passed as an argument to
    spgemm_* operations.

```
- **EN**: Introduces declarations for `GPU_SpGEMMWorkEstimationOrComputeKindAttr`, `GPU_SpGEMMCreateDescrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SpGEMMWorkEstimationOrComputeKindAttr`, `GPU_SpGEMMCreateDescrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2893-2911
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a `!gpu.async.token` in addition to the environment.

    Example:

    ```mlir
    %desc, %token = gpu.spgemm_create_descr async [%dep]
    ```
  }];
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies);
  let results = (outs GPU_SparseSpGEMMOpHandle:$desc,
                      Optional<GPU_AsyncToken>:$asyncToken);
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    attr-dict
  }];
}

```
- **EN**: Implements logic around `asynchronously`, `custom`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `asynchronously`, `custom` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2912-2922
```tablegen
def GPU_SpGEMMDestroyDescrOp : GPU_Op<"spgemm_destroy_descr", [GPU_AsyncOpInterface]> {
  let summary = "SpGEMM Destroy Descr operation";
  let description = [{
    The `gpu.spgemm_destroy_descr` destroys the SpGEMM operation descriptor.

    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a `!gpu.async.token` in addition to the environment.

    Example:

```
- **EN**: Introduces declarations for `GPU_SpGEMMDestroyDescrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SpGEMMDestroyDescrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2923-2936
```tablegen
    ```mlir
    %token = gpu.spgemm_destroy_descr async [%dep] %desc
    ```
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       GPU_SparseSpGEMMOpHandle:$desc);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $desc attr-dict
  }];
}

```
- **EN**: Implements logic around `custom`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `custom` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2937-2946
```tablegen
def GPU_SpGEMMWorkEstimationOrComputeOp : GPU_Op<"spgemm_work_estimation_or_compute", [GPU_AsyncOpInterface]> {
  let summary = "SpGEMM work estimation operation";
  let description = [{
    The `gpu.spgemm_work_estimation_or_compute` is used to call
    cusparseSpGEMM_workEstimation or cusparseSpGEMM_compute. Both of them are
    for both determining the buffer size and performing the actual computation.
    The operation expects handles returned by previous sparse operations to
    construct an environment and the operands for SpGEMM.
    The buffer must have been allocated on the device.

```
- **EN**: Introduces declarations for `GPU_SpGEMMWorkEstimationOrComputeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SpGEMMWorkEstimationOrComputeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2947-2961
```tablegen
    C' = alpha * op(A) * op(B) + beta * C

    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a `!gpu.async.token` in addition to the environment.

    Example:

    ```mlir
    %bufferSz, %token = gpu.spgemm_work_estimation_or_compute async [%dep] {COMPUTE}
                          %desc, %spmatA{NON_TRANSPOSE}, %spmatB{NON_TRANSPOSE},
                          %spmatC, %spgemmDesc, %c0, %alloc: f32 into
                          memref<0xi8>
    ```

```
- **EN**: Implements logic around `op`, `asynchronously`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `op`, `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2962-2980
```tablegen
    The matrix arguments can also be associated with one of the following
    operators: NON_TRANSPOSE, TRANSPOSE, CONJUGATE_TRANSPOSE. The default value
    is NON_TRANSPOSE.
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       GPU_SparseSpGEMMOpHandle:$desc,
                       GPU_TransposeModeAttr:$modeA,
                       GPU_TransposeModeAttr:$modeB,
                       GPU_SparseSpMatHandle:$spmatA,
                       GPU_SparseSpMatHandle:$spmatB,
                       GPU_SparseSpMatHandle:$spmatC,
                       TypeAttr:$computeType,
                       Index:$bufferSz,
                       AnyMemRef:$buffer,
                       GPU_SpGEMMWorkEstimationOrComputeKindAttr:$kind);
  let results = (outs Res<Index>:$bufferSzNew,
                      Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2981-2998
```tablegen
  let builders = [OpBuilder<(ins
    "Type":$bufferSzNew,
    "Type":$asyncToken,
    "ValueRange":$asyncDependencies,
    "Value":$desc,
    "Value":$spmatA,
    "Value":$spmatB,
    "Value":$spmatC,
    "Type":$computeType,
    "Value":$bufferSz,
    "Value":$buffer), [{
  auto modeA = gpu::TransposeMode::NON_TRANSPOSE;
  auto modeB = gpu::TransposeMode::NON_TRANSPOSE;
  auto kind = gpu::SpGEMMWorkEstimationOrComputeKind::WORK_ESTIMATION;
  return build($_builder, $_state, bufferSzNew, asyncToken, asyncDependencies, desc,
               modeA, modeB, spmatA, spmatB, spmatC, computeType, bufferSz, buffer, kind);}]>
  ];

```
- **EN**: Implements logic around `OpBuilder`, `build`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2999-3010
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    `{` $kind `}` $spmatA (`{` $modeA^ `}`)? `,` $spmatB (`{` $modeB^ `}`)? `,` $spmatC `,` $desc `,` $bufferSz `,` $buffer  attr-dict `:` $computeType `into` type($buffer)
  }];
}

def GPU_SpGEMMCopyOp : GPU_Op<"spgemm_copy", [GPU_AsyncOpInterface]> {
  let summary = "SpGEMM copy operation";
  let description = [{
    The `gpu.spgemm_copy` operation copies the sparse matrix result of
    a SpGEMM computation.

```
- **EN**: Introduces declarations for `GPU_SpGEMMCopyOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SpGEMMCopyOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 3011-3020
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a `!gpu.async.token` in addition to the environment.

    Example:

    ```mlir
    gpu.spgemm_copy %spmatA, %spmatB, %spmatC, %spgemmDesc: f32
    ```

```
- **EN**: Implements logic around `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `asynchronously` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 3021-3035
```tablegen
    The matrix arguments can also be associated with one of the following
    operators: NON_TRANSPOSE, TRANSPOSE, CONJUGATE_TRANSPOSE. The default value
    is NON_TRANSPOSE.
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       GPU_SparseSpGEMMOpHandle:$desc,
                       GPU_TransposeModeAttr:$modeA,
                       GPU_TransposeModeAttr:$modeB,
                       GPU_SparseSpMatHandle:$spmatA,
                       GPU_SparseSpMatHandle:$spmatB,
                       GPU_SparseSpMatHandle:$spmatC,
                       TypeAttr:$computeType);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 3036-3049
```tablegen
  let builders = [OpBuilder<(ins
    "Type":$asyncToken,
    "ValueRange":$asyncDependencies,
    "Value":$desc,
    "Value":$spmatA,
    "Value":$spmatB,
    "Value":$spmatC,
    "Type":$computeType), [{
  auto modeA = gpu::TransposeMode::NON_TRANSPOSE;
  auto modeB = gpu::TransposeMode::NON_TRANSPOSE;
  return build($_builder, $_state, asyncToken, asyncDependencies, desc,
               modeA, modeB, spmatA, spmatB, spmatC, computeType);}]>
  ];

```
- **EN**: Implements logic around `OpBuilder`, `build`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 3050-3061
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $spmatA (`{` $modeA^ `}`)? `,` $spmatB (`{` $modeB^ `}`)? `,` $spmatC `,` $desc attr-dict `:` $computeType
  }];
}

def GPU_SpMatGetSizeOp : GPU_Op<"spmat_get_size", [GPU_AsyncOpInterface]> {
  let summary = "SpMat get size operation";
  let description = [{
    The `gpu.spmat_get_size` operation retrieves the number of rows, number of
    columns, and number of non-zero elements of a sparse matrix.

```
- **EN**: Introduces declarations for `GPU_SpMatGetSizeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SpMatGetSizeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 3062-3072
```tablegen
    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a `!gpu.async.token` in addition to the environment.

    Example:

    ```mlir
    %rows, %cols, %nnz, %token = gpu.spmat_get_size async [%dep] %spmatC
    ```
  }];

```
- **EN**: Declares APIs or declarative rules around `asynchronously`; this block touches target- or accelerator-specific semantics.
- **CN**: 声明与 `asynchronously` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义。

### Lines 3073-3085
```tablegen
  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       GPU_SparseSpMatHandle:$spmat);
  let results = (outs Index:$rows,
                      Index:$cols,
                      Index:$nnz,
                      Optional<GPU_AsyncToken>:$asyncToken);

  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
    $spmat attr-dict
  }];
}

```
- **EN**: Implements logic around `custom`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 3086-3096
```tablegen
def GPU_SetCsrPointersOp : GPU_Op<"set_csr_pointers", [GPU_AsyncOpInterface]> {
  let summary = "SpGEMM get size operation";
  let description = [{
    The `gpu.set_csr_pointers` assigns the given positions, coordinates,
    and values buffer that reside on the device directly to the given sparse
    matrix descriptor in csr format.

    If the `async` keyword is present, the op is executed asynchronously (i.e.
    it does not block until the execution has finished on the device). In
    that case, it returns a `!gpu.async.token` in addition to the environment.

```
- **EN**: Introduces declarations for `GPU_SetCsrPointersOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SetCsrPointersOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 3097-3111
```tablegen
    Example:

    ```mlir
    %token = gpu.set_csr_pointers async [%dep] %positions, %coordinates, %values
          : memref<?xf32>, memref<?xindex>, memref<?xindex>
    ```
  }];

  let arguments = (ins Variadic<GPU_AsyncToken>:$asyncDependencies,
                       Arg<GPU_SparseSpMatHandle>:$spmat,
                       AnyMemRef:$positions,
                       AnyMemRef:$coordinates,
		       AnyMemRef:$values);
  let results = (outs Optional<GPU_AsyncToken>:$asyncToken);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 3112-3131
```tablegen
  let assemblyFormat = [{
    custom<AsyncDependencies>(type($asyncToken), $asyncDependencies)
      $spmat `,` $positions `,` $coordinates `,` $values attr-dict
        `:` type($positions) `,` type($coordinates) `,` type($values)
  }];
}

def GPU_WarpExecuteOnLane0Op : GPU_Op<"warp_execute_on_lane_0",
      [DeclareOpInterfaceMethods<RegionBranchOpInterface, [
          "areTypesCompatible", "getSuccessorInputs"]>,
       SingleBlockImplicitTerminator<"gpu::YieldOp">,
       RecursiveMemoryEffects]> {
  let summary = "Executes operations in the associated region on thread #0 of a"
                "SPMD program";
  let description = [{
    `warp_execute_on_lane_0` is an operation used to bridge the gap between
    vector programming and SPMD programming model like GPU SIMT. It allows to
    trivially convert a region of vector code meant to run on a multiple threads
    into a valid SPMD region and then allows incremental transformation to
    distribute vector operations on the threads.
```
- **EN**: Introduces declarations for `GPU_WarpExecuteOnLane0Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_WarpExecuteOnLane0Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 3132-3144
```tablegen

    Any code present in the region would only be executed on first thread/lane
    based on the `laneid` operand. The `laneid` operand is an integer ID between
    [0, `warp_size`). The `warp_size` attribute indicates the number of lanes in
    a warp.

    Operands are vector values distributed on all lanes that may be used by
    the single lane execution. The matching region argument is a vector of all
    the values of those lanes available to the single active lane. The
    distributed dimension is implicit based on the shape of the operand and
    argument. the properties of the distribution may be described by extra
    attributes (e.g. affine map).

```
- **EN**: Implements logic around `attributes`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `attributes` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 3145-3157
```tablegen
    Return values are distributed on all lanes using laneId as index. The
    vector is distributed based on the shape ratio between the vector type of
    the yield and the result type.
    If the shapes are the same this means the value is broadcasted to all lanes.
    In the future the distribution can be made more explicit using affine_maps
    and will support having multiple Ids.

    Therefore the `warp_execute_on_lane_0` operations allow to implicitly copy
    between lane0 and the lanes of the warp. When distributing a vector
    from lane0 to all the lanes, the data are distributed in a block cyclic way.
    For example `vector<64xf32>` gets distributed on 32 threads and map to
    `vector<2xf32>` where thread 0 contains vector[0] and vector[1].

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 3158-3175
```tablegen
    During lowering values passed as operands and return value need to be
    visible to different lanes within the warp. This would usually be done by
    going through memory.

    The region is *not* isolated from above. For values coming from the parent
    region not going through operands only the lane 0 value will be accesible so
    it generally only make sense for uniform values.

    Example:
    ```
    // Execute in parallel on all threads/lanes.
    gpu.warp_execute_on_lane_0 (%laneid)[32] {
      // Serial code running only on thread/lane 0.
      ...
    }
    // Execute in parallel on all threads/lanes.
    ```

```
- **EN**: Implements logic around `warp_execute_on_lane_0`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `warp_execute_on_lane_0` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 3176-3186
```tablegen
    This may be lowered to an scf.if region as below:
    ```
      // Execute in parallel on all threads/lanes.
      %cnd = arith.cmpi eq, %laneid, %c0 : index
      scf.if %cnd {
        // Serial code running only on thread/lane 0.
        ...
      }
      // Execute in parallel on all threads/lanes.
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 3187-3199
```tablegen
    When the region has operands and/or return values:
    ```
    // Execute in parallel on all threads/lanes.
    %0 = gpu.warp_execute_on_lane_0(%laneid)[32]
    args(%v0 : vector<4xi32>) -> (vector<1xf32>) {
    ^bb0(%arg0 : vector<128xi32>) :
      // Serial code running only on thread/lane 0.
      ...
      gpu.yield %1 : vector<32xf32>
    }
    // Execute in parallel on all threads/lanes.
    ```

```
- **EN**: Implements logic around `warp_execute_on_lane_0`, `args`, `bb0`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `warp_execute_on_lane_0`, `args`, `bb0` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 3200-3219
```tablegen
    values at the region boundary would go through memory:
    ```
    // Execute in parallel on all threads/lanes.
    ...
    // Store the data from each thread into memory and Synchronization.
    %tmp0 = memreg.alloc() : memref<128xf32>
    %tmp1 = memreg.alloc() : memref<32xf32>
    %cnd = arith.cmpi eq, %laneid, %c0 : index
    vector.store %v0, %tmp0[%laneid] : memref<128xf32>, vector<4xf32>
    some_synchronization_primitive
    scf.if %cnd {
      // Serialized code running only on thread 0.
      // Load the data from all the threads into a register from thread 0. This
      // allow threads 0 to access data from all the threads.
      %arg0 = vector.load %tmp0[%c0] : memref<128xf32>, vector<128xf32>
      ...
      // Store the data from thread 0 into memory.
      vector.store %1, %tmp1[%c0] : memref<32xf32>, vector<32xf32>
    }
    // Synchronization and load the data in a block cyclic way so that the
```
- **EN**: Implements logic around `alloc`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `alloc` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 3220-3234
```tablegen
    // vector is distributed on all threads.
    some_synchronization_primitive
    %0 = vector.load %tmp1[%laneid] : memref<32xf32>, vector<32xf32>
    // Execute in parallel on all threads/lanes.
    ```

  }];

  let hasVerifier = 1;
  let hasCustomAssemblyFormat = 1;
  let arguments = (ins Index:$laneid, I64Attr:$warp_size,
                       Variadic<AnyType>:$args);
  let results = (outs Variadic<AnyType>:$results);
  let regions = (region SizedRegion<1>:$warpRegion);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 3235-3246
```tablegen
  let skipDefaultBuilders = 1;
  let builders = [
    OpBuilder<(ins "TypeRange":$resultTypes, "Value":$laneid,
                   "int64_t":$warpSize)>,
    // `blockArgTypes` are different than `args` types as they are they
    // represent all the `args` instances visibile to lane 0. Therefore we need
    // to explicit pass the type.
    OpBuilder<(ins "TypeRange":$resultTypes, "Value":$laneid,
                   "int64_t":$warpSize, "ValueRange":$args,
                   "TypeRange":$blockArgTypes)>
  ];

```
- **EN**: Declares APIs or declarative rules around `OpBuilder`; this block packages logic as an MLIR pass or pass-related API; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `OpBuilder` 相关的 API 或声明式规则；该代码块将逻辑组织为 MLIR Pass 或相关 API，并处理方言 IR 实体，如操作、类型或属性。

### Lines 3247-3256
```tablegen
  let extraClassDeclaration = [{
    bool isDefinedOutsideOfRegion(Value value) {
      return !getRegion().isAncestor(value.getParentRegion());
    }

    /// Get the terminator of the warp region.
    gpu::YieldOp getTerminator();
  }];
}

```
- **EN**: Implements logic around `isDefinedOutsideOfRegion`, `getRegion`, `getTerminator`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `isDefinedOutsideOfRegion`, `getRegion`, `getTerminator` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 3257-3266
```tablegen
def GPU_BroadcastType : I32EnumAttr<"BroadcastType",
    "a lane to broadcast from",
    [
      I32EnumAttrCase<"first_active_lane", 0>,
      I32EnumAttrCase<"specific_lane", 1>
    ]>{
  let genSpecializedAttr = 0;
  let cppNamespace = "::mlir::gpu";
}
def GPU_BroadcastTypeAttr : EnumAttr<GPU_Dialect, GPU_BroadcastType, "broadcast">;
```
- **EN**: Introduces declarations for `GPU_BroadcastType`, `GPU_BroadcastTypeAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_BroadcastType`, `GPU_BroadcastTypeAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 3267-3280
```tablegen

def GPU_SubgroupBroadcastOp : GPU_Op<"subgroup_broadcast",
    [NoMemoryEffect, AllTypesMatch<["result", "src"]>,
    DeclareOpInterfaceMethods<InferIntRangeInterface, ["inferResultRanges"]>,
    DeclareOpInterfaceMethods<ConditionallySpeculatable, ["getSpeculatability"]>] #
    ElementwiseMappable.traits>,
  Arguments<(ins AnyType:$src,
                 Optional<I32>:$lane,
                 GPU_BroadcastTypeAttr:$broadcast_type)> {
  let summary = "Broadcasts a value from the specific lane across subgroup";
  let description = [{
      Broadcasts a value from one lane to all active lanes in a subgroup. The
      result is guaranteed to be uniform across the active lanes in subgroup.

```
- **EN**: Introduces declarations for `GPU_SubgroupBroadcastOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SubgroupBroadcastOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 3281-3297
```tablegen
      The possible broadcast types are:

      * `first_active_lane` - broadcasts the value from the first active lane
      in the subgroup.
      * `specific_lane` - broadcasts from the specified lane. The lane index
      must be uniform and within the subgroup size. The result is poison if the
      lane index is invalid, non subgroup-uniform, or if the source lane is not
      active.
  }];
  let results = (outs AnyType:$result);
  let assemblyFormat = [{
    $src `,` $broadcast_type ($lane^)? attr-dict `:` type($result)
  }];
  let hasFolder = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `broadcast_type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `broadcast_type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 3298-3309
```tablegen
def GPU_BallotOp : GPU_Op<"ballot", [NoMemoryEffect]>,
    Arguments<(ins I1:$predicate)>,
    Results<(outs AnySignlessInteger:$result)> {
  let summary = "Collects predicate values from all threads in a subgroup.";
  let description = [{
    The `ballot` op performs a ballot operation across all threads in a subgroup.
    Each thread contributes its predicate value as a single bit. The result is an
    integer where the Nth bit is set iff the Nth thread's predicate is true.

    The result type must be a signless integer type. The exact supported widths
    are target-dependent. Common GPU targets support i32 and i64.

```
- **EN**: Introduces declarations for `GPU_BallotOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_BallotOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 3310-3319
```tablegen
    Example:
    ```mlir
    %0 = gpu.ballot %pred : i32
    %1 = gpu.ballot %pred : i64
    ```
  }];
  let assemblyFormat = "$predicate attr-dict `:` type($result)";
}

#endif // GPU_OPS
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Describes the declarative or C++ interface for dialect operations, attributes, types, and registration helpers.
  - **CN**: 描述方言操作、属性、类型及注册辅助逻辑的声明式或 C++ 接口。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, interfaces, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束、接口或规范化规则。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Dialect/DLTI/DLTIBase.td`, `mlir/Dialect/GPU/IR/GPUBase.td`, `mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td`, `mlir/Dialect/GPU/IR/CompilationAttrs.td`, `mlir/Dialect/GPU/IR/GPUDeviceMappingAttr.td`, `mlir/Dialect/GPU/IR/ParallelLoopMapperAttr.td`, `mlir/IR/CommonTypeConstraints.td`, `mlir/IR/EnumAttr.td`, `mlir/IR/RegionKindInterface.td`, `mlir/IR/SymbolInterfaces.td` ... (+7 more)
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (6), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (6), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (5)
