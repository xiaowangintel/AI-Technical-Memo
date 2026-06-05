# SPIRVNonUniformOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/IR/SPIRVNonUniformOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR SPIRVNonUniformOps component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This file contains non-uniform ops for the SPIR-V dialect. It corresponds to.
- **用途（CN）**: 为 MLIR 的 SPIRVNonUniformOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
````tablegen
//===-- SPIRVNonUniformOps.td - MLIR SPIR-V NonUniform Ops -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains non-uniform ops for the SPIR-V dialect. It corresponds to
// "3.32.24. Non-Uniform Instructions" of the SPIR-V specification.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_IR_NON_UNIFORM_OPS
#define MLIR_DIALECT_SPIRV_IR_NON_UNIFORM_OPS

// -----
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 19-40
````tablegen
class SPIRV_GroupNonUniformArithmeticOp<string mnemonic, Type type,
      list<Trait> traits = []> : SPIRV_Op<mnemonic, !listconcat([
        SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>
      ], traits)> {

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_GroupOperationAttr:$group_operation,
    SPIRV_ScalarOrVectorOf<type>:$value,
    Optional<SPIRV_Integer>:$cluster_size
  );

  let results = (outs
    SPIRV_ScalarOrVectorOf<type>:$result
  );  
  
  let hasCustomAssemblyFormat = 0;

  let assemblyFormat = [{
    $execution_scope $group_operation $value (`cluster_size``(` $cluster_size^ `)`)? attr-dict `:` type($value) (`,` type($cluster_size)^)? `->` type(results)
  }]; 
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformArithmeticOp` as a `class` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformArithmeticOp` 定义为 `class` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, trait/接口组合。

### Lines 41-41
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 44-96
````tablegen
def SPIRV_GroupNonUniformBallotOp : SPIRV_Op<"GroupNonUniformBallot",[
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Result is a bitfield value combining the Predicate value from all
    invocations in the group that execute the same dynamic instance of this
    instruction. The bit is set to one if the corresponding invocation is
    active and the Predicate for that invocation evaluated to true;
    otherwise, it is set to zero.
  }];

  let description = [{
    Result Type  must be a vector of four components of integer type scalar,
    whose Signedness operand is 0.

    Result is a set of bitfields where the first invocation is represented
    in the lowest bit of the first vector component and the last (up to the
    size of the group) is the higher bit number of the last bitmask needed
    to represent all bits of the group invocations.

    Execution must be Workgroup or Subgroup Scope.

    Predicate must be a Boolean type.

    #### Example:

    ```mlir
    %0 = spirv.GroupNonUniformBallot <Subgroup> %predicate : vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformBallot]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_Bool:$predicate
  );

  let results = (outs
    SPIRV_IOrUIVec4:$result
  );

  let assemblyFormat = [{
    $execution_scope $predicate attr-dict `:` type($result)
  }];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformBallotOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformBallotOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 97-97
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 100-156
````tablegen
def SPIRV_GroupNonUniformBallotFindLSBOp : SPIRV_Op<"GroupNonUniformBallotFindLSB", [
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Find the least significant bit set to 1 in Value, considering only the
    bits in Value required to represent all bits of the group's invocations.
    If none of the considered bits is set to 1, the resulting value is
    undefined.
  }];

  let description = [{
    Result Type must be a scalar of integer type, whose Signedness operand
    is 0.

    Execution is a Scope that identifies the group of invocations affected
    by this command. It must be Subgroup.

    Value must be a vector of four components of integer type scalar, whose
    Width operand is 32 and whose Signedness operand is 0.

    Value is a set of bitfields where the first invocation is represented in
    the lowest bit of the first vector component and the last (up to the
    size of the group) is the higher bit number of the last bitmask needed
    to represent all bits of the group invocations.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformBallotFindLSB <Subgroup> %vector : vector<4xi32>, i32
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformBallot]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_IOrUIVec4:$value
  );

  let results = (outs
    SPIRV_SignlessOrUnsignedInt:$result
  );

  let assemblyFormat = [{
    $execution_scope $value attr-dict `:` type($value) `,` type($result)
  }];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformBallotFindLSBOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformBallotFindLSBOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 157-157
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 160-216
````tablegen
def SPIRV_GroupNonUniformBallotFindMSBOp : SPIRV_Op<"GroupNonUniformBallotFindMSB", [
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Find the most significant bit set to 1 in Value, considering only the
    bits in Value required to represent all bits of the group's invocations.
    If none of the considered bits is set to 1, the resulting value is
    undefined.
  }];

  let description = [{
    Result Type must be a scalar of integer type, whose Signedness operand
    is 0.

    Execution is a Scope that identifies the group of invocations affected
    by this command. It must be Subgroup.

    Value must be a vector of four components of integer type scalar, whose
    Width operand is 32 and whose Signedness operand is 0.

    Value is a set of bitfields where the first invocation is represented in
    the lowest bit of the first vector component and the last (up to the
    size of the group) is the higher bit number of the last bitmask needed
    to represent all bits of the group invocations.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformBallotFindMSB <Subgroup> %vector : vector<4xi32>, i32
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformBallot]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_IOrUIVec4:$value
  );

  let results = (outs
    SPIRV_SignlessOrUnsignedInt:$result
  );

  let assemblyFormat = [{
    $execution_scope $value attr-dict `:` type($value) `,` type($result)
  }];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformBallotFindMSBOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformBallotFindMSBOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 217-217
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 220-277
````tablegen
def SPIRV_GroupNonUniformBroadcastOp : SPIRV_Op<"GroupNonUniformBroadcast", [
  Pure, AllTypesMatch<["value", "result"]>,
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Result is the Value of the invocation identified by the id Id to all
    active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of floating-point type, integer
    type, or Boolean type.

    Execution must be Workgroup or Subgroup Scope.

    The type of Value must be the same as Result Type.

    Id  must be a scalar of integer type, whose Signedness operand is 0.

    Before version 1.5, Id must come from a constant instruction. Starting
    with version 1.5, Id must be dynamically uniform.

    The resulting value is undefined if Id is an inactive invocation, or is
    greater than or equal to the size of the group.

    #### Example:

    ```mlir
    %scalar_value = ... : f32
    %vector_value = ... : vector<4xf32>
    %id = ... : i32
    %0 = spirv.GroupNonUniformBroadcast <Subgroup> %scalar_value, %id : f32, i32
    %1 = spirv.GroupNonUniformBroadcast <Workgroup> %vector_value, %id :
      vector<4xf32>, i32
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformBallot]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_Type:$value,
    SPIRV_Integer:$id
  );

  let results = (outs
    SPIRV_Type:$result
  );

  let assemblyFormat = [{
    $execution_scope operands attr-dict `:` type($value) `,` type($id)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformBroadcastOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformBroadcastOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 278-278
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 281-333
````tablegen
def SPIRV_GroupNonUniformBroadcastFirstOp : SPIRV_Op<"GroupNonUniformBroadcastFirst", [
  Pure, AllTypesMatch<["value", "result"]>,
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Subgroup"]>]> {

  let summary = [{
    Broadcast the value from the active invocation with the lowest id in
    the subgroup.
  }];

  let description = [{
    Result is the Value of the invocation from the active invocations with
    the lowest id within the Execution scope to all active invocations
    within the Execution scope.

    Result Type must be a scalar or vector of floating-point type, integer
    type, or Boolean type.

    Execution must be Subgroup Scope.

    The type of Value must be the same as Result Type.

    #### Example:

    ```mlir
    %scalar_value = ... : f32
    %vector_value = ... : vector<4xf32>
    %0 = spirv.GroupNonUniformBroadcastFirst <Subgroup> %scalar_value : f32
    %1 = spirv.GroupNonUniformBroadcastFirst <Subgroup> %vector_value : vector<4xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformBallot]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    AnyTypeOf<[SPIRV_ScalarOrVectorOf<SPIRV_Float>, SPIRV_ScalarOrVectorOf<SPIRV_Integer>, SPIRV_ScalarOrVectorOf<SPIRV_Bool>]>:$value
  );

  let results = (outs
    AnyTypeOf<[SPIRV_ScalarOrVectorOf<SPIRV_Float>, SPIRV_ScalarOrVectorOf<SPIRV_Integer>, SPIRV_ScalarOrVectorOf<SPIRV_Bool>]>:$result
  );

  let hasVerifier = 0;

  let assemblyFormat = [{
    $execution_scope operands attr-dict `:` type($value)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformBroadcastFirstOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformBroadcastFirstOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 334-334
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 337-375
````tablegen
def SPIRV_GroupNonUniformElectOp : SPIRV_Op<"GroupNonUniformElect", [
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Result is true only in the active invocation with the lowest id in the
    group, otherwise result is false.
  }];

  let description = [{
    Result Type must be a Boolean type.

    Execution must be Workgroup or Subgroup Scope.

    #### Example:

    ```mlir
    %0 = spirv.GroupNonUniformElect <Workgroup> : i1
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniform]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope
  );

  let results = (outs
    SPIRV_Bool:$result
  );

  let assemblyFormat = "$execution_scope attr-dict `:` type($result)";

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformElectOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformElectOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 376-376
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 379-422
````tablegen
def SPIRV_GroupNonUniformFAddOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformFAdd", SPIRV_Float, []> {
  let summary = [{
    A floating point add group operation of all Value operands contributed
    by active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of floating-point type.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is 0. If Operation is ClusteredReduce,
    ClusterSize must be specified.

     The type of Value must be the same as Result Type.  The method used to
    perform the group operation on the contributed Value(s) from active
    invocations is implementation defined.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : f32
    %vector = ... : vector<4xf32>
    %0 = spirv.GroupNonUniformFAdd <Workgroup> <Reduce> %scalar : f32 -> f32
    %1 = spirv.GroupNonUniformFAdd <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xf32>, i32 -> vector<4xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformFAddOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformFAddOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 423-423
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 426-472
````tablegen
def SPIRV_GroupNonUniformFMaxOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformFMax", SPIRV_Float, []> {
  let summary = [{
    A floating point maximum group operation of all Value operands
    contributed by active invocations in by group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of floating-point type.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is -INF. If Operation is ClusteredReduce,
    ClusterSize must be specified.

     The type of Value must be the same as Result Type.  The method used to
    perform the group operation on the contributed Value(s) from active
    invocations is implementation defined. From the set of Value(s) provided
    by active invocations within a subgroup, if for any two Values one of
    them is a NaN, the other is chosen. If all Value(s) that are used by the
    current invocation are NaN, then the result is an undefined value.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : f32
    %vector = ... : vector<4xf32>
    %0 = spirv.GroupNonUniformFMax <Workgroup> <Reduce> %scalar : f32 -> f32
    %1 = spirv.GroupNonUniformFMax <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xf32>, i32 -> vector<4xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformFMaxOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformFMaxOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 473-473
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 476-522
````tablegen
def SPIRV_GroupNonUniformFMinOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformFMin", SPIRV_Float, []> {
  let summary = [{
    A floating point minimum group operation of all Value operands
    contributed by active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of floating-point type.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is +INF. If Operation is ClusteredReduce,
    ClusterSize must be specified.

     The type of Value must be the same as Result Type.  The method used to
    perform the group operation on the contributed Value(s) from active
    invocations is implementation defined. From the set of Value(s) provided
    by active invocations within a subgroup, if for any two Values one of
    them is a NaN, the other is chosen. If all Value(s) that are used by the
    current invocation are NaN, then the result is an undefined value.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : f32
    %vector = ... : vector<4xf32>
    %0 = spirv.GroupNonUniformFMin <Workgroup> <Reduce> %scalar : f32 -> i32
    %1 = spirv.GroupNonUniformFMin <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xf32>, i32 -> vector<4xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformFMinOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformFMinOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 523-523
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 526-569
````tablegen
def SPIRV_GroupNonUniformFMulOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformFMul", SPIRV_Float, []> {
  let summary = [{
    A floating point multiply group operation of all Value operands
    contributed by active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of floating-point type.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is 1. If Operation is ClusteredReduce,
    ClusterSize must be specified.

     The type of Value must be the same as Result Type.  The method used to
    perform the group operation on the contributed Value(s) from active
    invocations is implementation defined.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : f32
    %vector = ... : vector<4xf32>
    %0 = spirv.GroupNonUniformFMul <Workgroup> <Reduce> %scalar : f32 -> f32
    %1 = spirv.GroupNonUniformFMul <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xf32>, i32 -> vector<4xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformFMulOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformFMulOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 570-570
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 573-614
````tablegen
def SPIRV_GroupNonUniformIAddOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformIAdd", SPIRV_Integer, []> {
  let summary = [{
    An integer add group operation of all Value operands contributed by
    active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of integer type.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is 0. If Operation is ClusteredReduce,
    ClusterSize must be specified.

     The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i32
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformIAdd <Workgroup> <Reduce> %scalar : i32 -> i32
    %1 = spirv.GroupNonUniformIAdd <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xi32>, i32 -> vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformIAddOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformIAddOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 615-615
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 618-659
````tablegen
def SPIRV_GroupNonUniformIMulOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformIMul", SPIRV_Integer, []> {
  let summary = [{
    An integer multiply group operation of all Value operands contributed by
    active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of integer type.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is 1. If Operation is ClusteredReduce,
    ClusterSize must be specified.

     The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i32
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformIMul <Workgroup> <Reduce> %scalar : i32 -> i32
    %1 = spirv.GroupNonUniformIMul <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xi32>, i32 -> vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformIMulOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformIMulOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 660-660
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 663-706
````tablegen
def SPIRV_GroupNonUniformSMaxOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformSMax",
                                    SPIRV_Integer,
                                    [SignedOp]> {
  let summary = [{
    A signed integer maximum group operation of all Value operands
    contributed by active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of integer type.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is INT_MIN. If Operation is
    ClusteredReduce, ClusterSize must be specified.

     The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i32
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformSMax <Workgroup> <Reduce> %scalar : i32
    %1 = spirv.GroupNonUniformSMax <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xi32>, i32 -> vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformSMaxOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformSMaxOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 707-707
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 710-753
````tablegen
def SPIRV_GroupNonUniformSMinOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformSMin",
                                    SPIRV_Integer,
                                    [SignedOp]> {
  let summary = [{
    A signed integer minimum group operation of all Value operands
    contributed by active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of integer type.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is INT_MAX. If Operation is
    ClusteredReduce, ClusterSize must be specified.

     The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i32
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformSMin <Workgroup> <Reduce> %scalar : i32 -> i32
    %1 = spirv.GroupNonUniformSMin <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xi32>, i32 -> vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformSMinOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformSMinOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 754-754
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 757-807
````tablegen
def SPIRV_GroupNonUniformShuffleOp : SPIRV_Op<"GroupNonUniformShuffle", [
  Pure, AllTypesMatch<["value", "result"]>,
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Result is the Value of the invocation identified by the id Id.
  }];

  let description = [{
    Result Type  must be a scalar or vector of floating-point type, integer
    type, or Boolean type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

     The type of Value must be the same as Result Type.

    Id  must be a scalar of integer type, whose Signedness operand is 0.

    The resulting value is undefined if Id is an inactive invocation, or is
    greater than or equal to the size of the group.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %0 = spirv.GroupNonUniformShuffle <Subgroup> %val, %id : f32, i32
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformShuffle]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_ScalarOrVector:$value,
    SPIRV_Integer:$id
  );

  let results = (outs
    SPIRV_ScalarOrVector:$result
  );

  let assemblyFormat = [{
    $execution_scope operands attr-dict `:` type($value) `,` type($id)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformShuffleOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformShuffleOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 808-808
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 811-864
````tablegen
def SPIRV_GroupNonUniformShuffleDownOp : SPIRV_Op<"GroupNonUniformShuffleDown", [
  Pure, AllTypesMatch<["value", "result"]>,
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Result is the Value of the invocation identified by the current
    invocation’s id within the group + Delta.
  }];

  let description = [{
    Result Type  must be a scalar or vector of floating-point type, integer
    type, or Boolean type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

     The type of Value must be the same as Result Type.

    Delta  must be a scalar of integer type, whose Signedness operand is 0.

    Delta is treated as unsigned and the resulting value is undefined if
    Delta is greater than or equal to the size of the group, or if the
    current invocation’s id within the group + Delta is either an inactive
    invocation or greater than or equal to the size of the group.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %0 = spirv.GroupNonUniformShuffleDown <Subgroup> %val, %delta : f32, i32
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformShuffleRelative]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_ScalarOrVector:$value,
    SPIRV_Integer:$delta
  );

  let results = (outs
    SPIRV_Type:$result
  );

  let assemblyFormat = [{
    $execution_scope operands attr-dict `:` type($value) `,` type($delta)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformShuffleDownOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformShuffleDownOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 865-865
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 868-920
````tablegen
def SPIRV_GroupNonUniformShuffleUpOp : SPIRV_Op<"GroupNonUniformShuffleUp", [
  Pure, AllTypesMatch<["value", "result"]>,
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Result is the Value of the invocation identified by the current
    invocation’s id within the group - Delta.
  }];

  let description = [{
    Result Type  must be a scalar or vector of floating-point type, integer
    type, or Boolean type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

     The type of Value must be the same as Result Type.

    Delta  must be a scalar of integer type, whose Signedness operand is 0.

    Delta is treated as unsigned and the resulting value is undefined if
    Delta is greater than the current invocation’s id within the group or if
    the selected lane is inactive.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %0 = spirv.GroupNonUniformShuffleUp <Subgroup> %val, %delta : f32, i32
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformShuffleRelative]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_ScalarOrVector:$value,
    SPIRV_Integer:$delta
  );

  let results = (outs
    SPIRV_Type:$result
  );

  let assemblyFormat = [{
    $execution_scope operands attr-dict `:` type($value) `,` type($delta)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformShuffleUpOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformShuffleUpOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 921-921
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 924-976
````tablegen
def SPIRV_GroupNonUniformShuffleXorOp : SPIRV_Op<"GroupNonUniformShuffleXor", [
  Pure, AllTypesMatch<["value", "result"]>,
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Result is the Value of the invocation identified by the current
    invocation’s id within the group xor’ed with Mask.
  }];

  let description = [{
    Result Type  must be a scalar or vector of floating-point type, integer
    type, or Boolean type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

     The type of Value must be the same as Result Type.

    Mask  must be a scalar of integer type, whose Signedness operand is 0.

    The resulting value is undefined if current invocation’s id within the
    group xor’ed with Mask is an inactive invocation, or is greater than or
    equal to the size of the group.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %0 = spirv.GroupNonUniformShuffleXor <Subgroup> %val, %mask : f32, i32
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformShuffle]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_ScalarOrVector:$value,
    SPIRV_Integer:$mask
  );

  let results = (outs
    SPIRV_Type:$result
  );

  let assemblyFormat = [{
    $execution_scope operands attr-dict `:` type($value) `,` type($mask)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformShuffleXorOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformShuffleXorOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 977-977
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 980-1024
````tablegen
def SPIRV_GroupNonUniformUMaxOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformUMax",
                                    SPIRV_Integer,
                                    [UnsignedOp]> {
  let summary = [{
    An unsigned integer maximum group operation of all Value operands
    contributed by active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of integer type, whose
    Signedness operand is 0.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is 0. If Operation is ClusteredReduce,
    ClusterSize must be specified.

     The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i32
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformUMax <Workgroup> <Reduce> %scalar : i32 -> i32
    %1 = spirv.GroupNonUniformUMax <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xi32>, i32 -> vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformUMaxOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformUMaxOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1025-1025
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1028-1072
````tablegen
def SPIRV_GroupNonUniformUMinOp : SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformUMin",
                                    SPIRV_Integer,
                                    [UnsignedOp]> {
  let summary = [{
    An unsigned integer minimum group operation of all Value operands
    contributed by active invocations in the group.
  }];

  let description = [{
    Result Type  must be a scalar or vector of integer type, whose
    Signedness operand is 0.

    Execution must be Workgroup or Subgroup Scope.

    The identity I for Operation is UINT_MAX. If Operation is
    ClusteredReduce, ClusterSize must be specified.

     The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i32
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformUMin <Workgroup> <Reduce> %scalar : i32 -> i32
    %1 = spirv.GroupNonUniformUMin <Subgroup> <ClusteredReduce> %vector cluster_size(%four) : vector<4xi32>, i32 -> vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic, SPIRV_C_GroupNonUniformClustered, SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformUMinOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformUMinOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1073-1073
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1076-1122
````tablegen
def SPIRV_GroupNonUniformBitwiseAndOp :
  SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformBitwiseAnd",
    SPIRV_Integer, []> {
  let summary = [{
    A bitwise `and` group operation of all Value operands contributed by active
    invocations in the group.
  }];

  let description = [{
    Result Type must be a scalar or vector of integer type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

    The identity I for Operation is ~0. If Operation is ClusteredReduce,
    ClusterSize must be present.

    The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i32
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformBitwiseAnd <Workgroup> <Reduce> %scalar : i32 -> i32
    %1 = spirv.GroupNonUniformBitwiseAnd <Subgroup> <ClusteredReduce>
           %vector cluster_size(%four) : vector<4xi32>, i32 -> vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic,
                SPIRV_C_GroupNonUniformClustered,
                SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformBitwiseAndOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformBitwiseAndOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1123-1123
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1126-1172
````tablegen
def SPIRV_GroupNonUniformBitwiseOrOp :
  SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformBitwiseOr",
    SPIRV_Integer, []> {
  let summary = [{
    A bitwise `or` group operation of all Value operands contributed by active
    invocations in the group.
  }];

  let description = [{
    Result Type must be a scalar or vector of integer type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

    The identity I for Operation is 0. If Operation is ClusteredReduce,
    ClusterSize must be present.

    The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i32
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformBitwiseOr <Workgroup> <Reduce> %scalar : i32 -> i32
    %1 = spirv.GroupNonUniformBitwiseOr <Subgroup> <ClusteredReduce>
           %vector cluster_size(%four) : vector<4xi32>, i32 -> vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic,
                SPIRV_C_GroupNonUniformClustered,
                SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformBitwiseOrOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformBitwiseOrOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1173-1173
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1176-1222
````tablegen
def SPIRV_GroupNonUniformBitwiseXorOp :
  SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformBitwiseXor",
    SPIRV_Integer, []> {
  let summary = [{
    A bitwise `xor` group operation of all Value operands contributed by active
    invocations in the group.
  }];

  let description = [{
    Result Type must be a scalar or vector of integer type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

    The identity I for Operation is 0. If Operation is ClusteredReduce,
    ClusterSize must be present.

    The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i32
    %vector = ... : vector<4xi32>
    %0 = spirv.GroupNonUniformBitwiseXor <Workgroup> <Reduce> %scalar : i32 -> i32
    %1 = spirv.GroupNonUniformBitwiseXor <Subgroup> <ClusteredReduce>
           %vector cluster_size(%four) : vector<4xi32>, i32 -> vector<4xi32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic,
                SPIRV_C_GroupNonUniformClustered,
                SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformBitwiseXorOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformBitwiseXorOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1223-1223
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1226-1272
````tablegen
def SPIRV_GroupNonUniformLogicalAndOp :
  SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformLogicalAnd",
    SPIRV_Bool, []> {
  let summary = [{
    A logical `and` group operation of all Value operands contributed by active
    invocations in the group.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

    The identity I for Operation is ~0. If Operation is ClusteredReduce,
    ClusterSize must be present.

    The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i1
    %vector = ... : vector<4xi1>
    %0 = spirv.GroupNonUniformLogicalAnd <Workgroup> <Reduce> %scalar : i1 -> i1
    %1 = spirv.GroupNonUniformLogicalAnd <Subgroup> <ClusteredReduce>
           %vector cluster_size(%four) : vector<4xi1>, i32 -> vector<4xi1>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic,
                SPIRV_C_GroupNonUniformClustered,
                SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformLogicalAndOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformLogicalAndOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1273-1273
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1276-1322
````tablegen
def SPIRV_GroupNonUniformLogicalOrOp :
  SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformLogicalOr",
    SPIRV_Bool, []> {
  let summary = [{
    A logical `or` group operation of all Value operands contributed by active
    invocations in the group.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

    The identity I for Operation is 0. If Operation is ClusteredReduce,
    ClusterSize must be present.

    The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i1
    %vector = ... : vector<4xi1>
    %0 = spirv.GroupNonUniformLogicalOr <Workgroup> <Reduce> %scalar : i1 -> i1
    %1 = spirv.GroupNonUniformLogicalOr <Subgroup> <ClusteredReduce>
           %vector cluster_size(%four) : vector<4xi1>, i32 -> vector<4xi1>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic,
                SPIRV_C_GroupNonUniformClustered,
                SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformLogicalOrOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformLogicalOrOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1323-1323
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1326-1372
````tablegen
def SPIRV_GroupNonUniformLogicalXorOp :
  SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformLogicalXor",
    SPIRV_Bool, []> {
  let summary = [{
    A logical `xor` group operation of all Value operands contributed by active
    invocations in the group.
  }];

  let description = [{
    Result Type must be a scalar or vector of Boolean type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

    The identity I for Operation is 0. If Operation is ClusteredReduce,
    ClusterSize must be present.

    The type of Value must be the same as Result Type.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar
    of integer type, whose Signedness operand is 0. ClusterSize must come
    from a constant instruction. ClusterSize must be at least 1, and must be
    a power of 2. If ClusterSize is greater than the declared SubGroupSize,
    executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %scalar = ... : i1
    %vector = ... : vector<4xi1>
    %0 = spirv.GroupNonUniformLogicalXor <Workgroup> <Reduce> %scalar : i1 -> i1
    %1 = spirv.GroupNonUniformLogicalXor <Subgroup> <ClusteredReduce>
           %vector cluster_size(%four) : vector<4xi1>, i32 -> vector<4xi1>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformArithmetic,
                SPIRV_C_GroupNonUniformClustered,
                SPIRV_C_GroupNonUniformPartitionedNV]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformLogicalXorOp` as a `def` record for `SPIRVNonUniformOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformLogicalXorOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1373-1373
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1376-1437
````tablegen
def SPIRV_GroupNonUniformBallotBitCountOp : SPIRV_Op<"GroupNonUniformBallotBitCount", [
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Subgroup"]>]> {

  let summary = [{
    Result is the number of bits that are set to 1 in Value, considering
    only the bits in Value required to represent all bits of the scope
    restricted tangle.
  }];

  let description = [{
    Result Type must be a scalar of integer type, whose Signedness operand
    is 0.

    Execution is the scope defining the scope restricted tangle affected by
    this command. It must be Subgroup.

    The identity I for Operation is 0.

    Value must be a vector of four components of integer type scalar, whose
    Width operand is 32 and whose Signedness operand is 0.

    Value is a set of bitfields where the first invocation is represented in
    the lowest bit of the first vector component and the last (up to the
    size of the scope) is the higher bit number of the last bitmask needed
    to represent all bits of the invocations in the scope restricted tangle.

    An invocation will not execute a dynamic instance of this instruction
    (X') until all invocations in its scope restricted tangle have executed
    all dynamic instances that are program-ordered before X'.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %count = spirv.GroupNonUniformBallotBitCount <Subgroup> <Reduce> %val : vector<4xi32> -> i32
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformBallot]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_GroupOperationAttr:$group_operation,
    SPIRV_Vec4<SignlessOrUnsignedIntOfWidths<[32]>>:$value
  );

  let results = (outs
    SPIRV_SignlessOrUnsignedInt:$result
  );

  let hasVerifier = 0;

  let assemblyFormat = [{
    $execution_scope $group_operation $value attr-dict `:` type($value) `->` type($result)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformBallotBitCountOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformBallotBitCountOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1438-1438
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1441-1513
````tablegen
def SPIRV_GroupNonUniformRotateKHROp : SPIRV_Op<"GroupNonUniformRotateKHR", [
  Pure, AllTypesMatch<["value", "result"]>,
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Workgroup", "Subgroup"]>]> {

  let summary = [{
    Rotate values across invocations within a subgroup.
  }];

  let description = [{
    Return the Value of the invocation whose id within the group is calculated
    as follows:

    LocalId = SubgroupLocalInvocationId if Execution is Subgroup or
              LocalInvocationId if Execution is Workgroup
    RotationGroupSize = ClusterSize when ClusterSize is present, otherwise
    RotationGroupSize = SubgroupMaxSize if the Kernel capability is declared
                        and SubgroupSize if not.
    Invocation ID = ( (LocalId + Delta) & (RotationGroupSize - 1) ) +
                    (LocalId & ~(RotationGroupSize - 1))

    Result Type must be a scalar or vector of floating-point type, integer
    type, or Boolean type.

    Execution is a Scope. It must be either Workgroup or Subgroup.

    The type of Value must be the same as Result Type.

    Delta must be a scalar of integer type, whose Signedness operand is 0.
    Delta must be dynamically uniform within Execution.

    Delta is treated as unsigned and the resulting value is undefined if the
    selected lane is inactive.

    ClusterSize is the size of cluster to use. ClusterSize must be a scalar of
    integer type, whose Signedness operand is 0. ClusterSize must come from a
    constant instruction. Behavior is undefined unless ClusterSize is at least
    1 and a power of 2. If ClusterSize is greater than the declared
    SubGroupSize, executing this instruction results in undefined behavior.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %four = spirv.Constant 4 : i32
    %0 = spirv.GroupNonUniformRotateKHR <Subgroup> %value, %delta : f32, i32 -> f32
    %1 = spirv.GroupNonUniformRotateKHR <Workgroup> %value, %delta,
         cluster_size(%four) : f32, i32, i32 -> f32
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformRotateKHR]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    AnyTypeOf<[SPIRV_ScalarOrVectorOf<SPIRV_Float>, SPIRV_ScalarOrVectorOf<SPIRV_Integer>, SPIRV_ScalarOrVectorOf<SPIRV_Bool>]>:$value,
    SPIRV_SignlessOrUnsignedInt:$delta,
    Optional<SPIRV_SignlessOrUnsignedInt>:$cluster_size
  );

  let results = (outs
    AnyTypeOf<[SPIRV_ScalarOrVectorOf<SPIRV_Float>, SPIRV_ScalarOrVectorOf<SPIRV_Integer>, SPIRV_ScalarOrVectorOf<SPIRV_Bool>]>:$result
  );

  let assemblyFormat = [{
    $execution_scope $value `,` $delta (`,` `cluster_size` `(` $cluster_size^ `)`)? attr-dict `:` type($value) `,` type($delta) (`,` type($cluster_size)^)? `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformRotateKHROp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformRotateKHROp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1514-1514
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1517-1569
````tablegen
def SPIRV_GroupNonUniformAllOp : SPIRV_Op<"GroupNonUniformAll", [
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Subgroup"]>]> {

  let summary = [{
    Evaluates a predicate for all tangled invocations within the Execution
    scope, resulting in true if predicate evaluates to true for all tangled
    invocations within the Execution scope, otherwise the result is false.
  }];

  let description = [{
    Result Type must be a Boolean type.

    Execution is the scope defining the scope restricted tangle affected by
    this command. It must be Subgroup.

    Predicate must be a Boolean type.

    An invocation will not execute a dynamic instance of this instruction
    (X') until all invocations in its scope restricted tangle have executed
    all dynamic instances that are program-ordered before X'.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %predicate = ... : i1
    %0 = spirv.GroupNonUniformAll <Subgroup> %predicate : i1
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformVote]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_Bool:$predicate
  );

  let results = (outs
    SPIRV_Bool:$result
  );

  let hasVerifier = 0;

  let assemblyFormat = [{
    $execution_scope $predicate attr-dict `:` type($result)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformAllOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformAllOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1570-1570
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1573-1625
````tablegen
def SPIRV_GroupNonUniformAnyOp : SPIRV_Op<"GroupNonUniformAny", [
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Subgroup"]>]> {

  let summary = [{
    Evaluates a predicate for all tangled invocations within the Execution
    scope, resulting in true if predicate evaluates to true for any tangled
    invocations within the Execution scope, otherwise the result is false.
  }];

  let description = [{
    Result Type must be a Boolean type.

    Execution is the scope defining the scope restricted tangle affected by
    this command. It must be Subgroup.

    Predicate must be a Boolean type.

    An invocation will not execute a dynamic instance of this instruction
    (X') until all invocations in its scope restricted tangle have executed
    all dynamic instances that are program-ordered before X'.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %predicate = ... : i1
    %0 = spirv.GroupNonUniformAny <Subgroup> %predicate : i1
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformVote]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    SPIRV_Bool:$predicate
  );

  let results = (outs
    SPIRV_Bool:$result
  );

  let hasVerifier = 0;

  let assemblyFormat = [{
    $execution_scope $predicate attr-dict `:` type($result)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformAnyOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformAnyOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1626-1626
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1629-1686
````tablegen
def SPIRV_GroupNonUniformAllEqualOp : SPIRV_Op<"GroupNonUniformAllEqual", [
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Subgroup"]>]> {

  let summary = [{
    Evaluates a value for all tangled invocations within the Execution
    scope. The result is true if Value is equal for all tangled invocations
    within the Execution scope. Otherwise, the result is false.
  }];

  let description = [{
    Result Type must be a Boolean type.

    Execution is the scope defining the scope restricted tangle affected by
    this command. It must be Subgroup.

    Value must be a scalar or vector of floating-point type, integer type,
    or Boolean type. The compare operation is based on this type, and if it
    is a floating-point type, an ordered-and-equal compare is used.

    An invocation will not execute a dynamic instance of this instruction
    (X') until all invocations in its scope restricted tangle have executed
    all dynamic instances that are program-ordered before X'.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %scalar_value = ... : f32
    %vector_value = ... : vector<4xf32>
    %0 = spirv.GroupNonUniformAllEqual <Subgroup> %scalar_value : f32, i1
    %1 = spirv.GroupNonUniformAllEqual <Subgroup> %vector_value : vector<4xf32>, i1
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformVote]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    AnyTypeOf<[SPIRV_ScalarOrVectorOf<SPIRV_Float>, SPIRV_ScalarOrVectorOf<SPIRV_Integer>, SPIRV_ScalarOrVectorOf<SPIRV_Bool>]>:$value
  );

  let results = (outs
    SPIRV_Bool:$result
  );


  let hasVerifier = 0;

  let assemblyFormat = [{
    $execution_scope $value attr-dict `:` type($value) `,` type($result)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformAllEqualOp` as a `def` record for `SPIRVNonUniformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformAllEqualOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1687-1687
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 1690-1759
````tablegen
def SPIRV_GroupNonUniformQuadSwapOp : SPIRV_Op<"GroupNonUniformQuadSwap", [
  SPIRV_ExecutionScopeAttrIs<"execution_scope", ["Subgroup"]>, AllTypesMatch<["value", "result"]>]> {

  let summary = [{
    Swap the Value of the invocation within the quad with another invocation
    in the quad using Direction.
  }];

  let description = [{
    Result Type must be a scalar or vector of floating-point type, integer type,
    or Boolean type.

    Execution is a Scope, but has no effect on the behavior of this instruction.
    It must be Subgroup.

    The type of Value must be the same as Result Type.

    Direction is the kind of swap to perform.

    Direction must be a scalar of integer type, whose Signedness operand is 0.

    Direction must come from a constant instruction.

    The value returned in Result is the value provided to Value by another invocation
    in the same quad scope instance. The invocation providing this value is
    determined according to Direction.

    A Direction of 0 indicates a horizontal swap;
    - Invocations with quad indices of 0 and 1 swap values
    - Invocations with quad indices of 2 and 3 swap values
    A Direction of 1 indicates a vertical swap;
    - Invocations with quad indices of 0 and 2 swap values
    - Invocations with quad indices of 1 and 3 swap values
    A Direction of 2 indicates a diagonal swap;
    - Invocations with quad indices of 0 and 3 swap values
    - Invocations with quad indices of 1 and 2 swap values

    Direction must be one of the above values.

    If a tangled invocation within the quad reads Value from an invocation not part
    of the tangled invocation within the same quad, the resulting value is undefined.

    An invocation will not execute a dynamic instance of this instruction (X') until
    all invocations in its quad have executed all dynamic instances that are program-ordered
    before X'.

    #### Example:

    ```mlir
    %0 = spirv.GroupNonUniformQuadSwap <Subgroup> <Horizontal> %value : f32
    %1 = spirv.GroupNonUniformQuadSwap <Subgroup> <Vertical> %value : vector<4xf32>
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_3>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_GroupNonUniformQuad]>
  ];

  let arguments = (ins
    SPIRV_ScopeAttr:$execution_scope,
    AnyTypeOf<[SPIRV_ScalarOrVectorOf<SPIRV_Float>, SPIRV_ScalarOrVectorOf<SPIRV_Integer>, SPIRV_ScalarOrVectorOf<SPIRV_Bool>]>:$value,
    SPIRV_QuadSwapDirectionAttr:$direction
  );

  let results = (outs
    AnyTypeOf<[SPIRV_ScalarOrVectorOf<SPIRV_Float>, SPIRV_ScalarOrVectorOf<SPIRV_Integer>, SPIRV_ScalarOrVectorOf<SPIRV_Bool>]>:$result
  );
````
- **EN**: This TableGen block defines `SPIRV_GroupNonUniformQuadSwapOp` as a `def` record for `SPIRVNonUniformOps`. It covers operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_GroupNonUniformQuadSwapOp` 定义为 `def` 记录，用于描述 `SPIRVNonUniformOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1760-1765
````tablegen
  let hasVerifier = 0;

  let assemblyFormat = [{
    $execution_scope $direction $value attr-dict `:` type($value)
  }];
}
````
- **EN**: This block groups callable interfaces such as `type`, indicating how `SPIRVNonUniformOps` is queried or updated.
- **CN**: 该代码块聚合了 `type` 等可调用接口，展示了如何查询或更新 `SPIRVNonUniformOps`。

### Lines 1767-1767
````tablegen
#endif // MLIR_DIALECT_SPIRV_IR_NON_UNIFORM_OPS
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

- SPIRV_GroupNonUniformBallotOp builds on SPIRV_Op<"GroupNonUniformBallot",[
- SPIRV_GroupNonUniformBallotFindLSBOp builds on SPIRV_Op<"GroupNonUniformBallotFindLSB", [
- SPIRV_GroupNonUniformBallotFindMSBOp builds on SPIRV_Op<"GroupNonUniformBallotFindMSB", [
- SPIRV_GroupNonUniformBroadcastOp builds on SPIRV_Op<"GroupNonUniformBroadcast", [
- SPIRV_GroupNonUniformBroadcastFirstOp builds on SPIRV_Op<"GroupNonUniformBroadcastFirst", [
- SPIRV_GroupNonUniformElectOp builds on SPIRV_Op<"GroupNonUniformElect", [
- SPIRV_GroupNonUniformFAddOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformFAdd", SPIRV_Float, []>
- SPIRV_GroupNonUniformFMaxOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformFMax", SPIRV_Float, []>
- SPIRV_GroupNonUniformFMinOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformFMin", SPIRV_Float, []>
- SPIRV_GroupNonUniformFMulOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformFMul", SPIRV_Float, []>
- SPIRV_GroupNonUniformIAddOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformIAdd", SPIRV_Integer, []>
- SPIRV_GroupNonUniformIMulOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformIMul", SPIRV_Integer, []>
- SPIRV_GroupNonUniformSMaxOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformSMax",
- SPIRV_GroupNonUniformSMinOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformSMin",
- SPIRV_GroupNonUniformShuffleOp builds on SPIRV_Op<"GroupNonUniformShuffle", [
- SPIRV_GroupNonUniformShuffleDownOp builds on SPIRV_Op<"GroupNonUniformShuffleDown", [
- SPIRV_GroupNonUniformShuffleUpOp builds on SPIRV_Op<"GroupNonUniformShuffleUp", [
- SPIRV_GroupNonUniformShuffleXorOp builds on SPIRV_Op<"GroupNonUniformShuffleXor", [
- SPIRV_GroupNonUniformUMaxOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformUMax",
- SPIRV_GroupNonUniformUMinOp builds on SPIRV_GroupNonUniformArithmeticOp<"GroupNonUniformUMin",
