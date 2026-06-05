# GPUTransformOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/GPU/TransformOps/GPUTransformOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the GPU dialect, focused on transform dialect operation declarations and orchestration hooks and `GPUTransformOps`.
  - **CN**: 为 GPU 方言定义聚焦 `GPUTransformOps` 的声明式 TableGen 规格，覆盖Transform Dialect 操作声明与编排钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```tablegen
//===- GPUTransformOps.td - GPU transform ops --------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 9-16
```tablegen
#ifndef GPU_TRANSFORM_OPS
#define GPU_TRANSFORM_OPS

include "mlir/Dialect/Transform/IR/TransformDialect.td"
include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/IR/OpBase.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 17-32
```tablegen
//===----------------------------------------------------------------------===//
// Apply...ConversionPatternsOp
//===----------------------------------------------------------------------===//

def ApplyGPUToNVVMConversionPatternsOp : Op<Transform_Dialect,
    "apply_conversion_patterns.gpu.gpu_to_nvvm",
    [DeclareOpInterfaceMethods<ConversionPatternDescriptorOpInterface,
                               ["verifyTypeConverter"]>]> {
  let description = [{
    Collects patterns that convert GPU dialect ops to NVVM dialect ops. These
    patterns require an "LLVMTypeConverter".
  }];
  let arguments = (ins DefaultValuedAttr<I16Attr, "1">:$benefit);
  let assemblyFormat = "attr-dict";
}

```
- **EN**: Introduces declarations for `ApplyGPUToNVVMConversionPatternsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ApplyGPUToNVVMConversionPatternsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 33-44
```tablegen
def ApplyGPUWwmaToNVVMConversionPatternsOp : Op<Transform_Dialect,
    "apply_conversion_patterns.gpu.gpu_wmma_to_nvvm",
    [DeclareOpInterfaceMethods<ConversionPatternDescriptorOpInterface,
                               ["verifyTypeConverter"]>]> {
  let description = [{
    Collects patterns that convert GPU dialect ops related to wmma ops
    to NVVM dialect ops.
    These patterns require an "LLVMTypeConverter".
  }];
  let assemblyFormat = "attr-dict";
}

```
- **EN**: Introduces declarations for `ApplyGPUWwmaToNVVMConversionPatternsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ApplyGPUWwmaToNVVMConversionPatternsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 45-56
```tablegen
def ApplyGPUSubgroupReduceToNVVMConversionPatternsOp : Op<Transform_Dialect,
    "apply_conversion_patterns.gpu.gpu_subgroup_reduce_to_nvvm",
    [DeclareOpInterfaceMethods<ConversionPatternDescriptorOpInterface,
                               ["verifyTypeConverter"]>]> {
  let description = [{
    Collects patterns that convert GPU dialect ops related to wmma ops
    to NVVM dialect ops.
    These patterns require an "LLVMTypeConverter".
  }];
  let assemblyFormat = "attr-dict";
}

```
- **EN**: Introduces declarations for `ApplyGPUSubgroupReduceToNVVMConversionPatternsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ApplyGPUSubgroupReduceToNVVMConversionPatternsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 57-70
```tablegen
def ApplyGPUToROCDLConversionPatternsOp : Op<Transform_Dialect,
    "apply_conversion_patterns.gpu.gpu_to_rocdl",
    [DeclareOpInterfaceMethods<ConversionPatternDescriptorOpInterface,
                               ["verifyTypeConverter"]>]> {
  let description = [{
    Collects patterns that convert GPU dialect ops to ROCDL dialect ops. These
    patterns require an "LLVMTypeConverter".
  }];
  let arguments = (ins StrAttr:$chipset);
  let assemblyFormat = [{
    `chipset` `=` $chipset attr-dict
  }];
}

```
- **EN**: Introduces declarations for `ApplyGPUToROCDLConversionPatternsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ApplyGPUToROCDLConversionPatternsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 71-86
```tablegen
//===----------------------------------------------------------------------===//
// Apply...PatternsOp
//===----------------------------------------------------------------------===//

def ApplyGPURewritePatternsOp : Op<Transform_Dialect,
    "apply_patterns.gpu.gpu_rewrite_patterns",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Collects GPU rewrite patterns comprising:
      1. GpuAllReduceRewrite patterns
      2. GpuGlobalIdRewriter patterns
      3. GpuShuffleRewriter patterns
  }];
  let assemblyFormat = "attr-dict";
}

```
- **EN**: Introduces declarations for `ApplyGPURewritePatternsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ApplyGPURewritePatternsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 87-96
```tablegen
def ApplyUnrollVectorsSubgroupMmaOp : Op<Transform_Dialect,
    "apply_patterns.gpu.unroll_vectors_subgroup_mma",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Unrolls contractions to the target `m`, `n`, and `k` native vector size,
    along with other vector operations based on expected usage. `transfer_read`
    ops unroll based on the extract slice shape introduced by unrolling the
    contractions, while elementwise and `transfer_write` ops unroll to the shape of
    the C matrix (`m x n`).

```
- **EN**: Introduces declarations for `ApplyUnrollVectorsSubgroupMmaOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ApplyUnrollVectorsSubgroupMmaOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 97-104
```tablegen
    This operation applies to pure vector operations and should be applied before
    lowering to subgroup_mma ops.
  }];

  let arguments = (ins I64Attr:$m,
                       I64Attr:$n,
                       I64Attr:$k);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 105-117
```tablegen
  let assemblyFormat = [{
    `[` $m `,` $n `,` $k `]` attr-dict
  }];
}

def EliminateBarriersOp :
  Op<Transform_Dialect, "apply_patterns.gpu.eliminate_barriers",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Removes unnecessary GPU barriers from the function. If a barrier does not
    enforce any conflicting pair of memory effects, including a pair that is
    enforced by another barrier, it is unnecessary and can be removed.

```
- **EN**: Introduces declarations for `EliminateBarriersOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `EliminateBarriersOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 118-125
```tablegen
    The approach is based on "High-Performance GPU-to-CPU Transpilation and
    Optimization via High-Level Parallel Constructs" by  Moses, Ivanov,
    Domke, Endo, Doerfert, and Zinenko in PPoPP 2023. Specifically, it
    analyzes the memory effects of the operations before and after the given
    barrier and checks if the barrier enforces any of the memory
    effect-induced dependencies that aren't already enforced by another
    barrier.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 126-135
```tablegen
    For example, in the following code

    ```mlir
      store %A
      barrier  // enforces load-after-store
      load %A
      barrier  // load-after-store already enforced by the previous barrier
      load %A
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 136-151
```tablegen
    the second barrier can be removed.
  }];

  let assemblyFormat = [{ attr-dict }];
}

def MapNestedForallToThreads :
  Op<Transform_Dialect, "gpu.map_nested_forall_to_threads",
    [FunctionalStyleTransformOpTrait,
     MemoryEffectsOpInterface,
     TransformEachOpTrait,
     TransformOpInterface]> {
  let description = [{
      Target the `gpu.launch op` and rewrite all `scf.forall` nested in it to
      distributed `gpu.thread_id` attribute.

```
- **EN**: Introduces declarations for `MapNestedForallToThreads`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MapNestedForallToThreads` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 152-167
```tablegen
      The operation searches for `scf.forall` ops nested under `target` and maps
      each such op to GPU threads.

      `scf.forall` induction variables are rewritten to `gpu.thread_id` according
      to the `mapping` attribute.

      Different types of mappings attributes are supported:
        - the block_dims is a list of integers that specifies the number of
          threads in each dimension. This is a mandatory attribute that is used
          to constrain the number of threads in each dimension. If an
          `scf.forall` op is mapped to fewer threads, predication occurs.
        - the warp_dims is a list of integers that specifies the number of
          warps in each dimension. This is an optional attribute that is used
          to constrain the number of warps in each dimension. When present, this
          attribute must be specified in a way that is compatible with the
          block_dims attribute. If an `scf.forall` op is mapped to fewer warps,
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 168-176
```tablegen
          predication occurs.

      Dynamic `scf.forall` trip counts are currently not supported.
      Dynamic block dim sizes are currently not supported.

      Only **bufferized** `scf.forall` are currently supported.
      Only `scf.forall` distributed to **at most 3 dimensions** are
      currently supported.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 177-185
```tablegen
      The `sync_after_distribute`attribute controls whether a `gpu.barrier` is
      inserted after each scf.forall op. At this time, this is an all or nothing
      choice. This will need to be tightened in the future.

      The operation alters the block size of the given gpu_launch using the
      mandatory block_dims argument.

      #### Return modes:

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 186-195
```tablegen
      This operation ignores non-`gpu_launch` ops and drops them in the return.

      If any scf.forall with tensors is found, the transform definitely
      fails.

      If all the `scf.forall` operations with gpu.thread mapping contained
      within the `LaunchOp` referred to by the `target` handle lower to GPU
      properly, the transform succeeds. Otherwise the transform definitely
      fails.

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 196-204
```tablegen
      scf.forall operations with mappings other than gpu.thread are
      ignored.

      The returned handle points to the same LaunchOp operand, consuming it and
      producing a new SSA value to satisfy chaining and linearity of the IR
      properties.

      #### Example:

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 205-217
```tablegen
      ```
      gpu.launch blocks(%bx, %by, %bz) in (%x = %0, %y = %1, %z = %2)
                 threads(%tx, %ty, %tz) in (%tx = %3, %ty = %4, %tz = %5) {
        scf.forall (%i, %j) in (7, 9) {
          ... // body 1
        } {mapping = [#gpu.thread<x>, #gpu.thread<y>, #gpu.thread<z>]}
        scf.forall (%i) in (12) {
          ... // body 2
        } {mapping = [#gpu.thread<x>]}
        gpu.terminator
      }
      ```

```
- **EN**: Implements logic around `blocks`, `threads`, `forall`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `blocks`, `threads`, `forall` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 218-233
```tablegen
      is translated to:

      ```
      %bdimX = arith.constant 12 : index
      %bdimY = arith.constant 9 : index
      gpu.launch blocks(%bx, %by, %bz) in (%x = %0, %y = %1, %z = %2)
             threads(%tx, %ty, %tz) in (%tx = %bdimX, %ty = %bdimY, %tz = %5) {
        if (threadIdx.x < 9 && threadIdx.y < 7) {
          ... // body 1
        }
        gpu.barrier
        if (threadIdx.y < 1) {
          ... // body 2
        }
        gpu.barrier
        gpu.terminator
```
- **EN**: Implements logic around `blocks`, `threads`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `blocks`, `threads` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 234-243
```tablegen
      }
      ```
    }];

  let arguments = (ins TransformHandleTypeInterface:$target,
                   DefaultValuedAttr<DenseI64ArrayAttr, "{}">:$block_dims,
                   DefaultValuedAttr<BoolAttr, "true">:$sync_after_distribute,
                   DefaultValuedAttr<I64Attr, "32">:$warp_size);
  let results = (outs TransformHandleTypeInterface:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 244-259
```tablegen
  let assemblyFormat = [{
    $target
    `block_dims` `=` $block_dims
    (`sync_after_distribute` `=` $sync_after_distribute^)?
    (`warp_size` `=` $warp_size^)?
    attr-dict
    `:` functional-type($target, $result)
  }];
  let extraClassDeclaration = [{
    ::mlir::DiagnosedSilenceableFailure applyToOne(
        ::mlir::transform::TransformRewriter &rewriter,
        ::mlir::Operation *target,
        ::mlir::transform::ApplyToEachResultList &results,
        ::mlir::transform::TransformState &state);
  }];
}
```
- **EN**: Implements logic around `type`, `applyToOne`; this block uses rewrite-pattern infrastructure to transform operations; handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type`, `applyToOne` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 260-272
```tablegen

def MapForallToBlocks :
  Op<Transform_Dialect, "gpu.map_forall_to_blocks",
    [FunctionalStyleTransformOpTrait,
     MemoryEffectsOpInterface,
     TransformOpInterface,
     TransformEachOpTrait]> {
  let description = [{
    Target the gpu_launch op and rewrite the top level `scf.forall`
    to distributed gpu.block_id attribute. If `generate_gpu_launch` attribute
    is set, then first generates `gpu_launch` and moves the top level
    `scf.forall` inside.

```
- **EN**: Introduces declarations for `MapForallToBlocks`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MapForallToBlocks` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 273-280
```tablegen
    The operation searches top level `scf.forall` ops under
    `gpu_launch` and maps each such op to GPU blocks. Mapping is
    one-to-one and the induction variables of `scf.forall` are
    rewritten to gpu.block_id according to the `thread_dim_mapping` attribute.

    Dynamic, `scf.forall` trip counts are currently not supported.
    Dynamic block dim sizes are currently not supported.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 281-289
```tablegen
    Only **bufferized** scf.forall are currently supported.
    Only scf.forall distributed to **at most 3 dimensions** are
    currently supported.

    The operation alters the block size of the given gpu_launch using the
    grid_dims argument.

    #### Return modes:

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 290-298
```tablegen
    This operation ignores non-gpu_launch ops and drops them in the return.

    If any scf.forall with tensors is found, the transform definitely
    fails.

    If all the `scf.forall` operations contained within the LaunchOp
    referred to by the `target` handle lower to GPU properly, the
    transform succeeds. Otherwise the transform definitely fails.

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 299-308
```tablegen
    The returned handle points to the same LaunchOp operand, consuming it and
    producing a new SSA value to satisfy chaining and linearity of the IR
    properties.
  }];

  let arguments = (ins TransformHandleTypeInterface:$target,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$grid_dims,
                   UnitAttr:$generate_gpu_launch);
  let results = (outs TransformHandleTypeInterface:$result);

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 309-317
```tablegen
  let assemblyFormat = [{
    $target
    (`generate_gpu_launch` $generate_gpu_launch^)?
    (`grid_dims` `=` $grid_dims^)?
    attr-dict
    `:` functional-type($target, $result)
  }];
  let hasVerifier = 1;

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 318-326
```tablegen
  let extraClassDeclaration = [{
    ::mlir::DiagnosedSilenceableFailure applyToOne(
        ::mlir::transform::TransformRewriter &rewriter,
        ::mlir::Operation *target,
        ::mlir::transform::ApplyToEachResultList &results,
        ::mlir::transform::TransformState &state);
  }];
}

```
- **EN**: Implements logic around `applyToOne`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `applyToOne` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 327-339
```tablegen
def ApplyGPUPromoteShuffleToAMDGPUPatternsOp : Op<Transform_Dialect,
    "apply_patterns.gpu.gpu_shuffle_to_amdgpu",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Collects patterns that are tryin to promote `gpu.shuffle`s to specialized
    AMDGPU intrinsics.
  }];
  let arguments = (ins OptionalAttr<StrAttr>:$chipset);
  let assemblyFormat = [{
    (`chipset` `=` $chipset^)? attr-dict
  }];
}

```
- **EN**: Introduces declarations for `ApplyGPUPromoteShuffleToAMDGPUPatternsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ApplyGPUPromoteShuffleToAMDGPUPatternsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 340-341
```tablegen

#endif // GPU_TRANSFORM_OPS
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Dialect conversion pipeline / 方言转换流水线**:
  - **EN**: Coordinates legality checks, type adaptation, and target-specific lowering.
  - **CN**: 协调合法性检查、类型适配以及目标相关 lowering。
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

- **TableGen includes / TableGen 包含**: `mlir/Dialect/Transform/IR/TransformDialect.td`, `mlir/Dialect/Transform/Interfaces/TransformInterfaces.td`, `mlir/Interfaces/SideEffectInterfaces.td`, `mlir/IR/OpBase.td`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (2), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
