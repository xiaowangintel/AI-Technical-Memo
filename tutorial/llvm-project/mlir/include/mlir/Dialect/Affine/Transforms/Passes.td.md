# Passes.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/Transforms/Passes.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains definitions for passes within the Affine/ directory.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine/Transforms`，围绕 Affine 方言公开 `Passes` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- Passes.td - Affine pass definition file ------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains definitions for passes within the Affine/ directory.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 12-27
```tablegen

#ifndef MLIR_DIALECT_AFFINE_PASSES
#define MLIR_DIALECT_AFFINE_PASSES

include "mlir/Pass/PassBase.td"

def AffineDataCopyGeneration : Pass<"affine-data-copy-generate", "func::FuncOp"> {
  let summary = "Generate explicit copying for affine memory operations";
  let constructor = "mlir::affine::createAffineDataCopyGenerationPass()";
  let dependentDialects = ["memref::MemRefDialect"];
  let options = [
    Option<"fastMemoryCapacity", "fast-mem-capacity", "uint64_t",
           /*default=*/"std::numeric_limits<uint64_t>::max()",
           "Set fast memory space capacity in KiB (default: unlimited)">,
    Option<"fastMemorySpace", "fast-mem-space", "unsigned",
           /*default=*/"1",
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 28-43
```tablegen
           "Fast memory space identifier for copy generation (default: 1)">,
    Option<"generateDma", "generate-dma", "bool",
           /*default=*/"false", "Generate DMA instead of point-wise copy">,
    Option<"minDmaTransferSize", "min-dma-transfer", "int",
           /*default=*/"1024",
           "Minimum DMA transfer size supported by the target in bytes">,
    Option<"slowMemorySpace", "slow-mem-space", "unsigned",
           /*default=*/"0",
           "Slow memory space identifier for copy generation (default: 0)">,
    Option<"skipNonUnitStrideLoops", "skip-non-unit-stride-loops", "bool",
           /*default=*/"false", "Testing purposes: avoid non-unit stride loop "
                                "choice depths for copy placement">,
    Option<"tagMemorySpace", "tag-mem-space", "unsigned",
           /*default=*/"0",
           "Tag memory space identifier for copy generation (default: 0)">,
  ];
```
- **EN**: Declares APIs or declarative rules around `generation`.
- **CN**: 声明与 `generation` 相关的 API 或声明式规则。

### Lines 44-59
```tablegen
}

def AffineLoopFusion : Pass<"affine-loop-fusion"> {
  let summary = "Fuse affine loop nests";
  let description = [{
    This pass performs fusion of loop nests using a slicing-based approach. The
    transformation works on an MLIR `Block` granularity and applies to all
    blocks of the pass is run on. It combines two fusion strategies:
    producer-consumer fusion and sibling fusion. Producer-consumer fusion is
    aimed at fusing pairs of loops where the first one writes to a memref that
    the second reads. Sibling fusion targets pairs of loops that share no
    dependences between them but that load from the same memref. The fused loop
    nests, when possible, are rewritten to access significantly smaller local
    buffers instead of the original memref's, and the latter are often either
    completely optimized away or contracted. This transformation leads to
    enhanced locality and lower memory footprint through the elimination or
```
- **EN**: Introduces declarations for `AffineLoopFusion`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineLoopFusion` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 60-75
```tablegen
    contraction of temporaries/intermediate memref's. These benefits are
    sometimes achieved at the expense of redundant computation through a cost
    model that evaluates available choices such as the depth at which a source
    slice should be materialized in the designation slice.

    Example 1: Producer-consumer fusion.
    Input:
    ```mlir
    func.func @producer_consumer_fusion(%arg0: memref<10xf32>, %arg1: memref<10xf32>) {
      %0 = memref.alloc() : memref<10xf32>
      %1 = memref.alloc() : memref<10xf32>
      %cst = arith.constant 0.000000e+00 : f32
      affine.for %arg2 = 0 to 10 {
        affine.store %cst, %0[%arg2] : memref<10xf32>
        affine.store %cst, %1[%arg2] : memref<10xf32>
      }
```
- **EN**: Implements logic around `producer_consumer_fusion`, `alloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `producer_consumer_fusion`, `alloc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 76-91
```tablegen
      affine.for %arg2 = 0 to 10 {
        %2 = affine.load %0[%arg2] : memref<10xf32>
        %3 = arith.addf %2, %2 : f32
        affine.store %3, %arg0[%arg2] : memref<10xf32>
      }
      affine.for %arg2 = 0 to 10 {
        %2 = affine.load %1[%arg2] : memref<10xf32>
        %3 = arith.mulf %2, %2 : f32
        affine.store %3, %arg1[%arg2] : memref<10xf32>
      }
      return
    }
    ```
    Output:
    ```mlir
    func.func @producer_consumer_fusion(%arg0: memref<10xf32>, %arg1: memref<10xf32>) {
```
- **EN**: Implements logic around `producer_consumer_fusion`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `producer_consumer_fusion` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 92-107
```tablegen
      %0 = memref.alloc() : memref<1xf32>
      %1 = memref.alloc() : memref<1xf32>
      %cst = arith.constant 0.000000e+00 : f32
      affine.for %arg2 = 0 to 10 {
        affine.store %cst, %0[0] : memref<1xf32>
        affine.store %cst, %1[0] : memref<1xf32>
        %2 = affine.load %1[0] : memref<1xf32>
        %3 = arith.mulf %2, %2 : f32
        affine.store %3, %arg1[%arg2] : memref<10xf32>
        %4 = affine.load %0[0] : memref<1xf32>
        %5 = arith.addf %4, %4 : f32
        affine.store %5, %arg0[%arg2] : memref<10xf32>
      }
      return
    }
    ```
```
- **EN**: Implements logic around `alloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `alloc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 108-123
```tablegen

    Example 2: Sibling fusion.
    Input:
    ```mlir
    func.func @sibling_fusion(%arg0: memref<10x10xf32>, %arg1: memref<10x10xf32>,
                         %arg2: memref<10x10xf32>, %arg3: memref<10x10xf32>,
                         %arg4: memref<10x10xf32>) {
      affine.for %arg5 = 0 to 3 {
        affine.for %arg6 = 0 to 3 {
          %0 = affine.load %arg0[%arg5, %arg6] : memref<10x10xf32>
          %1 = affine.load %arg1[%arg5, %arg6] : memref<10x10xf32>
          %2 = arith.mulf %0, %1 : f32
          affine.store %2, %arg3[%arg5, %arg6] : memref<10x10xf32>
        }
      }
      affine.for %arg5 = 0 to 3 {
```
- **EN**: Implements logic around `sibling_fusion`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `sibling_fusion` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 124-139
```tablegen
        affine.for %arg6 = 0 to 3 {
          %0 = affine.load %arg0[%arg5, %arg6] : memref<10x10xf32>
          %1 = affine.load %arg2[%arg5, %arg6] : memref<10x10xf32>
          %2 = arith.addf %0, %1 : f32
          affine.store %2, %arg4[%arg5, %arg6] : memref<10x10xf32>
        }
      }
      return
    }
    ```
    Output:
    ```mlir
    func.func @sibling_fusion(%arg0: memref<10x10xf32>, %arg1: memref<10x10xf32>,
                         %arg2: memref<10x10xf32>, %arg3: memref<10x10xf32>,
                         %arg4: memref<10x10xf32>) {
      affine.for %arg5 = 0 to 3 {
```
- **EN**: Implements logic around `sibling_fusion`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `sibling_fusion` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 140-155
```tablegen
        affine.for %arg6 = 0 to 3 {
          %0 = affine.load %arg0[%arg5, %arg6] : memref<10x10xf32>
          %1 = affine.load %arg1[%arg5, %arg6] : memref<10x10xf32>
          %2 = arith.mulf %0, %1 : f32
          affine.store %2, %arg3[%arg5, %arg6] : memref<10x10xf32>
          %3 = affine.load %arg0[%arg5, %arg6] : memref<10x10xf32>
          %4 = affine.load %arg2[%arg5, %arg6] : memref<10x10xf32>
          %5 = arith.addf %3, %4 : f32
          affine.store %5, %arg4[%arg5, %arg6] : memref<10x10xf32>
        }
      }
      return
    }
    ```
  }];
  let constructor = "mlir::affine::createLoopFusionPass()";
```
- **EN**: Implements logic around `createLoopFusionPass`; this block packages logic as an MLIR pass or pass-related API; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createLoopFusionPass` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并协调核心结构化 MLIR 方言之间的行为。

### Lines 156-171
```tablegen
  let options = [
    Option<"computeToleranceThreshold", "compute-tolerance", "double",
           /*default=*/"0.30f", "Fractional increase in additional computation "
                                "tolerated while fusing">,
    Option<"fastMemorySpace", "fast-mem-space", "unsigned",
           /*default=*/"0",
           "Faster memory space number to promote fusion buffers to">,
    Option<"localBufSizeThreshold", "local-buf-threshold", "uint64_t",
           /*default=*/"0", "Threshold size (KiB) for promoting local buffers "
                            "to fast memory space">,
    Option<"maximalFusion", "maximal", "bool", /*default=*/"false",
           "Enables maximal loop fusion">,
    Option<"affineFusionMode", "mode", "enum FusionMode",
           "mlir::affine::FusionMode::Greedy", "fusion mode to attempt",
           "llvm::cl::values(clEnumValN(mlir::affine::FusionMode::Greedy,"
           " \"greedy\", \"Perform greedy (both producer-consumer and sibling)  fusion\"), "
```
- **EN**: Introduces declarations for `FusionMode`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `FusionMode` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 172-179
```tablegen
           "clEnumValN( mlir::affine::FusionMode::ProducerConsumer, "
           "\"producer\", \"Perform only producer-consumer fusion\"), "
           "clEnumValN( mlir::affine::FusionMode::Sibling, "
           "\"sibling\", \"Perform only sibling fusion\"))">,
    ];
  let dependentDialects = ["memref::MemRefDialect"];
}

```
- **EN**: Declares APIs or declarative rules around `clEnumValN`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `clEnumValN` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 180-195
```tablegen
def AffineLoopInvariantCodeMotion
    : Pass<"affine-loop-invariant-code-motion", "func::FuncOp"> {
  let summary = "Hoist loop invariant instructions outside of affine loops";
  let constructor = "mlir::affine::createAffineLoopInvariantCodeMotionPass()";
}

def AffineLoopTiling : Pass<"affine-loop-tile", "func::FuncOp"> {
  let summary = "Tile affine loop nests";
  let constructor = "mlir::affine::createLoopTilingPass()";
  let options = [
    Option<"cacheSizeInKiB", "cache-size", "uint64_t", /*default=*/"512",
           "Set size of cache to tile for in KiB (default: 512)">,
    Option<"separate", "separate", "bool", /*default=*/"false",
           "Separate full and partial tiles (default: false)">,
    Option<"tileSize", "tile-size", "unsigned", /*default=*/"",
           "Use this tile size for all loops">,
```
- **EN**: Introduces declarations for `AffineLoopInvariantCodeMotion`, `AffineLoopTiling`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineLoopInvariantCodeMotion`, `AffineLoopTiling` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 196-211
```tablegen
    ListOption<"tileSizes", "tile-sizes", "unsigned",
               "List of tile sizes for each perfect nest "
               "(overridden by -tile-size)">,
  ];
}

def AffineLoopUnroll : InterfacePass<"affine-loop-unroll", "FunctionOpInterface"> {
  let summary = "Unroll affine loops";
  let constructor = "mlir::affine::createLoopUnrollPass()";
  let options = [
    Option<"unrollFactor", "unroll-factor", "int64_t", /*default=*/"4",
           "Use this unroll factor for all loops being unrolled, "
           "set it to -1, and it will fully unroll.">,
    Option<"unrollUpToFactor", "unroll-up-to-factor", "bool",
           /*default=*/"false", "Allow unrolling up to the factor specified">,
    Option<"numRepetitions", "unroll-num-reps", "unsigned", /*default=*/"1",
```
- **EN**: Introduces declarations for `AffineLoopUnroll`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineLoopUnroll` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 212-220
```tablegen
           "Unroll innermost loops repeatedly this many times">,
    Option<"unrollFullThreshold", "unroll-full-threshold", "unsigned",
           /*default=*/"1",
           "Unroll all loops with trip count less than or equal to this">,
    Option<"cleanUpUnroll", "cleanup-unroll", "bool", /*default=*/"false",
           "Fully unroll the cleanup loop when possible.">,
  ];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 221-230
```tablegen
def AffineLoopUnrollAndJam : InterfacePass<"affine-loop-unroll-jam", "FunctionOpInterface"> {
  let summary = "Unroll and jam affine loops";
  let constructor = "mlir::affine::createLoopUnrollAndJamPass()";
  let options = [
    Option<"unrollJamFactor", "unroll-jam-factor", "unsigned",
           /*default=*/"4",
           "Use this unroll jam factor for all loops (default 4)">,
  ];
}

```
- **EN**: Introduces declarations for `AffineLoopUnrollAndJam`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineLoopUnrollAndJam` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 231-239
```tablegen
def AffinePipelineDataTransfer
    : Pass<"affine-pipeline-data-transfer", "func::FuncOp"> {
  let summary = "Pipeline non-blocking data transfers between explicitly "
                "managed levels of the memory hierarchy";
  let description = [{
    This pass performs a transformation to overlap non-blocking DMA operations
    in a loop with computations through double buffering. This is achieved by
    advancing dma_start operations with respect to other operations.

```
- **EN**: Introduces declarations for `AffinePipelineDataTransfer`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffinePipelineDataTransfer` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 240-255
```tablegen
    Input

    ```mlir
    func.func @pipelinedatatransfer() {
      %0 = memref.alloc() : memref<256xf32>
      %1 = memref.alloc() : memref<32xf32, 1>
      %2 = memref.alloc() : memref<1xf32>
      %c0 = arith.constant 0 : index
      %c128 = arith.constant 128 : index
      affine.for %i0 = 0 to 8 {
        affine.dma_start %0[%i0], %1[%i0], %2[%c0], %c128 : memref<256xf32>, memref<32xf32, 1>, memref<1xf32>
        affine.dma_wait %2[%c0], %c128 : memref<1xf32>
        %3 = affine.load %1[%i0] : memref<32xf32, 1>
        %4 = "compute"(%3) : (f32) -> f32
        affine.store %4, %1[%i0] : memref<32xf32, 1>
      }
```
- **EN**: Implements logic around `pipelinedatatransfer`, `alloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `pipelinedatatransfer`, `alloc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 256-271
```tablegen
      return
    }
    ```

    Output

    ```mlir
    module {
      func.func @pipelinedatatransfer() {
        %c8 = arith.constant 8 : index
        %c0 = arith.constant 0 : index
        %0 = memref.alloc() : memref<256xf32>
        %c0_0 = arith.constant 0 : index
        %c128 = arith.constant 128 : index
        %1 = memref.alloc() : memref<2x32xf32, 1>
        %2 = memref.alloc() : memref<2x1xf32>
```
- **EN**: Implements logic around `pipelinedatatransfer`, `alloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `pipelinedatatransfer`, `alloc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 272-287
```tablegen
        affine.dma_start %0[%c0], %1[%c0 mod 2, %c0], %2[%c0 mod 2, symbol(%c0_0)], %c128 : memref<256xf32>, memref<2x32xf32, 1>, memref<2x1xf32>
        affine.for %arg0 = 1 to 8 {
          affine.dma_start %0[%arg0], %1[%arg0 mod 2, %arg0], %2[%arg0 mod 2, symbol(%c0_0)], %c128 : memref<256xf32>, memref<2x32xf32, 1>, memref<2x1xf32>
          %8 = affine.apply #map3(%arg0)
          %9 = affine.apply #map4(%8)
          %10 = affine.apply #map4(%8)
          affine.dma_wait %2[%8 mod 2, symbol(%c0_0)], %c128 : memref<2x1xf32>
          %11 = affine.load %1[%8 mod 2, %8] : memref<2x32xf32, 1>
          %12 = "compute"(%11) : (f32) -> f32
          affine.store %12, %1[%8 mod 2, %8] : memref<2x32xf32, 1>
        }
        %3 = affine.apply #map3(%c8)
        %4 = affine.apply #map4(%3)
        %5 = affine.apply #map4(%3)
        affine.dma_wait %2[%3 mod 2, symbol(%c0_0)], %c128 : memref<2x1xf32>
        %6 = affine.load %1[%3 mod 2, %3] : memref<2x32xf32, 1>
```
- **EN**: Implements logic around `symbol`, `map3`, `map4`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `symbol`, `map3`, `map4` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 288-299
```tablegen
        %7 = "compute"(%6) : (f32) -> f32
        affine.store %7, %1[%3 mod 2, %3] : memref<2x32xf32, 1>
        memref.dealloc %2 : memref<2x1xf32>
        memref.dealloc %1 : memref<2x32xf32, 1>
        return
      }
    }
    ```
  }];
  let constructor = "mlir::affine::createPipelineDataTransferPass()";
}

```
- **EN**: Declares APIs or declarative rules around `createPipelineDataTransferPass`; this block packages logic as an MLIR pass or pass-related API; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `createPipelineDataTransferPass` 相关的 API 或声明式规则；该代码块将逻辑组织为 MLIR Pass 或相关 API，并协调核心结构化 MLIR 方言之间的行为。

### Lines 300-307
```tablegen
def AffineScalarReplacement : Pass<"affine-scalrep", "func::FuncOp"> {
  let summary = "Replace affine memref accesses by scalars by forwarding stores "
                "to loads and eliminating redundant loads";
  let description = [{
    This pass performs store to load forwarding and redundant load elimination
    for affine memref accesses and potentially eliminates the entire memref
    if all its accesses are forwarded.

```
- **EN**: Introduces declarations for `AffineScalarReplacement`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineScalarReplacement` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 308-323
```tablegen
    Input

    ```mlir
    func.func @store_load_affine_apply() -> memref<10x10xf32> {
      %cf7 = arith.constant 7.0 : f32
      %m = memref.alloc() : memref<10x10xf32>
      affine.for %i0 = 0 to 10 {
        affine.for %i1 = 0 to 10 {
          affine.store %cf7, %m[%i0, %i1] : memref<10x10xf32>
          %v0 = affine.load %m[%i0, %i1] : memref<10x10xf32>
          %v1 = arith.addf %v0, %v0 : f32
        }
      }
      return %m : memref<10x10xf32>
    }
    ```
```
- **EN**: Implements logic around `store_load_affine_apply`, `alloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `store_load_affine_apply`, `alloc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 324-339
```tablegen

    Output

    ```mlir
    module {
      func.func @store_load_affine_apply() -> memref<10x10xf32> {
        %cst = arith.constant 7.000000e+00 : f32
        %0 = memref.alloc() : memref<10x10xf32>
        affine.for %arg0 = 0 to 10 {
          affine.for %arg1 = 0 to 10 {
            affine.store %cst, %0[%arg0, %arg1] : memref<10x10xf32>
            %1 = arith.addf %cst, %cst : f32
          }
        }
        return %0 : memref<10x10xf32>
      }
```
- **EN**: Implements logic around `store_load_affine_apply`, `alloc`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `store_load_affine_apply`, `alloc` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 340-355
```tablegen
    }
    ```
  }];
  let constructor = "mlir::affine::createAffineScalarReplacementPass()";
}

def AffineVectorize : Pass<"affine-super-vectorize", "func::FuncOp"> {
  let summary = "Vectorize to a target independent n-D vector abstraction";
  let dependentDialects = ["vector::VectorDialect"];
  let options = [
    ListOption<"vectorSizes", "virtual-vector-size", "int64_t",
               "Specify an n-D virtual vector size for vectorization. "
               "This must be greater than zero.">,
    // Optionally, the fixed mapping from loop to fastest varying MemRef
    // dimension for all the MemRefs within a loop pattern:
    //   the index represents the loop depth, the value represents the k^th
```
- **EN**: Introduces declarations for `AffineVectorize`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineVectorize` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 356-369
```tablegen
    //   fastest varying memory dimension.
    // This is voluntarily restrictive and is meant to precisely target a
    // particular loop/op pair, for testing purposes.
    ListOption<"fastestVaryingPattern", "test-fastest-varying", "int64_t",
               "Specify a 1-D, 2-D or 3-D pattern of fastest varying memory "
               "dimensions to match. See defaultPatterns in Vectorize.cpp for "
               "a description and examples. This is used for testing purposes">,
    Option<"vectorizeReductions", "vectorize-reductions", "bool",
           /*default=*/"false",
           "Vectorize known reductions expressed via iter_args. "
           "Switched off by default.">
  ];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 370-381
```tablegen
def AffineParallelize : Pass<"affine-parallelize", "func::FuncOp"> {
  let summary = "Convert affine.for ops into 1-D affine.parallel";
  let options = [
    Option<"maxNested", "max-nested", "unsigned", /*default=*/"-1u",
           "Maximum number of nested parallel loops to produce. "
           "Defaults to unlimited (UINT_MAX).">,
    Option<"parallelReductions", "parallel-reductions", "bool",
           /*default=*/"false",
           "Whether to parallelize reduction loops. Defaults to false.">
  ];
}

```
- **EN**: Introduces declarations for `AffineParallelize`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineParallelize` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 382-390
```tablegen
def AffineLoopNormalize : Pass<"affine-loop-normalize", "func::FuncOp"> {
  let summary = "Apply normalization transformations to affine loop-like ops";
  let constructor = "mlir::affine::createAffineLoopNormalizePass()";
  let options = [
    Option<"promoteSingleIter", "promote-single-iter", "bool",
           /*default=*/"true", "Promote single iteration loops">,
  ];
}

```
- **EN**: Introduces declarations for `AffineLoopNormalize`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineLoopNormalize` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 391-398
```tablegen
def LoopCoalescing : Pass<"affine-loop-coalescing", "func::FuncOp"> {
  let summary = "Coalesce nested loops with independent bounds into a single "
                "loop";
  let constructor = "mlir::affine::createLoopCoalescingPass()";
  let dependentDialects = ["affine::AffineDialect","arith::ArithDialect"];
}

def RaiseMemrefDialect : Pass<"affine-raise-from-memref", "func::FuncOp"> {
```
- **EN**: Introduces declarations for `LoopCoalescing`, `RaiseMemrefDialect`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LoopCoalescing`, `RaiseMemrefDialect` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 399-409
```tablegen
  let summary = "Turn some memref operators to affine operators where supported";
  let description = [{
    Raise memref.load and memref.store to affine.store and affine.load, inferring
    the affine map of those operators if needed. This allows passes like --affine-scalrep
    to optimize those loads and stores (forwarding them or eliminating them).
    They can be turned back to memref dialect ops with --lower-affine.
  }];
  let constructor = "mlir::affine::createRaiseMemrefToAffine()";
  let dependentDialects = ["affine::AffineDialect"];
}

```
- **EN**: Implements logic around `stores`, `createRaiseMemrefToAffine`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `stores`, `createRaiseMemrefToAffine` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 410-425
```tablegen
def SimplifyAffineStructures : Pass<"affine-simplify-structures", "func::FuncOp"> {
  let summary = "Simplify affine expressions in maps/sets and normalize "
                "memrefs";
  let constructor = "mlir::affine::createSimplifyAffineStructuresPass()";
}

def SimplifyAffineMinMaxPass : InterfacePass<"affine-simplify-min-max", "FunctionOpInterface"> {
  let summary = "Simplify affine min/max/apply";
  let description = [{
    Apply the SimplifyAffineMaxOp, SimplifyAffineMinOp and SimplifyAffineApplyOp
    patterns in addition to AffineMin/Max canonicalization patterns until a
    fixed point is reached.
    These patterns apply ValueBoundsOp interface on AffineMin/Max ops and
    additional simplifications such as:
    ```
       min(x, y, cst) / cst -> 1
```
- **EN**: Introduces declarations for `SimplifyAffineStructures`, `SimplifyAffineMinMaxPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SimplifyAffineStructures`, `SimplifyAffineMinMaxPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 426-433
```tablegen
    ```
    when x, y, cst are all >= 0.
    This is typically useful to extract more static informationfrom IR after
    tiling but can also come at a cost due to Presburger-style analysis.
  }];
}

def SimplifyAffineWithBounds : Pass<"affine-simplify-with-bounds"> {
```
- **EN**: Introduces declarations for `SimplifyAffineWithBounds`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SimplifyAffineWithBounds` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 434-444
```tablegen
  let summary = "Simplify affine index operations using value bounds analysis";
  let description = [{
    This pass simplifies `affine.delinearize_index` / `affine.linearize_index`
    pairs by using value bounds analysis to match basis products. Unlike the
    built-in canonicalization patterns which only use exact `OpFoldResult`
    comparisons, this pass can prove equality of dynamic basis products through
    `ValueBoundsConstraintSet`.
  }];
  let dependentDialects = ["affine::AffineDialect", "arith::ArithDialect"];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 445-456
```tablegen
def AffineExpandIndexOps : Pass<"affine-expand-index-ops"> {
  let summary = "Lower affine operations operating on indices into more fundamental operations";
  let constructor = "mlir::affine::createAffineExpandIndexOpsPass()";
  let dependentDialects = ["vector::VectorDialect"];
}

def AffineExpandIndexOpsAsAffine : Pass<"affine-expand-index-ops-as-affine"> {
  let summary = "Lower affine operations operating on indices into affine.apply operations";
  let constructor = "mlir::affine::createAffineExpandIndexOpsAsAffinePass()";
  let dependentDialects = ["vector::VectorDialect", "ub::UBDialect"];
}

```
- **EN**: Introduces declarations for `AffineExpandIndexOps`, `AffineExpandIndexOpsAsAffine`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineExpandIndexOps`, `AffineExpandIndexOpsAsAffine` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 457-467
```tablegen
def AffineFoldMemRefAliasOps : Pass<"affine-fold-memref-alias-ops"> {
  let summary = "Fold memref alias ops into affine memory ops";
  let description = [{
    The pass folds memref.subview, memref.expand_shape, and memref.collapse_shape
    operations into affine memory operations (currently only `affine.load` and
    `affine.store`) . This is similar to the `fold-memref-alias-ops` pass in the
    `memref` dialect but adds handling specific to affine operations.
  }];
  let dependentDialects = ["memref::MemRefDialect"];
}

```
- **EN**: Introduces declarations for `AffineFoldMemRefAliasOps`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineFoldMemRefAliasOps` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 468-468
```tablegen
#endif // MLIR_DIALECT_AFFINE_PASSES
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
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
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
