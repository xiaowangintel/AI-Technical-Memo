# SPIRVCanonicalization.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SPIRV/IR/SPIRVCanonicalization.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines SPIR-V canonicalization patterns.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SPIRV/IR`，围绕 SPIRV 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//==- SPIRVCanonicalization.td - Canonicalization Patterns ---*- tablegen -*==//

// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-11
```tablegen
//
// This file defines SPIR-V canonicalization patterns.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 12-15
```tablegen

include "mlir/IR/PatternBase.td"
include "mlir/Dialect/SPIRV/IR/SPIRVOps.td"

```
- **EN**: Imports declarative TableGen definitions needed here, including `mlir/IR/PatternBase.td`, `mlir/Dialect/SPIRV/IR/SPIRVOps.td`.
- **CN**: 引入此处所需的声明式 TableGen 定义，其中包括 `mlir/IR/PatternBase.td`, `mlir/Dialect/SPIRV/IR/SPIRVOps.td`。

### Lines 16-19
```tablegen
//===----------------------------------------------------------------------===//
// spirv.LogicalNot
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 20-23
```tablegen
def ConvertLogicalNotOfIEqual : Pat<
    (SPIRV_LogicalNotOp (SPIRV_IEqualOp $lhs, $rhs)),
    (SPIRV_INotEqualOp $lhs, $rhs)>;

```
- **EN**: Introduces declarations for `ConvertLogicalNotOfIEqual`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertLogicalNotOfIEqual` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 24-27
```tablegen
def ConvertLogicalNotOfINotEqual : Pat<
    (SPIRV_LogicalNotOp (SPIRV_INotEqualOp $lhs, $rhs)),
    (SPIRV_IEqualOp $lhs, $rhs)>;

```
- **EN**: Introduces declarations for `ConvertLogicalNotOfINotEqual`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertLogicalNotOfINotEqual` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 28-31
```tablegen
def ConvertLogicalNotOfLogicalEqual : Pat<
    (SPIRV_LogicalNotOp (SPIRV_LogicalEqualOp $lhs, $rhs)),
    (SPIRV_LogicalNotEqualOp $lhs, $rhs)>;

```
- **EN**: Introduces declarations for `ConvertLogicalNotOfLogicalEqual`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertLogicalNotOfLogicalEqual` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 32-35
```tablegen
def ConvertLogicalNotOfLogicalNotEqual : Pat<
    (SPIRV_LogicalNotOp (SPIRV_LogicalNotEqualOp $lhs, $rhs)),
    (SPIRV_LogicalEqualOp $lhs, $rhs)>;

```
- **EN**: Introduces declarations for `ConvertLogicalNotOfLogicalNotEqual`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertLogicalNotOfLogicalNotEqual` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 36-39
```tablegen
//===----------------------------------------------------------------------===//
// spirv.Select -> spirv.GL.*Clamp
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 40-47
```tablegen
def ValuesAreEqual : Constraint<CPred<"$0 == $1">>;

foreach CmpClampPair = [
    [SPIRV_FOrdLessThanOp, SPIRV_GLFClampOp],
    [SPIRV_FOrdLessThanEqualOp, SPIRV_GLFClampOp],
    [SPIRV_SLessThanOp, SPIRV_GLSClampOp],
    [SPIRV_SLessThanEqualOp, SPIRV_GLSClampOp],
    [SPIRV_ULessThanOp, SPIRV_GLUClampOp],
```
- **EN**: Introduces declarations for `ValuesAreEqual`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ValuesAreEqual` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 48-51
```tablegen
    [SPIRV_ULessThanEqualOp, SPIRV_GLUClampOp]] in {

// Detect: $min < $input, $input < $max
def ConvertComparisonIntoClamp1_#CmpClampPair[0] : Pat<
```
- **EN**: Introduces declarations for `ConvertComparisonIntoClamp1_`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertComparisonIntoClamp1_` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 52-59
```tablegen
    (SPIRV_SelectOp
        (CmpClampPair[0]
            (SPIRV_SelectOp:$middle0
                (CmpClampPair[0] $min, $input),
                $input,
                $min
            ),
            $max
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 60-65
```tablegen
        ),
        $middle1,
        $max),
    (CmpClampPair[1] $input, $min, $max),
    [(ValuesAreEqual $middle0, $middle1)]>;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 66-73
```tablegen
// Detect: $input < $min, $max < $input
def ConvertComparisonIntoClamp2_#CmpClampPair[0] : Pat<
    (SPIRV_SelectOp
        (CmpClampPair[0] $max, $input),
        $max,
        (SPIRV_SelectOp
            (CmpClampPair[0] $input, $min),
            $min,
```
- **EN**: Introduces declarations for `ConvertComparisonIntoClamp2_`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertComparisonIntoClamp2_` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 74-77
```tablegen
            $input
        )),
    (CmpClampPair[1] $input, $min, $max)>;
}
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Defines or implements dialect operations, attributes, types, traits, and registration hooks.
  - **CN**: 定义或实现方言操作、属性、类型、trait 以及注册钩子。
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束或规范化规则。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/IR/PatternBase.td`, `mlir/Dialect/SPIRV/IR/SPIRVOps.td`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (1)
