# SPIRVLogicalOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/IR/SPIRVLogicalOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR SPIRVLogicalOps component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This file contains arithmetic ops for the SPIR-V dialect. It corresponds.
- **用途（CN）**: 为 MLIR 的 SPIRVLogicalOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
````tablegen
//===-- SPIRVLogicalOps.td - MLIR SPIR-V Logical Ops -------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains arithmetic ops for the SPIR-V dialect. It corresponds
// to "3.32.15. Relational and Logical Instructions" of the SPIR-V spec.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_IR_LOGICAL_OPS
#define MLIR_DIALECT_SPIRV_IR_LOGICAL_OPS

include "mlir/Dialect/SPIRV/IR/SPIRVBase.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 20-33
````tablegen
class SPIRV_LogicalBinaryOp<string mnemonic, Type operandsType,
                            list<Trait> traits = []> :
      // Result type is SPIRV_Bool.
      SPIRV_BinaryOp<mnemonic, SPIRV_Bool, operandsType,
                   !listconcat(traits, [
                     Pure, SameTypeOperands,
                     SameOperandsAndResultShape,
                     TypesMatchWith<"type of result to correspond to the `i1` "
                                    "equivalent of the operand",
                                    "operand1", "result",
                                    "getMatchingBoolType($_self)"
                     >])> {
  let assemblyFormat = "$operand1 `,` $operand2 `:` type($operand1) attr-dict";
}
````
- **EN**: This TableGen block defines `SPIRV_LogicalBinaryOp` as a `class` record for `SPIRVLogicalOps`. It covers assembly syntax, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_LogicalBinaryOp` 定义为 `class` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 汇编语法, trait/接口组合。

### Lines 35-47
````tablegen
class SPIRV_LogicalUnaryOp<string mnemonic, Type operandType,
                         list<Trait> traits = []> :
      // Result type is SPIRV_Bool.
      SPIRV_UnaryOp<mnemonic, SPIRV_Bool, operandType,
                  !listconcat(traits, [
                     Pure, SameTypeOperands, SameOperandsAndResultShape,
                     TypesMatchWith<"type of result to correspond to the `i1` "
                                    "equivalent of the operand",
                                    "operand", "result",
                                    "getMatchingBoolType($_self)"
                     >])> {
  let assemblyFormat = "$operand `:` type($operand) attr-dict";
}
````
- **EN**: This TableGen block defines `SPIRV_LogicalUnaryOp` as a `class` record for `SPIRVLogicalOps`. It covers assembly syntax, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_LogicalUnaryOp` 定义为 `class` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 汇编语法, trait/接口组合。

### Lines 48-48
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 51-79
````tablegen
def SPIRV_AnyOp : SPIRV_Op<"Any", [Pure]> {
  let summary = "Result is true if any component of Vector is true.";

  let description = [{
    Result Type must be a Boolean type scalar.

    Vector must be a vector of Boolean type.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %result = spirv.Any %vector : vector<4xi1>
    ```
  }];

  let arguments = (ins
    SPIRV_VectorOf<SPIRV_Bool>:$vector
  );

  let results = (outs
    SPIRV_Bool:$result
  );

  let assemblyFormat = "$vector attr-dict `:` type($vector)";

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_AnyOp` as a `def` record for `SPIRVLogicalOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_AnyOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 80-80
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 83-111
````tablegen
def SPIRV_AllOp : SPIRV_Op<"All", [Pure]> {
  let summary = "Result is true if all components of Vector are true.";

  let description = [{
    Result Type must be a Boolean type scalar.

    Vector must be a vector of Boolean type.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %result = spirv.All %vector : vector<4xi1>
    ```
  }];

  let arguments = (ins
    SPIRV_VectorOf<SPIRV_Bool>:$vector
  );

  let results = (outs
    SPIRV_Bool:$result
  );

  let assemblyFormat = "$vector attr-dict `:` type($vector)";

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_AllOp` as a `def` record for `SPIRVLogicalOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_AllOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 112-112
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 115-134
````tablegen
def SPIRV_FOrdEqualOp : SPIRV_LogicalBinaryOp<"FOrdEqual", SPIRV_Float, [Commutative]> {
  let summary = "Floating-point comparison for being ordered and equal.";

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FOrdEqual %0, %1 : f32
    %5 = spirv.FOrdEqual %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FOrdEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FOrdEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 135-135
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 138-160
````tablegen
def SPIRV_FOrdGreaterThanOp : SPIRV_LogicalBinaryOp<"FOrdGreaterThan", SPIRV_Float, []> {
  let summary = [{
    Floating-point comparison if operands are ordered and Operand 1 is
    greater than  Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FOrdGreaterThan %0, %1 : f32
    %5 = spirv.FOrdGreaterThan %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FOrdGreaterThanOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FOrdGreaterThanOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 161-161
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 164-186
````tablegen
def SPIRV_FOrdGreaterThanEqualOp : SPIRV_LogicalBinaryOp<"FOrdGreaterThanEqual", SPIRV_Float, []> {
  let summary = [{
    Floating-point comparison if operands are ordered and Operand 1 is
    greater than or equal to Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FOrdGreaterThanEqual %0, %1 : f32
    %5 = spirv.FOrdGreaterThanEqual %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FOrdGreaterThanEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FOrdGreaterThanEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 187-187
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 190-212
````tablegen
def SPIRV_FOrdLessThanOp : SPIRV_LogicalBinaryOp<"FOrdLessThan", SPIRV_Float, []> {
  let summary = [{
    Floating-point comparison if operands are ordered and Operand 1 is less
    than Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FOrdLessThan %0, %1 : f32
    %5 = spirv.FOrdLessThan %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FOrdLessThanOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FOrdLessThanOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 213-213
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 216-238
````tablegen
def SPIRV_FOrdLessThanEqualOp : SPIRV_LogicalBinaryOp<"FOrdLessThanEqual", SPIRV_Float, []> {
  let summary = [{
    Floating-point comparison if operands are ordered and Operand 1 is less
    than or equal to Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FOrdLessThanEqual %0, %1 : f32
    %5 = spirv.FOrdLessThanEqual %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FOrdLessThanEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FOrdLessThanEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 239-239
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 242-261
````tablegen
def SPIRV_FOrdNotEqualOp : SPIRV_LogicalBinaryOp<"FOrdNotEqual", SPIRV_Float, [Commutative]> {
  let summary = "Floating-point comparison for being ordered and not equal.";

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FOrdNotEqual %0, %1 : f32
    %5 = spirv.FOrdNotEqual %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FOrdNotEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FOrdNotEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 262-262
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 265-284
````tablegen
def SPIRV_FUnordEqualOp : SPIRV_LogicalBinaryOp<"FUnordEqual", SPIRV_Float, [Commutative]> {
  let summary = "Floating-point comparison for being unordered or equal.";

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FUnordEqual %0, %1 : f32
    %5 = spirv.FUnordEqual %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FUnordEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FUnordEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 285-285
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 288-310
````tablegen
def SPIRV_FUnordGreaterThanOp : SPIRV_LogicalBinaryOp<"FUnordGreaterThan", SPIRV_Float, []> {
  let summary = [{
    Floating-point comparison if operands are unordered or Operand 1 is
    greater than  Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FUnordGreaterThan %0, %1 : f32
    %5 = spirv.FUnordGreaterThan %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FUnordGreaterThanOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FUnordGreaterThanOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 311-311
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 314-336
````tablegen
def SPIRV_FUnordGreaterThanEqualOp : SPIRV_LogicalBinaryOp<"FUnordGreaterThanEqual", SPIRV_Float, []> {
  let summary = [{
    Floating-point comparison if operands are unordered or Operand 1 is
    greater than or equal to Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FUnordGreaterThanEqual %0, %1 : f32
    %5 = spirv.FUnordGreaterThanEqual %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FUnordGreaterThanEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FUnordGreaterThanEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 337-337
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 340-362
````tablegen
def SPIRV_FUnordLessThanOp : SPIRV_LogicalBinaryOp<"FUnordLessThan", SPIRV_Float, []> {
  let summary = [{
    Floating-point comparison if operands are unordered or Operand 1 is less
    than Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FUnordLessThan %0, %1 : f32
    %5 = spirv.FUnordLessThan %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FUnordLessThanOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FUnordLessThanOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 363-363
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 366-388
````tablegen
def SPIRV_FUnordLessThanEqualOp : SPIRV_LogicalBinaryOp<"FUnordLessThanEqual", SPIRV_Float, []> {
  let summary = [{
    Floating-point comparison if operands are unordered or Operand 1 is less
    than or equal to Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FUnordLessThanEqual %0, %1 : f32
    %5 = spirv.FUnordLessThanEqual %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FUnordLessThanEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FUnordLessThanEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 389-389
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 392-411
````tablegen
def SPIRV_FUnordNotEqualOp : SPIRV_LogicalBinaryOp<"FUnordNotEqual", SPIRV_Float, [Commutative]> {
  let summary = "Floating-point comparison for being unordered or not equal.";

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    floating-point type.  They must have the same type, and they must have
    the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.FUnordNotEqual %0, %1 : f32
    %5 = spirv.FUnordNotEqual %2, %3 : vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_FUnordNotEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_FUnordNotEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 412-412
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 415-438
````tablegen
def SPIRV_IEqualOp : SPIRV_LogicalBinaryOp<"IEqual",
                                       SPIRV_Integer,
                                       [Commutative, UsableInSpecConstantOp]> {
  let summary = "Integer comparison for equality.";

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.IEqual %0, %1 : i32
    %5 = spirv.IEqual %2, %3 : vector<4xi32>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_IEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_IEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 439-439
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 442-466
````tablegen
def SPIRV_INotEqualOp : SPIRV_LogicalBinaryOp<"INotEqual",
                                          SPIRV_Integer,
                                          [Commutative, UsableInSpecConstantOp]> {
  let summary = "Integer comparison for inequality.";

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.INotEqual %0, %1 : i32
    %5 = spirv.INotEqual %2, %3 : vector<4xi32>

    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_INotEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_INotEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 467-467
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 470-488
````tablegen
def SPIRV_IsFiniteOp : SPIRV_LogicalUnaryOp<"IsFinite", SPIRV_Float, []> {
  let summary = "Result is true if x is an IEEE Finite, otherwise result is false";

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    x must be a scalar or vector of floating-point type.  It must have the
    same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %2 = spirv.IsFinite %0: f32
    %3 = spirv.IsFinite %1: vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_IsFiniteOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_IsFiniteOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 489-489
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 492-510
````tablegen
def SPIRV_IsInfOp : SPIRV_LogicalUnaryOp<"IsInf", SPIRV_Float, []> {
  let summary = "Result is true if x is an IEEE Inf, otherwise result is false";

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    x must be a scalar or vector of floating-point type.  It must have the
    same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %2 = spirv.IsInf %0: f32
    %3 = spirv.IsInf %1: vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_IsInfOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_IsInfOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 511-511
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 514-534
````tablegen
def SPIRV_IsNanOp : SPIRV_LogicalUnaryOp<"IsNan", SPIRV_Float, []> {
  let summary = [{
    Result is true if x is an IEEE NaN, otherwise result is false.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    x must be a scalar or vector of floating-point type.  It must have the
    same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %2 = spirv.IsNan %0: f32
    %3 = spirv.IsNan %1: vector<4xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_IsNanOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_IsNanOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 535-535
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 538-565
````tablegen
def SPIRV_LogicalAndOp : SPIRV_LogicalBinaryOp<"LogicalAnd",
                                           SPIRV_Bool,
                                           [Commutative,
                                            UsableInSpecConstantOp]> {
  let summary = [{
    Result is true if both Operand 1 and Operand 2 are true. Result is false
    if either Operand 1 or Operand 2 are false.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 must be the same as Result Type.

    The type of Operand 2 must be the same as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %2 = spirv.LogicalAnd %0, %1 : i1
    %2 = spirv.LogicalAnd %0, %1 : vector<4xi1>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_LogicalAndOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_LogicalAndOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 566-566
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 569-596
````tablegen
def SPIRV_LogicalEqualOp : SPIRV_LogicalBinaryOp<"LogicalEqual",
                                             SPIRV_Bool,
                                             [Commutative,
                                              UsableInSpecConstantOp]> {
  let summary = [{
    Result is true if Operand 1 and Operand 2 have the same value. Result is
    false if Operand 1 and Operand 2 have different values.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 must be the same as Result Type.

    The type of Operand 2 must be the same as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %2 = spirv.LogicalEqual %0, %1 : i1
    %2 = spirv.LogicalEqual %0, %1 : vector<4xi1>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_LogicalEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_LogicalEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 597-597
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 600-624
````tablegen
def SPIRV_LogicalNotOp : SPIRV_LogicalUnaryOp<"LogicalNot",
                                          SPIRV_Bool,
                                          [UsableInSpecConstantOp]> {
  let summary = [{
    Result is true if Operand is false.  Result is false if Operand is true.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand must be the same as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %2 = spirv.LogicalNot %0 : i1
    %2 = spirv.LogicalNot %0 : vector<4xi1>
    ```
  }];

  let hasCanonicalizer = 1;
  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_LogicalNotOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_LogicalNotOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 625-625
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 628-655
````tablegen
def SPIRV_LogicalNotEqualOp : SPIRV_LogicalBinaryOp<"LogicalNotEqual",
                                                SPIRV_Bool,
                                                [Commutative,
                                                 UsableInSpecConstantOp]> {
  let summary = [{
    Result is true if Operand 1 and Operand 2 have different values. Result
    is false if Operand 1 and Operand 2 have the same value.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 must be the same as Result Type.

    The type of Operand 2 must be the same as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %2 = spirv.LogicalNotEqual %0, %1 : i1
    %2 = spirv.LogicalNotEqual %0, %1 : vector<4xi1>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_LogicalNotEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_LogicalNotEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 656-656
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 659-686
````tablegen
def SPIRV_LogicalOrOp : SPIRV_LogicalBinaryOp<"LogicalOr",
                                          SPIRV_Bool,
                                          [Commutative,
                                           UsableInSpecConstantOp]> {
  let summary = [{
    Result is true if either Operand 1 or Operand 2 is true. Result is false
    if both Operand 1 and Operand 2 are false.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 must be the same as Result Type.

    The type of Operand 2 must be the same as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %2 = spirv.LogicalOr %0, %1 : i1
    %2 = spirv.LogicalOr %0, %1 : vector<4xi1>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_LogicalOrOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_LogicalOrOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 687-687
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 690-720
````tablegen
def SPIRV_OrderedOp : SPIRV_LogicalBinaryOp<"Ordered", SPIRV_Float, [Commutative]> {
  let summary = [{
    Result is true if both x == x and y == y are true, where IEEE comparison
    is used, otherwise result is false.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    x must be a scalar or vector of floating-point type.  It must have the
    same number of components as Result Type.

    y must have the same type as x.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.Ordered %0, %1 : f32
    %5 = spirv.Ordered %2, %3 : vector<4xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_Kernel]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_OrderedOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_OrderedOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 721-721
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 724-750
````tablegen
def SPIRV_SGreaterThanOp : SPIRV_LogicalBinaryOp<"SGreaterThan",
                                             SPIRV_Integer,
                                             [UsableInSpecConstantOp, SignedOp]> {
  let summary = [{
    Signed-integer comparison if Operand 1 is greater than  Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.SGreaterThan %0, %1 : i32
    %5 = spirv.SGreaterThan %2, %3 : vector<4xi32>

    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_SGreaterThanOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_SGreaterThanOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 751-751
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 754-781
````tablegen
def SPIRV_SGreaterThanEqualOp : SPIRV_LogicalBinaryOp<"SGreaterThanEqual",
                                                  SPIRV_Integer,
                                                  [UsableInSpecConstantOp,
                                                   SignedOp]> {
  let summary = [{
    Signed-integer comparison if Operand 1 is greater than or equal to
    Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.SGreaterThanEqual %0, %1 : i32
    %5 = spirv.SGreaterThanEqual %2, %3 : vector<4xi32>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_SGreaterThanEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_SGreaterThanEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 782-782
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 785-811
````tablegen
def SPIRV_SLessThanOp : SPIRV_LogicalBinaryOp<"SLessThan",
                                          SPIRV_Integer,
                                          [UsableInSpecConstantOp, SignedOp]> {
  let summary = [{
    Signed-integer comparison if Operand 1 is less than Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.SLessThan %0, %1 : i32
    %5 = spirv.SLessThan %2, %3 : vector<4xi32>

    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_SLessThanOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_SLessThanOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 812-812
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 815-842
````tablegen
def SPIRV_SLessThanEqualOp : SPIRV_LogicalBinaryOp<"SLessThanEqual",
                                               SPIRV_Integer,
                                               [UsableInSpecConstantOp,
                                                SignedOp]> {
  let summary = [{
    Signed-integer comparison if Operand 1 is less than or equal to Operand
    2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.SLessThanEqual %0, %1 : i32
    %5 = spirv.SLessThanEqual %2, %3 : vector<4xi32>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_SLessThanEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_SLessThanEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 843-843
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 846-900
````tablegen
def SPIRV_SelectOp : SPIRV_Op<"Select",
    [Pure,
     AllTypesMatch<["true_value", "false_value", "result"]>,
     UsableInSpecConstantOp,
     DeclareOpInterfaceMethods<SelectLikeOpInterface>]> {
  let summary = [{
    Select between two objects. Before version 1.4, results are only
    computed per component.
  }];

  let description = [{
    Before version 1.4, Result Type must be a pointer, scalar, or vector.

    The types of Object 1 and Object 2 must be the same as Result Type.

    Condition must be a scalar or vector of Boolean type.

    If Condition is a scalar and true, the result is Object 1. If Condition
    is a scalar and false, the result is Object 2.

    If Condition is a vector, Result Type must be a vector with the same
    number of components as Condition and the result is a mix of Object 1
    and Object 2: When a component of Condition is true, the corresponding
    component in the result is taken from Object 1, otherwise it is taken
    from Object 2.

    #### Example:

    ```mlir
    %3 = spirv.Select %0, %1, %2 : i1, f32
    %3 = spirv.Select %0, %1, %2 : i1, vector<3xi32>
    %3 = spirv.Select %0, %1, %2 : vector<3xi1>, vector<3xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_ScalarOrVectorOf<SPIRV_Bool>:$condition,
    SPIRV_SelectType:$true_value,
    SPIRV_SelectType:$false_value
  );

  let results = (outs
    SPIRV_SelectType:$result
  );

  let assemblyFormat = [{
    operands attr-dict `:` type($condition) `,` type($result)
  }];

  // These ops require dynamic availability specification based on operand and
  // result types.
  bit autogenAvailability = 0;

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_SelectOp` as a `def` record for `SPIRVLogicalOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_SelectOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 901-901
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 904-930
````tablegen
def SPIRV_UGreaterThanOp : SPIRV_LogicalBinaryOp<"UGreaterThan",
                                             SPIRV_Integer,
                                             [UnsignedOp,
                                              UsableInSpecConstantOp]> {
  let summary = [{
    Unsigned-integer comparison if Operand 1 is greater than  Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.UGreaterThan %0, %1 : i32
    %5 = spirv.UGreaterThan %2, %3 : vector<4xi32>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_UGreaterThanOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_UGreaterThanOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 931-931
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 934-961
````tablegen
def SPIRV_UGreaterThanEqualOp : SPIRV_LogicalBinaryOp<"UGreaterThanEqual",
                                                  SPIRV_Integer,
                                                  [UnsignedOp,
                                                   UsableInSpecConstantOp]> {
  let summary = [{
    Unsigned-integer comparison if Operand 1 is greater than or equal to
    Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.UGreaterThanEqual %0, %1 : i32
    %5 = spirv.UGreaterThanEqual %2, %3 : vector<4xi32>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_UGreaterThanEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_UGreaterThanEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 962-962
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 965-990
````tablegen
def SPIRV_ULessThanOp : SPIRV_LogicalBinaryOp<"ULessThan",
                                          SPIRV_Integer,
                                          [UnsignedOp, UsableInSpecConstantOp]> {
  let summary = [{
    Unsigned-integer comparison if Operand 1 is less than Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.ULessThan %0, %1 : i32
    %5 = spirv.ULessThan %2, %3 : vector<4xi32>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_ULessThanOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_ULessThanOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 991-991
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 994-1024
````tablegen
def SPIRV_UnorderedOp : SPIRV_LogicalBinaryOp<"Unordered", SPIRV_Float, [Commutative]> {
  let summary = [{
    Result is true if either x or y is an IEEE NaN, otherwise result is
    false.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    x must be a scalar or vector of floating-point type.  It must have the
    same number of components as Result Type.

    y must have the same type as x.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.Unordered %0, %1 : f32
    %5 = spirv.Unordered %2, %3 : vector<4xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_Kernel]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_UnorderedOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_UnorderedOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1025-1025
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1028-1055
````tablegen
def SPIRV_ULessThanEqualOp : SPIRV_LogicalBinaryOp<"ULessThanEqual",
                                               SPIRV_Integer,
                                               [UnsignedOp,
                                                UsableInSpecConstantOp]> {
  let summary = [{
    Unsigned-integer comparison if Operand 1 is less than or equal to
    Operand 2.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    The type of Operand 1 and Operand 2  must be a scalar or vector of
    integer type.  They must have the same component width, and they must
    have the same number of components as Result Type.

    Results are computed per component.

    #### Example:

    ```mlir
    %4 = spirv.ULessThanEqual %0, %1 : i32
    %5 = spirv.ULessThanEqual %2, %3 : vector<4xi32>
    ```
  }];

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_ULessThanEqualOp` as a `def` record for `SPIRVLogicalOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_ULessThanEqualOp` 定义为 `def` 记录，用于描述 `SPIRVLogicalOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1056-1056
````tablegen
#endif // MLIR_DIALECT_SPIRV_IR_LOGICAL_OPS
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

- mlir/Dialect/SPIRV/IR/SPIRVBase.td
- mlir/Interfaces/SideEffectInterfaces.td
- SPIRV_AnyOp builds on SPIRV_Op<"Any", [Pure]>
- SPIRV_AllOp builds on SPIRV_Op<"All", [Pure]>
- SPIRV_FOrdEqualOp builds on SPIRV_LogicalBinaryOp<"FOrdEqual", SPIRV_Float, [Commutative]>
- SPIRV_FOrdGreaterThanOp builds on SPIRV_LogicalBinaryOp<"FOrdGreaterThan", SPIRV_Float, []>
- SPIRV_FOrdGreaterThanEqualOp builds on SPIRV_LogicalBinaryOp<"FOrdGreaterThanEqual", SPIRV_Float, []>
- SPIRV_FOrdLessThanOp builds on SPIRV_LogicalBinaryOp<"FOrdLessThan", SPIRV_Float, []>
- SPIRV_FOrdLessThanEqualOp builds on SPIRV_LogicalBinaryOp<"FOrdLessThanEqual", SPIRV_Float, []>
- SPIRV_FOrdNotEqualOp builds on SPIRV_LogicalBinaryOp<"FOrdNotEqual", SPIRV_Float, [Commutative]>
- SPIRV_FUnordEqualOp builds on SPIRV_LogicalBinaryOp<"FUnordEqual", SPIRV_Float, [Commutative]>
- SPIRV_FUnordGreaterThanOp builds on SPIRV_LogicalBinaryOp<"FUnordGreaterThan", SPIRV_Float, []>
- SPIRV_FUnordGreaterThanEqualOp builds on SPIRV_LogicalBinaryOp<"FUnordGreaterThanEqual", SPIRV_Float, []>
- SPIRV_FUnordLessThanOp builds on SPIRV_LogicalBinaryOp<"FUnordLessThan", SPIRV_Float, []>
- SPIRV_FUnordLessThanEqualOp builds on SPIRV_LogicalBinaryOp<"FUnordLessThanEqual", SPIRV_Float, []>
- SPIRV_FUnordNotEqualOp builds on SPIRV_LogicalBinaryOp<"FUnordNotEqual", SPIRV_Float, [Commutative]>
- SPIRV_IEqualOp builds on SPIRV_LogicalBinaryOp<"IEqual",
- SPIRV_INotEqualOp builds on SPIRV_LogicalBinaryOp<"INotEqual",
- SPIRV_IsFiniteOp builds on SPIRV_LogicalUnaryOp<"IsFinite", SPIRV_Float, []>
- SPIRV_IsInfOp builds on SPIRV_LogicalUnaryOp<"IsInf", SPIRV_Float, []>
