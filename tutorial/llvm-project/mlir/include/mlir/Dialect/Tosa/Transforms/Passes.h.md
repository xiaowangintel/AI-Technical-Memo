# Passes.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tosa/Transforms/Passes.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR Passes component. The leading comments describe it as: This file declares the optimization passes for the TOSA Dialect in MLIR.
- **用途（CN）**: 声明 MLIR Passes 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
````cpp
//===-- Passes.h - TOSA optimization pass declarations ----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the optimization passes for the TOSA Dialect in MLIR.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TOSA_TRANSFORMS_PASSES_H
#define MLIR_DIALECT_TOSA_TRANSFORMS_PASSES_H

#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/Dialect/Tosa/IR/TosaOps.h"
#include "mlir/Pass/Pass.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 20-51
````cpp
namespace mlir {
class TypeConverter;
namespace tosa {

#define GEN_PASS_DECL
#include "mlir/Dialect/Tosa/Transforms/Passes.h.inc"

// Expose Rewrite Functions that decompose TOSA Ops into further TOSA Ops.
// The rewrites can be selectively added to a conversion pass.
void populateTosaDecomposeTransposeConv(MLIRContext *ctx,
                                        RewritePatternSet &patterns);
void populateTosaDecomposeDepthwise(MLIRContext *ctx,
                                    RewritePatternSet &patterns);
void populateTosaFoldConstantReciprocalPatterns(MLIRContext *ctx,
                                                RewritePatternSet &patterns);
void populateTosaFoldConstantTransposePatterns(MLIRContext *ctx,
                                               RewritePatternSet &patterns);
void populateTosaConstantReduction(MLIRContext *ctx,
                                   RewritePatternSet &patterns,
                                   bool aggressiveReduceConstant);

void populateTosaTypeConversion(TypeConverter &converter);

std::unique_ptr<Pass> createTosaTestQuantUtilAPIPass();
std::unique_ptr<Pass>
createTosaInputShapePass(std::vector<std::string> args = {});

#define GEN_PASS_REGISTRATION
#include "mlir/Dialect/Tosa/Transforms/Passes.h.inc"

} // namespace tosa
} // namespace mlir
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 53-53
````cpp
#endif // MLIR_DIALECT_TOSA_TRANSFORMS_PASSES_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: IR construction and mutation helpers
  **CN**: IR 构造与变更辅助接口
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/Dialect/Tensor/IR/Tensor.h
- mlir/Dialect/Tosa/IR/TosaOps.h
- mlir/Pass/Pass.h
- mlir/Dialect/Tosa/Transforms/Passes.h.inc
