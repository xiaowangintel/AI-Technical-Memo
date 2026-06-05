# SPIRVMatrixOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/IR/SPIRVMatrixOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR SPIRVMatrixOps component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This file contains matrix operations for the SPIR-V dialect.
- **用途（CN）**: 为 MLIR 的 SPIRVMatrixOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
````tablegen
//===-- SPIRVMatrixOps.td - MLIR SPIR-V Matrix Ops ---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains matrix operations for the SPIR-V dialect.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_IR_MATRIX_OPS
#define MLIR_DIALECT_SPIRV_IR_MATRIX_OPS
include "mlir/Interfaces/SideEffectInterfaces.td"

// -----
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 19-27
````tablegen
class SPIRV_MatrixDimMatch<string lhsArg, string lhsDim, string rhsArg, string rhsDim> : 
  PredOpTrait<
    lhsArg # " " # lhsDim # " count matches " # rhsArg # " " # rhsDim # " count",
    CPred<"::llvm::cast<::mlir::spirv::MatrixType>($" # lhsArg # ".getType())"
          ".getNum" # firstCharToUpper<lhsDim>.ret # "() == "
          "::llvm::cast<::mlir::spirv::MatrixType>($" # rhsArg # ".getType())"
          ".getNum" # firstCharToUpper<rhsDim>.ret # "()"
    >
  >;
````
- **EN**: This TableGen block defines `SPIRV_MatrixDimMatch` as a `class` record for `SPIRVMatrixOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_MatrixDimMatch` 定义为 `class` 记录，用于描述 `SPIRVMatrixOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 29-39
````tablegen
class SPIRV_MatrixVectorDimMatch<string matrixArg, string matrixDim, string vectorArg> :
  PredOpTrait<
    matrixArg # " " # matrixDim # " count matches " # vectorArg # " elements count",
    CPred<"::llvm::cast<::mlir::spirv::MatrixType>($" # matrixArg # ".getType())"
          ".getNum" # firstCharToUpper<matrixDim>.ret # "() == "
          "::llvm::cast<::mlir::VectorType>($" # vectorArg # ".getType())"
          ".getNumElements()"
    >
  >;

// -----
````
- **EN**: This TableGen block defines `SPIRV_MatrixVectorDimMatch` as a `class` record for `SPIRVMatrixOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_MatrixVectorDimMatch` 定义为 `class` 记录，用于描述 `SPIRVMatrixOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 41-92
````tablegen
def SPIRV_MatrixTimesMatrixOp : SPIRV_Op<"MatrixTimesMatrix", [
    Pure,
    AllElementTypesMatch<["leftmatrix", "rightmatrix", "result"]>,
    SPIRV_MatrixDimMatch<"leftmatrix", "columns", "rightmatrix", "rows">,
    SPIRV_MatrixDimMatch<"rightmatrix", "columns", "result", "columns">,
    SPIRV_MatrixDimMatch<"leftmatrix", "rows", "result", "rows">
  ]> {
  let summary = "Linear-algebraic multiply of LeftMatrix X RightMatrix.";

  let description = [{
    Result Type must be an OpTypeMatrix whose Column Type is a vector of
    floating-point type.

    LeftMatrix must be a matrix whose Column Type is the same as the Column
    Type in Result Type.

    RightMatrix must be a matrix with the same Component Type as the
    Component Type in Result Type. Its number of columns must equal the
    number of columns in Result Type. Its columns must have the same number
    of components as the number of columns in LeftMatrix.

    #### Example:

    ```mlir
    %0 = spirv.MatrixTimesMatrix %matrix_1, %matrix_2 :
        !spirv.matrix<4 x vector<3xf32>>, !spirv.matrix<3 x vector<4xf32>> ->
        !spirv.matrix<4 x vector<4xf32>>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_Matrix]>
  ];

  let arguments = (ins
    SPIRV_AnyMatrix:$leftmatrix,
    SPIRV_AnyMatrix:$rightmatrix
  );

  let results = (outs
    SPIRV_AnyMatrix:$result
  );

  let assemblyFormat = [{
    operands attr-dict `:` type($leftmatrix) `,` type($rightmatrix) `->` type($result)
  }];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_MatrixTimesMatrixOp` as a `def` record for `SPIRVMatrixOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_MatrixTimesMatrixOp` 定义为 `def` 记录，用于描述 `SPIRVMatrixOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 93-93
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 96-143
````tablegen
def SPIRV_OuterProductOp : SPIRV_Op<"OuterProduct", [
    Pure,
    AllElementTypesMatch<["vector1", "vector2", "result"]>,
    SPIRV_MatrixVectorDimMatch<"result", "rows", "vector1">,
    SPIRV_MatrixVectorDimMatch<"result", "columns", "vector2">
  ]> {
  let summary = "Linear-algebraic outer product of vector1 and vector2.";

  let description = [{
    Result Type must be a Matrix Type whose Column Type is a vector of
    floating-point type.

    Vector1 must have the same type as the Column Type in Result Type.

    Vector2 must be a vector with the same Component Type as the Component
    Type in Result Type. Its number of components must equal the number of
    columns in Result Type.

    #### Example:

    ```mlir
    %0 = spirv.OuterProduct %vector1, %vector2 :
        vector<3xf32>, vector<2xf32> -> !spirv.matrix<2 x vector<3xf32>>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_Matrix]>
  ];

  let arguments = (ins
    SPIRV_VectorOf<SPIRV_Float>:$vector1,
    SPIRV_VectorOf<SPIRV_Float>:$vector2
  );

  let results = (outs
    SPIRV_MatrixOf<SPIRV_Float>:$result
  );

  let assemblyFormat = [{
    operands attr-dict `:` type($vector1) `,` type($vector2) `->` type($result)
  }];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_OuterProductOp` as a `def` record for `SPIRVMatrixOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_OuterProductOp` 定义为 `def` 记录，用于描述 `SPIRVMatrixOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 144-144
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 147-192
````tablegen
def SPIRV_MatrixTimesScalarOp : SPIRV_Op<"MatrixTimesScalar", [
    Pure,
    AllTypesMatch<["matrix", "result"]>,
    AllElementTypesMatch<["matrix", "scalar"]>
  ]> {
  let summary = "Scale a floating-point matrix.";

  let description = [{
    Result Type must be a matrix type with a float component type.

    The type of Matrix must be the same as Result Type. Each component in
    each column in Matrix is multiplied by Scalar.

    Scalar must have the same type as the Component Type in Result Type.

    #### Example:

    ```mlir
    %0 = spirv.MatrixTimesScalar %matrix, %scalar :
    !spirv.matrix<3 x vector<3xf32>>, f32 -> !spirv.matrix<3 x vector<3xf32>>

    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_Matrix]>
  ];

  let arguments = (ins
    SPIRV_MatrixOrCoopMatrixOf<SPIRV_Float>:$matrix,
    SPIRV_Float:$scalar
  );

  let results = (outs
    SPIRV_MatrixOrCoopMatrixOf<SPIRV_Float>:$result
  );

  let assemblyFormat = [{
    operands attr-dict `:` type($matrix) `,` type($scalar)
  }];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_MatrixTimesScalarOp` as a `def` record for `SPIRVMatrixOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_MatrixTimesScalarOp` 定义为 `def` 记录，用于描述 `SPIRVMatrixOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 193-193
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 196-241
````tablegen
def SPIRV_MatrixTimesVectorOp : SPIRV_Op<"MatrixTimesVector", [
    Pure,
    AllElementTypesMatch<["matrix", "result"]>,
    AllElementTypesMatch<["vector", "result"]>,
    SPIRV_MatrixVectorDimMatch<"matrix", "columns", "vector">,
    SPIRV_MatrixVectorDimMatch<"matrix", "rows", "result">
  ]> {
  let summary = "Linear-algebraic Matrix X Vector.";

  let description = [{
    Result Type must be a vector of floating-point type.

    Matrix must be an OpTypeMatrix whose Column Type is Result Type.

    Vector must be a vector with the same Component Type as the Component Type in Result Type. Its number of components must equal the number of columns in Matrix.

    #### Example:

    ```mlir
    %0 = spirv.MatrixTimesVector %matrix, %vector : 
        !spirv.matrix<3 x vector<2xf32>>, vector<3xf32> -> vector<2xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_Matrix]>
  ];

  let arguments = (ins
    SPIRV_MatrixOf<SPIRV_Float>:$matrix,
    SPIRV_VectorOf<SPIRV_Float>:$vector
  );

  let results = (outs
    SPIRV_VectorOf<SPIRV_Float>:$result
  );

  let assemblyFormat = [{
    operands attr-dict `:` type($matrix) `,` type($vector) `->` type($result)
  }];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_MatrixTimesVectorOp` as a `def` record for `SPIRVMatrixOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_MatrixTimesVectorOp` 定义为 `def` 记录，用于描述 `SPIRVMatrixOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 242-242
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 245-291
````tablegen
def SPIRV_TransposeOp : SPIRV_Op<"Transpose", [
    Pure,
    AllElementTypesMatch<["matrix", "result"]>,
    SPIRV_MatrixDimMatch<"matrix", "rows", "result", "columns">,
    SPIRV_MatrixDimMatch<"matrix", "columns", "result", "rows">
  ]> {
  let summary = "Transpose a matrix.";

  let description = [{
    Result Type must be an OpTypeMatrix.

    Matrix must be an object of type OpTypeMatrix. The number of columns and
    the column size of Matrix must be the reverse of those in Result Type.
    The types of the scalar components in Matrix and Result Type must be the
    same.

    Matrix must have of type of OpTypeMatrix.

    #### Example:

    ```mlir
    %0 = spirv.Transpose %matrix: !spirv.matrix<2 x vector<3xf32>> ->
    !spirv.matrix<3 x vector<2xf32>>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_Matrix]>
  ];

  let arguments = (ins
    SPIRV_AnyMatrix:$matrix
  );

  let results = (outs
    SPIRV_AnyMatrix:$result
  );

  let assemblyFormat = [{
    operands attr-dict `:` type($matrix) `->` type($result)
  }];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_TransposeOp` as a `def` record for `SPIRVMatrixOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TransposeOp` 定义为 `def` 记录，用于描述 `SPIRVMatrixOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 292-292
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 295-345
````tablegen
def SPIRV_VectorTimesMatrixOp : SPIRV_Op<"VectorTimesMatrix", [
    Pure,
    AllElementTypesMatch<["vector", "result"]>,
    AllElementTypesMatch<["matrix", "result"]>,
    SPIRV_MatrixVectorDimMatch<"matrix", "rows", "vector">,
    SPIRV_MatrixVectorDimMatch<"matrix", "columns", "result">
  ]> {
  let summary = "Linear-algebraic Vector X Matrix.";

  let description = [{
    Result Type must be a vector of floating-point type.

    Vector must be a vector with the same Component Type as the Component
    Type in Result Type. Its number of components must equal the number of
    components in each column in Matrix.

    Matrix must be a matrix with the same Component Type as the Component
    Type in Result Type. Its number of columns must equal the number of
    components in Result Type.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %result = spirv.VectorTimesMatrix %vector, %matrix : vector<4xf32>, !spirv.matrix<4 x vector<4xf32>> -> vector<4xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_Matrix]>
  ];

  let arguments = (ins
    SPIRV_VectorOf<SPIRV_Float>:$vector,
    SPIRV_MatrixOf<SPIRV_Float>:$matrix
  );

  let results = (outs
    SPIRV_VectorOf<SPIRV_Float>:$result
  );

  let assemblyFormat = [{
    operands attr-dict `:` type($vector) `,` type($matrix) `->` type($result)
  }];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_VectorTimesMatrixOp` as a `def` record for `SPIRVMatrixOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_VectorTimesMatrixOp` 定义为 `def` 记录，用于描述 `SPIRVMatrixOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 346-348
````tablegen
// -----

#endif // MLIR_DIALECT_SPIRV_IR_MATRIX_OPS
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Custom assembly syntax specification
  **CN**: 自定义汇编语法规格
- **EN**: Operand/result schema definition
  **CN**: 操作数/结果模式定义
- **EN**: Trait and interface composition
  **CN**: trait 与接口组合
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/Interfaces/SideEffectInterfaces.td
- SPIRV_MatrixTimesMatrixOp builds on SPIRV_Op<"MatrixTimesMatrix", [
- SPIRV_OuterProductOp builds on SPIRV_Op<"OuterProduct", [
- SPIRV_MatrixTimesScalarOp builds on SPIRV_Op<"MatrixTimesScalar", [
- SPIRV_MatrixTimesVectorOp builds on SPIRV_Op<"MatrixTimesVector", [
- SPIRV_TransposeOp builds on SPIRV_Op<"Transpose", [
- SPIRV_VectorTimesMatrixOp builds on SPIRV_Op<"VectorTimesMatrix", [
