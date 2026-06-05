# Passes.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/XeGPU/Transforms/Passes.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR Passes component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 Passes 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
````tablegen
//===-- Passes.td - XeGPU transformation definition file ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_XEGPU_TRANSFORMS_PASSES_TD
#define MLIR_DIALECT_XEGPU_TRANSFORMS_PASSES_TD

include "mlir/Pass/PassBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 14-21
````tablegen
def XeGPUSubgroupDistribute : Pass<"xegpu-subgroup-distribute"> {
  let summary = "Distribute XeGPU ops to work items";
  let description = [{
    The pass distributes subgroup level (SIMD) XeGPU ops to work items.
  }];
  let dependentDialects = ["memref::MemRefDialect", "xegpu::XeGPUDialect",
                           "vector::VectorDialect", "index::IndexDialect"];
}
````
- **EN**: This TableGen block defines `XeGPUSubgroupDistribute` as a `def` record for `Passes`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `XeGPUSubgroupDistribute` 定义为 `def` 记录，用于描述 `Passes` 相关的声明式信息。 其中涉及 语义文档。

### Lines 23-64
````tablegen
def XeGPUPropagateLayout : Pass<"xegpu-propagate-layout"> {
  let summary = "Propagate and assign XeGPU layout information";
  let description = [{
    This pass propagates the XeGPU layout information accross ops. Starting
    from a set of anchor operations (e.g. `dpas`, `store_nd`), this will
    propagate the layouts required for their operands to the producers. With
    this propagated layout information, pass will then update op result type
    with the layout information.

    `layout-kind` option values:
    - `inst`
      Propagate the `inst_data` field of the layout attribute. The default is chosen to
       maximize instruction-level granularity so that the user shape can be processed
       with the fewest instructions. For N-D operations, this granularity depends on
       W (width) and H (height) of the instruction shape.
       The B (block) dimension (or array length) is not included in the default
       configuration and must be enabled via a separate optimization pass.

    - `lane`
      Propagate the `lane_layout` and `lane_data` fields of the layout attribute.
      Default values are selected to align with hardware.

    - `subgroup`
      Propagate the `sg_layout` and `sg_data` fields of the layout attribute.
      Default values are selected to align with hardware.
  }];
  let dependentDialects = ["memref::MemRefDialect", "xegpu::XeGPUDialect",
                           "vector::VectorDialect"];
  let options = [Option<
    "printOnly", "print-analysis-only", "bool",
    /*default=*/"false",
    "Print the result of layout propagation analysis and exit.">,
    Option<
    "layoutKind", "layout-kind", "std::string",
    /*default=*/"\"lane\"",
    "Propagate `subgroup` / `inst` / `lane` level of xegpu layouts.">,
    Option<
    "indexBitWidth", "index-bitwidth", "unsigned",
    /*default=*/"64",
    "Vectors of `index` type should also be distributable, inst-data and lower levels need to know the index size.">
  ];
}
````
- **EN**: This TableGen block defines `XeGPUPropagateLayout` as a `def` record for `Passes`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `XeGPUPropagateLayout` 定义为 `def` 记录，用于描述 `Passes` 相关的声明式信息。 其中涉及 语义文档。

### Lines 66-76
````tablegen
def XeGPUWgToSgDistribute : Pass<"xegpu-wg-to-sg-distribute"> {
  let summary = "Transform WorkGroup level XeGPU code to SubGroup level";
  let description = [{
    This transform pass distributes the workgroup level computation to
    multiple subgroups based on the sg_layout and sg_data attributes.
  }];

  let dependentDialects = ["memref::MemRefDialect", "xegpu::XeGPUDialect",
                           "vector::VectorDialect", "arith::ArithDialect",
                           "gpu::GPUDialect", "index::IndexDialect"];
}
````
- **EN**: This TableGen block defines `XeGPUWgToSgDistribute` as a `def` record for `Passes`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `XeGPUWgToSgDistribute` 定义为 `def` 记录，用于描述 `Passes` 相关的声明式信息。 其中涉及 语义文档。

### Lines 78-89
````tablegen
def XeGPUBlocking: Pass<"xegpu-blocking"> {
  let summary = "Block XeGPU ops into smaller size.";
  let description = [{
    This pass partitions operations that process large shapes into multiple
    operations on smaller shapes, as specified by the inst_data in the layout
    attribute. This enables each resulting operation to be efficiently mapped
    to a hardware instruction.
  }];
  let dependentDialects = [
      "memref::MemRefDialect", "xegpu::XeGPUDialect", "vector::VectorDialect",
      "index::IndexDialect"];
}
````
- **EN**: This TableGen block defines `XeGPUBlocking` as a `def` record for `Passes`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `XeGPUBlocking` 定义为 `def` 记录，用于描述 `Passes` 相关的声明式信息。 其中涉及 语义文档。

### Lines 91-98
````tablegen
def XeGPUVectorLinearize : Pass<"xegpu-vector-linearize"> {
  let summary = "Linearize n-D vectors to 1-D vectors";
  let description = [{
    This pass linearizes n-D vectors to 1-D vectors for lowering to XeVM.
  }];
  let dependentDialects = ["arith::ArithDialect", "memref::MemRefDialect",
                           "scf::SCFDialect", "ub::UBDialect", "vector::VectorDialect"];
}
````
- **EN**: This TableGen block defines `XeGPUVectorLinearize` as a `def` record for `Passes`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `XeGPUVectorLinearize` 定义为 `def` 记录，用于描述 `Passes` 相关的声明式信息。 其中涉及 语义文档。

### Lines 100-110
````tablegen
def XeGPUPeepHoleOptimizer : Pass<"xegpu-optimize-peephole"> {
  let summary = "Optimize XeGPU block load operations";
  let description = [{
    This pass rewrites XeGPU loadNd operations into more optimal forms
    to improve performance. This includes,
    - Rewriting transpose B loads into more optimal forms to use HW block
      transpose instructions for better performance.
  }];
  let dependentDialects = ["memref::MemRefDialect", "xegpu::XeGPUDialect",
                           "vector::VectorDialect"];
}
````
- **EN**: This TableGen block defines `XeGPUPeepHoleOptimizer` as a `def` record for `Passes`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `XeGPUPeepHoleOptimizer` 定义为 `def` 记录，用于描述 `Passes` 相关的声明式信息。 其中涉及 语义文档。

### Lines 112-119
````tablegen
def XeGPUSgToWiDistributeExperimental : Pass<"xegpu-sg-to-wi-distribute-experimental"> {
  let summary = "Distribute XeGPU ops to work items";
  let description = [{
    The pass distributes subgroup level XeGPU ops to work item level XeGPU ops.
  }];
  let dependentDialects = ["memref::MemRefDialect", "xegpu::XeGPUDialect",
                           "vector::VectorDialect", "index::IndexDialect"];
}
````
- **EN**: This TableGen block defines `XeGPUSgToWiDistributeExperimental` as a `def` record for `Passes`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `XeGPUSgToWiDistributeExperimental` 定义为 `def` 记录，用于描述 `Passes` 相关的声明式信息。 其中涉及 语义文档。

### Lines 120-120
````tablegen
#endif // MLIR_DIALECT_XEGPU_TRANSFORMS_PASSES_TD
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/Pass/PassBase.td
- XeGPUSubgroupDistribute builds on Pass<"xegpu-subgroup-distribute">
- XeGPUPropagateLayout builds on Pass<"xegpu-propagate-layout">
- XeGPUWgToSgDistribute builds on Pass<"xegpu-wg-to-sg-distribute">
- XeGPUBlocking builds on Pass<"xegpu-blocking">
- XeGPUVectorLinearize builds on Pass<"xegpu-vector-linearize">
- XeGPUPeepHoleOptimizer builds on Pass<"xegpu-optimize-peephole">
- XeGPUSgToWiDistributeExperimental builds on Pass<"xegpu-sg-to-wi-distribute-experimental">
