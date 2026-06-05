# XeGPUDialect.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/XeGPU/IR/XeGPUDialect.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR XeGPUDialect component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 XeGPUDialect 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
````tablegen
//===- XeGPUDialect.td - XeGPU dialect definition -----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_XEGPU_IR_XEGPUDIALECT_TD
#define MLIR_DIALECT_XEGPU_IR_XEGPUDIALECT_TD

include "mlir/IR/OpBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 14-58
````tablegen
def XeGPU_Dialect : Dialect {
    let name = "xegpu";
    let cppNamespace = "::mlir::xegpu";
    let summary = "The XeGPU dialect that models Intel GPU's ISA";
    let description = [{
      The XeGPU dialect closely models a subset of the Xe GPU's ISA, providing an
      abstraction to support high-performance GEMM code generation. It serves as a
      bridge dialect in the MLIR gradual lowering process, working with MLIR memref
      and vector types, and complements the Arith, Math, Vector, and Memref dialects.
      XeGPU operations are introduced for special Xe instructions not modeled by the
      LLVM/SPIR-V dialect, such as DPAS and 2D block load and store.

      It supports a tile-based programming model, decomposing the GEMM kernel into
      large predefined tile sizes at the subgroup and workgroup levels. XeGPU allows
      the high-level GEMM algorithm to be easily expressed. Underneath, it uses
      target-specific recipes and hardware features to achieve optimal performance
      on specific hardware. By decomposing GEMM at submatrix granularity and mapping it
      to registers, it naturally supports optimizations like fusing with neighboring
      operations.
    }];

    let dependentDialects = ["arith::ArithDialect"];

    let useDefaultTypePrinterParser = true;
    let useDefaultAttributePrinterParser = true;

    let extraClassDeclaration = [{
      /// Checks if the given memref type represents shared local memory (SLM).
      /// Returns true if the memory space is address space 3, MemorySpace::SLM,
      /// xevm::AddrSpace::SHARED, or a GPU workgroup memory address space.
      static bool isSharedMemory(const MemRefType &memrefTy);

      /// drops/slices the shape in the specified dims, and return the rest. e.g.,
      /// for shape = [32, 64, 8], dims = [0, 2], it will return [64]
      template<typename T, typename U>
      static llvm::SmallVector<T> slice(llvm::ArrayRef<T> shape, llvm::ArrayRef<U> dims) {
        llvm::SmallVector<T> result;
        for (auto [i, v]: llvm::enumerate(shape)) {
          if (!llvm::is_contained(dims, i))
            result.push_back(v);
        }
        return result;
      }
    }];
}
````
- **EN**: This TableGen block defines `XeGPU_Dialect` as a `def` record for `XeGPUDialect`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `XeGPU_Dialect` 定义为 `def` 记录，用于描述 `XeGPUDialect` 相关的声明式信息。 其中涉及 语义文档。

### Lines 59-59
````tablegen
#endif // MLIR_DIALECT_XEGPU_IR_XEGPUDIALECT_TD
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/IR/OpBase.td
- XeGPU_Dialect builds on Dialect
