# Passes.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Bufferization/Transforms/Passes.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the Bufferization dialect, focused on rewrite patterns, passes, and canonicalization helpers and `Passes`.
  - **CN**: 为 Bufferization 方言定义聚焦 `Passes` 的声明式 TableGen 规格，覆盖重写模式、Pass 与规范化辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```tablegen
//===-- Passes.td - Bufferization passes definition file ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 9-22
```tablegen
#ifndef MLIR_DIALECT_BUFFERIZATION_TRANSFORMS_PASSES
#define MLIR_DIALECT_BUFFERIZATION_TRANSFORMS_PASSES

include "mlir/Pass/PassBase.td"

def OwnershipBasedBufferDeallocationPass
    : Pass<"ownership-based-buffer-deallocation"> {
  let summary = "Adds all required dealloc operations for all allocations in "
                "the input program";
  let description = [{
    This pass implements an algorithm to automatically introduce all required
    deallocation operations for all buffers in the input program. This ensures
    that the resulting program does not have any memory leaks.

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 23-38
```tablegen
    The Buffer Deallocation pass operates on the level of operations
    implementing the FunctionOpInterface. Such operations can take MemRefs as
    arguments, but also return them. To ensure compatibility among all functions
    (including external ones), some rules have to be enforced. They are just
    assumed to hold for all external functions. Functions for which the
    definition is available ideally also already adhere to the ABI.
    Otherwise, all MemRef write operations in the input IR must dominate all
    MemRef read operations in the input IR. Then, the pass may modify the input
    IR by inserting `bufferization.clone` operations such that the output IR
    adheres to the function boundary ABI:
    * When a MemRef is passed as a function argument, ownership is never
      acquired. It is always the caller's responsibility to deallocate such
      MemRefs.
    * Returning a MemRef from a function always passes ownership to the caller,
      i.e., it is also the caller's responsibility to deallocate MemRefs
      returned from a called function.
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 39-54
```tablegen
    * A function must not return a MemRef with the same allocated base buffer as
      one of its arguments (in this case a copy has to be created). Note that in
      this context two subviews of the same buffer that don't overlap are also
      considered an alias.

    It is recommended to bufferize all operations first such that no tensor
    values remain in the IR once this pass is applied. That way all allocated
    MemRefs will be properly deallocated without any additional manual work.
    Otherwise, the pass that bufferizes the remaining tensors is responsible to
    add the corresponding deallocation operations. Note that this pass does not
    consider any values of tensor type and assumes that MemRef values defined by
    `bufferization.to_buffer` do not return ownership and do not have to be
    deallocated. `bufferization.to_tensor` operations are handled similarly to
    `bufferization.clone` operations with the exception that the result value is
    not handled because it's a tensor (not a MemRef).

```
- **EN**: Implements logic around `arguments`, `tensor`; this block packages logic as an MLIR pass or pass-related API; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `arguments`, `tensor` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 55-70
```tablegen
    Input

    ```mlir
    #map0 = affine_map<(d0) -> (d0)>
    module {
      func.func @condBranch(%arg0: i1,
                            %arg1: memref<2xf32>,
                            %arg2: memref<2xf32>) {
        cf.cond_br %arg0, ^bb1, ^bb2
      ^bb1:
        cf.br ^bb3(%arg1 : memref<2xf32>)
      ^bb2:
        %0 = memref.alloc() : memref<2xf32>
        linalg.generic {
          indexing_maps = [#map0, #map0],
          iterator_types = ["parallel"]}
```
- **EN**: Implements logic around `condBranch`, `bb3`, `alloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `condBranch`, `bb3`, `alloc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 71-83
```tablegen
        outs(%arg1, %0 : memref<2xf32>, memref<2xf32>) {
        ^bb0(%gen1_arg0: f32, %gen1_arg1: f32):
          %tmp1 = exp %gen1_arg0 : f32
          linalg.yield %tmp1 : f32
        }
        cf.br ^bb3(%0 : memref<2xf32>)
      ^bb3(%1: memref<2xf32>):
        "memref.copy"(%1, %arg2) : (memref<2xf32>, memref<2xf32>) -> ()
        return
      }
    }
    ```

```
- **EN**: Implements logic around `outs`, `bb0`, `bb3`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `outs`, `bb0`, `bb3` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 84-99
```tablegen
    Output

    ```mlir
    #map = affine_map<(d0) -> (d0)>
    module {
      func.func @condBranch(%arg0: i1,
                            %arg1: memref<2xf32>,
                            %arg2: memref<2xf32>) {
        %false = arith.constant false
        %true = arith.constant true
        cf.cond_br %arg0, ^bb1, ^bb2
      ^bb1:  // pred: ^bb0
        cf.br ^bb3(%arg1, %false : memref<2xf32>, i1)
      ^bb2:  // pred: ^bb0
        %alloc = memref.alloc() : memref<2xf32>
        linalg.generic {
```
- **EN**: Implements logic around `condBranch`, `bb3`, `alloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `condBranch`, `bb3`, `alloc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 100-115
```tablegen
          indexing_maps = [#map, #map],
          iterator_types = ["parallel"]}
        outs(%arg1, %alloc : memref<2xf32>, memref<2xf32>)
        ^bb0(%out: f32, %out_0: f32):
          %2 = math.exp %out : f32
          linalg.yield %2, %out_0 : f32, f32
        }
        cf.br ^bb3(%alloc, %true : memref<2xf32>, i1)
      ^bb3(%0: memref<2xf32>, %1: i1):  // 2 preds: ^bb1, ^bb2
        memref.copy %0, %arg2 : memref<2xf32> to memref<2xf32>
        %base_buffer, %offset, %sizes, %strides =
          memref.extract_strided_metadata %0 :
          memref<2xf32> -> memref<f32>, index, index, index
        bufferization.dealloc (%base_buffer : memref<f32>) if (%1)
        return
      }
```
- **EN**: Implements logic around `outs`, `bb0`, `bb3`, `dealloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `outs`, `bb0`, `bb3`, `dealloc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 116-131
```tablegen
    }
    ```

    The `private-function-dynamic-ownership` pass option allows the pass to add
    additional arguments to private functions to dynamically give ownership of
    MemRefs to callees. This can enable earlier deallocations and allows the
    pass to by-pass the function boundary ABI and thus potentially leading to
    fewer MemRef clones being inserted. For example, the private function
    ```mlir
    func.func private @passthrough(%memref: memref<2xi32>) -> memref<2xi32> {
      return %memref : memref<2xi32>
    }
    ```
    would be converted to
    ```mlir
    func.func private @passthrough(%memref: memref<2xi32>,
```
- **EN**: Implements logic around `passthrough`; this block packages logic as an MLIR pass or pass-related API; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `passthrough` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并协调核心结构化 MLIR 方言之间的行为。

### Lines 132-147
```tablegen
                                   %ownership: i1) -> (memref<2xi32>, i1) {
      return %memref, %ownership : memref<2xi32>, i1
    }
    ```
    and thus allows the returned MemRef to alias with the MemRef passed as
    argument (which would otherwise be forbidden according to the function
    boundary ABI).
  }];
  let options =
      [Option<
           "privateFuncDynamicOwnership", "private-function-dynamic-ownership",
           "bool", /*default=*/"false",
           "Allows to add additional arguments to private functions to "
           "dynamically pass ownership of memrefs to callees. This can enable "
           "earlier deallocations.">,
  ];
```
- **EN**: Implements logic around `argument`; this block packages logic as an MLIR pass or pass-related API; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `argument` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并协调核心结构化 MLIR 方言之间的行为。

### Lines 148-155
```tablegen

  let dependentDialects = [
    "mlir::bufferization::BufferizationDialect", "mlir::arith::ArithDialect",
    "mlir::memref::MemRefDialect", "mlir::scf::SCFDialect"
  ];
}

def BufferDeallocationSimplificationPass
```
- **EN**: Introduces declarations for `BufferDeallocationSimplificationPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `BufferDeallocationSimplificationPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 156-165
```tablegen
    : Pass<"buffer-deallocation-simplification"> {
  let summary = "Optimizes `bufferization.dealloc` operation for more "
                "efficient codegen";
  let description = [{
    This pass uses static alias analysis to reduce the number of alias checks
    required at runtime. Such checks are sometimes necessary to make sure that
    memrefs aren't deallocated before their last usage (use after free) or that
    some memref isn't deallocated twice (double free).
  }];

```
- **EN**: Implements logic around `usage`, `twice`; this block packages logic as an MLIR pass or pass-related API; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `usage`, `twice` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并协调核心结构化 MLIR 方言之间的行为。

### Lines 166-181
```tablegen
  let dependentDialects = [
    "mlir::bufferization::BufferizationDialect", "mlir::arith::ArithDialect",
    "mlir::memref::MemRefDialect"
  ];
}

def OptimizeAllocationLivenessPass
    : Pass<"optimize-allocation-liveness", "func::FuncOp"> {
  let summary = "This pass optimizes the liveness of temp allocations in the "
                "input function";
  let description = [{
      This pass will find all operations that have a memory allocation effect.
      It will search for the corresponding deallocation and move it right after
      the last user of the allocation.
      This will optimize the liveness of the allocations.

```
- **EN**: Introduces declarations for `OptimizeAllocationLivenessPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `OptimizeAllocationLivenessPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 182-197
```tablegen
      The pass is expected to run after the deallocation pipeline.
  }];
  let dependentDialects = ["mlir::memref::MemRefDialect"];
}

def LowerDeallocationsPass : Pass<"bufferization-lower-deallocations"> {
  let summary = "Lowers `bufferization.dealloc` operations to `memref.dealloc`"
                "operations";
  let description = [{
    This pass lowers `bufferization.dealloc` operations to the `memref` dialect.
    It can be applied to a `builtin.module` or operations implementing the
    `FunctionOpInterface`. For the latter, only simple `dealloc` operations can
    be lowered because the library function necessary for the fully generic
    lowering cannot be inserted. In this case, an error will be emitted.
    Next to `memref.dealloc` operations, it may also emit operations from the
    `arith`, `scf`, and `func` dialects to build conditional deallocations and
```
- **EN**: Introduces declarations for `LowerDeallocationsPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LowerDeallocationsPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 198-206
```tablegen
    library functions to avoid code-size blow-up.
  }];

  let dependentDialects = [
    "arith::ArithDialect", "memref::MemRefDialect", "scf::SCFDialect",
    "func::FuncDialect"
  ];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 207-215
```tablegen
def BufferHoistingPass : Pass<"buffer-hoisting", "func::FuncOp"> {
  let summary = "Optimizes placement of allocation operations by moving them "
                "into common dominators and out of nested regions";
  let description = [{
    This pass implements an approach to aggressively move allocations upwards
    into common dominators and out of nested regions.
  }];
}

```
- **EN**: Introduces declarations for `BufferHoistingPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `BufferHoistingPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 216-224
```tablegen
def BufferLoopHoistingPass : Pass<"buffer-loop-hoisting", "func::FuncOp"> {
  let summary = "Optimizes placement of allocation operations by moving them "
                "out of loop nests";
  let description = [{
    This pass implements an approach to aggressively move allocations upwards
    out of loop nests. It does not move allocations into common dominators.
  }];
}

```
- **EN**: Introduces declarations for `BufferLoopHoistingPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `BufferLoopHoistingPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 225-232
```tablegen
def BufferResultsToOutParamsPass
    : Pass<"buffer-results-to-out-params", "ModuleOp"> {
  let summary = "Converts memref-typed function results to out-params";
  let description = [{
    Some calling conventions prefer to pass output memrefs as "out params". The
    conversion to this calling convention must be done as an atomic
    transformation of the entire program (hence this is a module pass).

```
- **EN**: Introduces declarations for `BufferResultsToOutParamsPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `BufferResultsToOutParamsPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 233-240
```tablegen
    For example, if a call is rewritten, the callee needs to be rewritten
    otherwise the IR will end up invalid. Thus, this transformation
    require an atomic change to the entire program (e.g. the whole module).

    This pass is expected to run immediately after bufferization is finished.
    At that point, tensor-typed results will have been converted to memref-typed
    results, and can be consistently converted to out params.

```
- **EN**: Implements logic around `program`; this block packages logic as an MLIR pass or pass-related API; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `program` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并协调核心结构化 MLIR 方言之间的行为。

### Lines 241-256
```tablegen
    All memref-typed results are appended to the function argument list.

    The main issue with this pass (and the out-param calling convention) is that
    buffers for results need to be allocated in the caller. This currently only
    works for static shaped memrefs.

    If the hoist-static-allocs option is on, the pass tries to eliminate the
    allocation for the returned memref and avoid the memory-copy if possible.
    This optimization applies on the returned memref which has static shape and
    is allocated by memref.alloc in the function. It will use the memref given
    in function argument to replace the allocated memref.
  }];
  let options =
      [Option<"addResultAttribute", "add-result-attr", "bool",
              /*default=*/"false",
              "Add the attribute 'bufferize.result' to all output parameters.">,
```
- **EN**: Declares APIs or declarative rules around `pass`; this block packages logic as an MLIR pass or pass-related API; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `pass` 相关的 API 或声明式规则；该代码块将逻辑组织为 MLIR Pass 或相关 API，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 257-267
```tablegen
       Option<"hoistStaticAllocs", "hoist-static-allocs", "bool",
              /*default=*/"false", "Hoist static allocations to call sites.">,
       Option<"hoistDynamicAllocs", "hoist-dynamic-allocs", "bool",
              /*default=*/"false", "Hoist dynamic allocations to call sites.">,
       Option<"modifyPublicFunctions", "modify-public-functions", "bool",
              /*default=*/"false", "Modify function signatures of public "
              "functions.">,
  ];
  let dependentDialects = ["memref::MemRefDialect"];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 268-275
```tablegen
def DropEquivalentBufferResultsPass
    : Pass<"drop-equivalent-buffer-results", "ModuleOp"> {
  let summary = "Remove MemRef return values that are equivalent to a bbArg";
  let description = [{
    This pass removes MemRef return values from functions if they are equivalent
    to a function bbArg. In that case, the return value is redundant and the
    respective CallOp operand can be used at the call site.

```
- **EN**: Introduces declarations for `DropEquivalentBufferResultsPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DropEquivalentBufferResultsPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 276-286
```tablegen
    Note: If a bbArg buffer is not returned directly but casted to beforehand,
    the buffer is still considered equivalent.
  }];
  let options = [
    Option<"modifyPublicFunctions", "modify-public-functions", "bool",
           /*default=*/"false", "Modify function signatures of public "
           "functions.">,
  ];
  let dependentDialects = ["memref::MemRefDialect"];
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 287-296
```tablegen
def EmptyTensorToAllocTensorPass : Pass<"empty-tensor-to-alloc-tensor"> {
  let summary = "Replace all empty ops by alloc_tensor ops.";
  let description = [{
    tensor.empty ops return a tensor of unspecified contents who's only purpose
    is to carry the tensor shape. This pass converts such ops to
    bufferization.alloc_tensor ops, which bufferize to buffer allocations.
  }];
  let dependentDialects = ["tensor::TensorDialect"];
}

```
- **EN**: Introduces declarations for `EmptyTensorToAllocTensorPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `EmptyTensorToAllocTensorPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 297-305
```tablegen
def layoutMapClValues {
  string values = [{
  ::llvm::cl::values(
    clEnumValN(LayoutMapOption::InferLayoutMap, "infer-layout-map", ""),
    clEnumValN(LayoutMapOption::IdentityLayoutMap, "identity-layout-map", ""),
    clEnumValN(LayoutMapOption::FullyDynamicLayoutMap, "fully-dynamic-layout-map", "")
    )}];
}

```
- **EN**: Introduces declarations for `layoutMapClValues`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `layoutMapClValues` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 306-315
```tablegen
def OneShotBufferizePass : Pass<"one-shot-bufferize", "ModuleOp"> {
  let summary = "One-Shot Bufferize";
  let description = [{
    This pass bufferizes all ops that implement `BufferizableOpInterface`. It
    first performs an inplacability analysis on SSA use-def chains of tensor
    values to determine which OpOperands may bufferize in-place, i.e., without
    inserting a buffer copy. It then rewrites the IR, inserting a buffer
    allocation and copy for each OpOperand that was decided to bufferize
    out-of-place.

```
- **EN**: Introduces declarations for `OneShotBufferizePass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `OneShotBufferizePass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 316-326
```tablegen
    One-Shot Bufferize (and `BufferizableOpInterface`) was designed for ops that
    are in destination-passing style. When bufferizing such ops, it is possible
    to reuse the buffer of a tensor OpOperand for a tensor OpResult. In essence,
    a possible destination of an operation is already passed as an SSA value.

    `tensor.insert` is an example for an op in destination-passing style. E.g.,
    when bufferizing `%t0 = tensor.insert %f into %dest[%idx]`, `buffer(%t0)` is
    identical to `buffer(%dest)` in the absence of RaW conflicts. As a counter
    example, `tensor.generate` is not in destination-passing style and always
    results in a new buffer allocation.

```
- **EN**: Implements logic around `Bufferize`, `buffer`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Bufferize`, `buffer` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 327-338
```tablegen
    One-Shot Bufferize does not deallocate any buffers that it allocates. The
    `-buffer-deallocation-pipeline` pipeline should be run after One-Shot
    Bufferize to insert the deallocation operations necessary to eliminate
    memory leaks.

    One-Shot Bufferize will by default reject IR that contains non-bufferizable
    op, i.e., ops that do not implemement BufferizableOpInterface. Such IR can
    be allowed with `allow-unknown-ops=1`. In that case, to_buffer and to_tensor
    ops will be generated at the bufferization boundary. This is useful for
    compatibility with existing partial bufferization passes: These can
    bufferize the remaining IR after running One-Shot Bufferize.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 339-351
```tablegen
    Note: Running One-Shot Bufferize after a partial bufferization pass is
    currently not supported. Running partial bufferization passes after running
    One-Shot Bufferize is supported and the recommended way to gradually
    migrate from partial bufferization to One-Shot Bufferize.

    With `dialect-filter`, bufferization can be restricted to a set of dialects.
    If no filter is specified, all ops that implement `BufferizableOpInterface`
    are bufferized. Ops from the `std` dialect are an exception: These ops are
    always ignored, even if no filter is specified. When specifying a dialect
    filter and `allow-unknown-ops` is not turned on, bufferization would fail
    when encountering an op that is not included in the filter (even if it is
    bufferizable).

```
- **EN**: Implements logic around `filter`; this block packages logic as an MLIR pass or pass-related API; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `filter` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并协调核心结构化 MLIR 方言之间的行为。

### Lines 352-362
```tablegen
    One-Shot Bufferize will by default assume memref types with fully dynamic
    layout maps when a precise layout cannot be inferred. E.g., this is the case
    when wrapping a non-bufferizable op in to_buffer/to_tensor ops. This
    behavior can be overridden with `unknown-type-conversion`. Valid values are
    `fully-dynamic-layout-map` and `identity-layout-map`.

    For testing/debugging purposes, `test-analysis-only=1 print-conflicts=1`
    prints analysis results and explains why an OpOperand was decided to
    bufferize out-of-place. This is useful for understanding why One-Shot
    Bufferize chose to insert a certain buffer copy.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 363-378
```tablegen
    `bufferize-function-boundaries` is an experimental flag for bufferizing
    `FuncOp`, `ReturnOp` and `CallOp`. This feature is still under development
    and supports only simple cases at the moment. In particular:

    * Recursive or circular function call graphs are not supported.
    * External functions (without bodies) that return a tensor are not
      supported.
    * Function with multiple blocks or multiple ReturnOps are not supported.
    * Layout maps on function signatures can be controlled with a separate
      `function-boundary-type-conversion` option, which is similar to
      `unknown-type-conversion` but supports an additional `infer-layout-map`
      option. `fully-dynamic-layout-map` and `identity-layout-map` ensure that
      function signatures bufferize to easily predictable types, potentially at
      the cost of additional casts and copies, respectively. When layout maps
      are inferred, function return types may be more precise, but less
      predictable. Function argument types cannot be inferred and always have
```
- **EN**: Implements logic around `functions`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `functions` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 379-389
```tablegen
      fully dynamic layout maps with `infer-layout-map`.

    One-Shot Bufferize implements the following contract around function calls:
    The buffer of function arguments is always writable (unless annotated with
    `bufferization.writable = false`). A buffer copy may be inserted at the call
    site where necessary. Alias sets and equivalence info is propagated through
    function calls. Whenever a function is bufferized, all other functions that
    are being called were already analyzed and bufferized, so exact alias and
    equivalence information is available. This is why recursive function calls
    are not yet supported.

```
- **EN**: Implements logic around `writable`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `writable` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 390-400
```tablegen
    One-Shot Bufferize gathers additional information during the analysis phase
    when function boundary bufferization is activated. E.g., whether a function
    argument is read/written and which returned values are aliasing/equivalent.
    For debugging purposes, such information can be printed with
    `test-analysis-only`.

    The order in which ops are analyzed is important. The analysis is greedy and
    ops that are analyzed earlier are more likely to bufferize in-place. The
    heuristic can be set with `analysis-heuristic`. At the moment, the following
    heuristics are available:

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 401-416
```tablegen
    * `bottom-up` (default): Analyze ops from bottom to top.
    * `top-down`: Analyze ops from top to bottom.
    * `fuzzer`: Randomize the ordering of ops with `analysis-fuzzer-seed`.
    * `bottom-up-from-terminators`: Traverse the reverse use-def chains of
      tensor IR, starting from region branch terminators (bottom-up). Nested
      regions are traversed before enclosing regions. Analyze the traversed ops
      first, then analyze the remaining ops bottom-up. This heuristic is useful
      for bufferizing loop constructs. One-Shot Bufferize currently supports
      only such IR where yielded tensor values bufferize to equivalent region
      iter_args, and first analyzing all ops on the path from the "yielding" op
      to the beginning of the loop body makes it more likely for the region
      iter_args and yielded values to bufferize to equivalent buffers.
  }];
  let options =
      [Option<"allowReturnAllocsFromLoops", "allow-return-allocs-from-loops",
              "bool", /*default=*/"false",
```
- **EN**: Declares APIs or declarative rules around `terminators`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects; uses declarative TableGen records to describe reusable IR contracts.
- **CN**: 声明与 `terminators` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为，并使用声明式 TableGen 记录描述可复用 IR 契约。

### Lines 417-432
```tablegen
              "Allows returning/yielding new allocations from a loop.">,
       Option<"allowUnknownOps", "allow-unknown-ops", "bool",
              /*default=*/"false",
              "Allows unknown (not bufferizable) ops in the input IR.">,
       Option<
           "analysisFuzzerSeed", "analysis-fuzzer-seed", "unsigned",
           /*default=*/"0",
           "Test only: Analyze ops in random order with a given seed (fuzzer)">,
       Option<"analysisHeuristic", "analysis-heuristic", "std::string",
              /*default=*/"\"bottom-up\"",
              "Heuristic that control the IR traversal during analysis">,
       Option<"bufferizeFunctionBoundaries", "bufferize-function-boundaries",
              "bool", /*default=*/"0",
              "Bufferize function boundaries (experimental).">,
       Option<"checkParallelRegions", "check-parallel-regions", "bool",
              /*default=*/"true",
```
- **EN**: Implements logic around `unknown`, `seed`, `boundaries`.
- **CN**: 围绕 `unknown`, `seed`, `boundaries` 实现具体逻辑。

### Lines 433-448
```tablegen
              "Account for parallel regions in RaW analysis.">,
       Option<"copyBeforeWrite", "copy-before-write", "bool",
              /*default=*/"false",
              "Skip the analysis. Make a buffer copy on every write.">,
       ListOption<"dialectFilter", "dialect-filter", "std::string",
                  "Restrict bufferization to ops from these dialects.">,
       Option<"dumpAliasSets", "dump-alias-sets", "bool", /*default=*/"false",
              "Test only: Annotate tensor IR with alias sets">,
       ListOption<"noAnalysisFuncFilter", "no-analysis-func-filter",
                  "std::string",
                  "Skip analysis of functions with these symbol names."
                  "Set copyBeforeWrite to true when bufferizing them.">,
       Option<"functionBoundaryTypeConversion",
              "function-boundary-type-conversion", "LayoutMapOption",
              /*default=*/"LayoutMapOption::InferLayoutMap",
              "Controls layout maps when bufferizing function signatures.",
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 449-464
```tablegen
              layoutMapClValues.values>,
       Option<"mustInferMemorySpace", "must-infer-memory-space", "bool",
              /*default=*/"false",
              "The memory space of an memref types must always be inferred. If "
              "unset, a default memory space of 0 is used otherwise.">,
       Option<"useEncodingForMemorySpace", "use-encoding-for-memory-space",
              "bool",
              /*default=*/"false",
              "Use the Tensor encoding attribute for the memory space. "
              "Exclusive to"
              " the 'must-infer-memory-space' option">,
       Option<"testAnalysisOnly", "test-analysis-only", "bool",
              /*default=*/"false",
              "Test only: Only run inplaceability analysis and annotate IR">,
       Option<"printConflicts", "print-conflicts", "bool",
              /*default=*/"false",
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 465-475
```tablegen
              "Test only: Annotate IR with RaW conflicts. Requires "
              "test-analysis-only.">,
       Option<"unknownTypeConversion", "unknown-type-conversion", "LayoutMapOption",
              /*default=*/"LayoutMapOption::FullyDynamicLayoutMap",
              "Controls layout maps for non-inferrable memref types.",
              layoutMapClValues.values>,
       Option<"bufferAlignment", "buffer-alignment", "uint64_t",
              /*default=*/"64",
              "Sets the alignment of newly allocated buffers.">,
  ];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 476-484
```tablegen
  let statistics = [
    Statistic<"numBufferAlloc", "num-buffer-alloc",
              "Number of buffer allocations">,
    Statistic<"numTensorInPlace", "num-tensor-in-place",
              "Number of in-place tensor OpOperands">,
    Statistic<"numTensorOutOfPlace", "num-tensor-out-of-place",
              "Number of out-of-place tensor OpOperands">,
  ];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 485-500
```tablegen
  let dependentDialects = [
    "bufferization::BufferizationDialect", "memref::MemRefDialect"
  ];
}

def PromoteBuffersToStackPass
    : Pass<"promote-buffers-to-stack", "func::FuncOp"> {
  let summary = "Promotes heap-based allocations to automatically managed "
                "stack-based allocations";
  let description = [{
    This pass implements a simple algorithm to convert heap-based memory
    allocations to stack-based ones. It uses a built-in heuristic to decide
    whether it makes sense to convert an allocation. Furthermore, dynamic
    shaped buffers that are limited by the rank of the tensor can be
    converted. They are only transformed if they are considered to be small.
  }];
```
- **EN**: Introduces declarations for `PromoteBuffersToStackPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `PromoteBuffersToStackPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 501-510
```tablegen
  let options = [
    Option<"maxAllocSizeInBytes", "max-alloc-size-in-bytes", "unsigned",
           /*default=*/"1024",
           "Maximal size in bytes to promote allocations to stack.">,
    Option<"maxRankOfAllocatedMemRef", "max-rank-of-allocated-memref", "unsigned",
           /*default=*/"1",
           "Maximal memref rank to promote dynamic buffers.">,
  ];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 511-518
```tablegen
def EmptyTensorEliminationPass : Pass<"eliminate-empty-tensors"> {
  let summary = "Try to eliminate all tensor.empty ops.";
  let description = [{
    Try to eliminate "tensor.empty" ops inside `op`. This transformation looks
    for subset ops that insert a tensor that originates from a "tensor.empty"
    (as per the reverse use-def chain). Such "tensor.empty" ops are replaced
    with the destination subset.

```
- **EN**: Introduces declarations for `EmptyTensorEliminationPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `EmptyTensorEliminationPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 519-532
```tablegen
    E.g.:
    ```
    %0 = tensor.empty() : tensor<10xf32>
    %1 = linalg.fill ... outs(%0 : tensor<10xf32>)
    %2 = tensor.insert_slice %1 into %t ...
    ```

    In the above example, the subset op is "tensor.insert_slice". When tracing
    back the reverse use-def chain of a the source, we end up at a
    "tensor.empty" op. The "tensor.empty" op is replaced with a
    "tensor.extract_slice" op.
  }];
}

```
- **EN**: Declares APIs or declarative rules around `empty`, `outs`; this block coordinates behavior across core structured MLIR dialects; uses declarative TableGen records to describe reusable IR contracts.
- **CN**: 声明与 `empty`, `outs` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为，并使用声明式 TableGen 记录描述可复用 IR 契约。

### Lines 533-533
```tablegen
#endif // MLIR_DIALECT_BUFFERIZATION_TRANSFORMS_PASSES
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Rewrite and lowering support / 重写与 lowering 支持**:
  - **EN**: Collects patterns or pass declarations used to canonicalize, legalize, or lower IR.
  - **CN**: 汇集用于规范化、合法化或 lowering IR 的模式或 Pass 声明。
- **Textual assembly syntax / 文本汇编语法**:
  - **EN**: Implements dialect-specific parsing and printing hooks for MLIR assembly.
  - **CN**: 实现 MLIR 汇编的方言专用解析与打印钩子。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, interfaces, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束、接口或规范化规则。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Pass/PassBase.td`
- **Subsystem categories / 子系统类别**: pass infrastructure and registration support / Pass 基础设施与注册支持 (1)
