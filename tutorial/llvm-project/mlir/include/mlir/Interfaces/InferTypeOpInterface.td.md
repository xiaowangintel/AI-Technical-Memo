# InferTypeOpInterface.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Interfaces/InferTypeOpInterface.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR InferTypeOpInterface component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This file contains a set of interfaces that can be used to define information.
- **用途（CN）**: 为 MLIR 的 InferTypeOpInterface 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
````tablegen
//===- InferTypeOpInterface.td - Infer Type interfaces -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a set of interfaces that can be used to define information
// related to type inference.
//
// This interface is also used by ODS to create builders for operations that
// do not require result type to be specified. Including this interface in
// dialect op definitions is sufficient to result in such builders being
// automatically generated for trivially buildable result types.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_INFERTYPEOPINTERFACE
#define MLIR_INFERTYPEOPINTERFACE

include "mlir/IR/OpBase.td"

// OpInterface to compute the return type of an operation. The arguments match
// those in Operation::create with the exception that the location is optional
// (if no location is provided, then the method will not emit an error on
// mismatch).
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 28-125
````tablegen
def InferTypeOpInterface : OpInterface<"InferTypeOpInterface"> {
  let description = [{
    Interface to infer the return types for an operation that could be used
    during op construction, verification or type inference.
  }];
  let cppNamespace = "::mlir";

  let methods = [
    StaticInterfaceMethod<
      /*desc=*/[{Infer the return types that an op would generate.

      The method takes an optional location which, if set, will be used to
      report errors on. The operands and attributes correspond to those with
      which an Operation would be created (e.g., as used in Operation::create)
      and the regions of the op. Be aware that this method is supposed to be
      called with valid arguments, e.g., operands are verified, or it may result
      in an undefined behavior.

      The inferred result types may be less precise than what may be specified
      directly or produced by refinement, but are required to be compatible
      (as defined by the op's compatibility function).
      }],
      /*retTy=*/"::llvm::LogicalResult",
      /*methodName=*/"inferReturnTypes",
      /*args=*/(ins "::mlir::MLIRContext *":$context,
                    "::std::optional<::mlir::Location>":$location,
                    "::mlir::ValueRange":$operands,
                    "::mlir::DictionaryAttr":$attributes,
                    "::mlir::PropertyRef":$properties,
                    "::mlir::RegionRange":$regions,
                    "::llvm::SmallVectorImpl<::mlir::Type>&":$inferredReturnTypes)
    >,
    StaticInterfaceMethod<
      /*desc=*/[{Refine the return types that an op would generate.

      This method computes the return types as `inferReturnTypes` does but
      additionally takes the existing result types as input. The existing
      result types can be checked as part of inference to provide more
      op-specific error messages as well as part of inference to merge
      additional information, attributes, during inference. It is called during
      verification for ops implementing this trait with default behavior
      reporting mismatch with current and inferred types printed.

      The operands and attributes correspond to those with which an Operation
      would be created (e.g., as used in Operation::create) and the regions of
      the op. The method takes an optional location which, if set, will be used
      to report errors on.

      The return types may be elided or specific elements be null for elements
      that should just be returned but not verified.

      This method may be called from within different stages of IR verification,
      implementations should not assume the arguments to represent fully valid
      IR and are responsible for checking inputs for validity to the degree
      necessary to perform the return type inference.
      }],
      /*retTy=*/"::llvm::LogicalResult",
      /*methodName=*/"refineReturnTypes",
      /*args=*/(ins "::mlir::MLIRContext *":$context,
                    "::std::optional<::mlir::Location>":$location,
                    "::mlir::ValueRange":$operands,
                    "::mlir::DictionaryAttr":$attributes,
                    "::mlir::PropertyRef":$properties,
                    "::mlir::RegionRange":$regions,
                    "::llvm::SmallVectorImpl<::mlir::Type>&":$returnTypes),
      /*methodBody=*/[{}],
      /*defaultImplementation=*/[{
          llvm::SmallVector<Type, 4> inferredReturnTypes;
          if (failed(ConcreteOp::inferReturnTypes(context, location, operands,
                                                  attributes, properties, regions,
                                                  inferredReturnTypes)))
            return failure();
          if (!ConcreteOp::isCompatibleReturnTypes(inferredReturnTypes,
                                                   returnTypes)) {
            return emitOptionalError(
                location, "'", ConcreteOp::getOperationName(),
                "' op inferred type(s) ", inferredReturnTypes,
                " are incompatible with return type(s) of operation ",
                returnTypes);
          }
          return success();
      }]
    >,
    StaticInterfaceMethod<
      /*desc=*/"Returns whether two type ranges are compatible result types.",
      /*retTy=*/"bool",
      /*methodName=*/"isCompatibleReturnTypes",
      /*args=*/(ins "::mlir::TypeRange":$lhs, "::mlir::TypeRange":$rhs),
      /*methodBody=*/[{
        return ConcreteOp::isCompatibleReturnTypes(lhs, rhs);
      }],
      /*defaultImplementation=*/[{
        /// Returns whether two arrays are equal as strongest check for
        /// compatibility by default.
        return lhs == rhs;
      }]
    >,
  ];
````
- **EN**: This TableGen block defines `InferTypeOpInterface` as a `def` record for `InferTypeOpInterface`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `InferTypeOpInterface` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 126-131
````tablegen
  // Inferring result types may need to access the region operations.
  let verifyWithRegions = 1;
  let verify = [{
    return detail::verifyInferredResultTypes($_op);
  }];
}
````
- **EN**: This block groups callable interfaces such as `verifyInferredResultTypes`, indicating how `InferTypeOpInterface` is queried or updated.
- **CN**: 该代码块聚合了 `verifyInferredResultTypes` 等可调用接口，展示了如何查询或更新 `InferTypeOpInterface`。

### Lines 134-200
````tablegen
def InferShapedTypeOpInterface : OpInterface<"InferShapedTypeOpInterface"> {
  let description = [{
    Interface to infer the components of a ShapedType returned by an operation
    that could be used during op construction, verification or shape inference.

    The components consists of element type, shape and raw attribute.
  }];
  let cppNamespace = "::mlir";

  let methods = [
    StaticInterfaceMethod<
      /*desc=*/[{Infer the components of return type of shape containter.

      The method takes an optional location which, if set, will be used to
      report errors on. The operands and attributes correspond to those with
      which an Operation would be created (e.g., as used in Operation::create)
      and the regions of the op.

      Unknown (e.g., unranked) shape and nullptrs for element type and attribute
      may be returned by this function while returning success. E.g., partial
      population of components is not error condition.

      Because this method can be called from within different stages of IR
      verification, implementations should not assume the arguments to
      represent fully valid IR and are responsible for checking inputs for
      validity to the degree necessary to perform the return type inference.
      }],
      /*retTy=*/"::llvm::LogicalResult",
      /*methodName=*/"inferReturnTypeComponents",
      /*args=*/(ins "::mlir::MLIRContext*":$context,
                    "::std::optional<::mlir::Location>":$location,
                    "::mlir::ValueShapeRange":$operands,
                    "::mlir::DictionaryAttr":$attributes,
                    "::mlir::PropertyRef":$properties,
                    "::mlir::RegionRange":$regions,
                    "::llvm::SmallVectorImpl<::mlir::ShapedTypeComponents>&":
                      $inferredReturnShapes),
      /*methodBody=*/[{}],
      /*defaultImplementation=*/[{ return ::mlir::failure(); }]
    >,
    InterfaceMethod<
      /*desc=*/[{Reify the shape computation for the operation.

      Insert operations using the given OpBuilder that computes the
      result shape. This interface is supposed to be workable during dialect
      conversion (e.g. convert from tensor world to buffer world),
      where `getOperand` may be invalid. For example, some ops (e.g.
      dynamic_reshape(input, target_shape)) may depend on their operands
      to calculate the result shape. When the `matchAndRewrite ` method
      of a conversion pattern is called, the operands of the op to convert
      may have been converted into other types, which makes it invalid to
      call the `getOperand` method of such op directly inside the
      conversion pattern.  To solve this problem, this interface follows
      the design of the conversion pattern, that is, accepting passed in
      operands to avoid calling `getOperand` directly inside the interface
      implementation.
      }],
      /*retTy=*/"::llvm::LogicalResult",
      /*methodName=*/"reifyReturnTypeShapes",
      /*args=*/(ins "::mlir::OpBuilder&":$builder,
          "::mlir::ValueRange":$operands,
          "::llvm::SmallVectorImpl<::mlir::Value> &":$reifiedReturnShapes),
      /*methodBody=*/[{}],
      /*defaultImplementation=*/[{ return ::mlir::failure(); }]
    >
  ];
}
````
- **EN**: This TableGen block defines `InferShapedTypeOpInterface` as a `def` record for `InferTypeOpInterface`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `InferShapedTypeOpInterface` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 201-202
````tablegen
// Convenient trait to define a wrapper to inferReturnTypes that passes in the
// Op Adaptor directly
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 204-204
````tablegen
class InferTypeOpAdaptorBase<code additionalDecls = [{}]> : TraitList<
````
- **EN**: This TableGen block defines `InferTypeOpAdaptorBase` as a `class` record for `InferTypeOpInterface`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `InferTypeOpAdaptorBase` 定义为 `class` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 205-231
````tablegen
  [
    // Op implements infer type op interface.
    DeclareOpInterfaceMethods<InferTypeOpInterface>,
    NativeOpTrait<
      /*name=*/"InferTypeOpAdaptor",
      /*traits=*/[],
      /*extraOpDeclaration=*/[{
        static ::llvm::LogicalResult
        inferReturnTypes(::mlir::MLIRContext *context,
                                std::optional<::mlir::Location> location,
                                Adaptor adaptor,
                                ::llvm::SmallVectorImpl<::mlir::Type> &inferredReturnTypes);
      }] # additionalDecls,
      /*extraOpDefinition=*/[{
        ::llvm::LogicalResult
        $cppClass::inferReturnTypes(::mlir::MLIRContext *context,
                          std::optional<::mlir::Location> location,
                          ::mlir::ValueRange operands, ::mlir::DictionaryAttr attributes,
                          ::mlir::PropertyRef properties, ::mlir::RegionRange regions,
                          ::llvm::SmallVectorImpl<::mlir::Type> &inferredReturnTypes) {
          $cppClass::Adaptor adaptor(operands, attributes, properties, regions);
          return $cppClass::inferReturnTypes(context,
            location, adaptor, inferredReturnTypes);
        }
      }]
    >
  ]>;
````
- **EN**: This block groups callable interfaces such as `inferReturnTypes`, `adaptor`, indicating how `InferTypeOpInterface` is queried or updated.
- **CN**: 该代码块聚合了 `inferReturnTypes`, `adaptor` 等可调用接口，展示了如何查询或更新 `InferTypeOpInterface`。

### Lines 233-233
````tablegen
def InferTypeOpAdaptor : InferTypeOpAdaptorBase;
````
- **EN**: This TableGen block defines `InferTypeOpAdaptor` as a `def` record for `InferTypeOpInterface`.
- **CN**: 该 TableGen 代码块将 `InferTypeOpAdaptor` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。

### Lines 234-237
````tablegen
def InferTypeOpAdaptorWithIsCompatible : InferTypeOpAdaptorBase<
  [{
    static bool isCompatibleReturnTypes(::mlir::TypeRange l, ::mlir::TypeRange r);
  }]
````
- **EN**: This TableGen block defines `InferTypeOpAdaptorWithIsCompatible` as a `def` record for `InferTypeOpInterface`.
- **CN**: 该 TableGen 代码块将 `InferTypeOpAdaptorWithIsCompatible` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。

### Lines 238-241
````tablegen
>;

// Convenient trait to define a wrapper to inferReturnTypeComponents that passes
// in the Op Adaptor directly. Only uses the current types of the operands.
````
- **EN**: This section focuses on >;, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“>;”这一主题，把相关声明与辅助接口组织在一起。

### Lines 242-255
````tablegen
class InferShapedTypeOpAdaptorBase<list<string> overridenMethods = []> : TraitList<
  [
    // Op implements infer type op interface.
    DeclareOpInterfaceMethods<InferShapedTypeOpInterface, overridenMethods>,
    NativeOpTrait<
      /*name=*/"InferShapedTypeOpAdaptor",
      /*traits=*/[],
      /*extraOpDeclaration=*/[{
        static ::llvm::LogicalResult
        inferReturnTypeComponents(::mlir::MLIRContext *context,
                                std::optional<::mlir::Location> location,
                                Adaptor adaptor,
                                ::llvm::SmallVectorImpl<::mlir::ShapedTypeComponents> &inferredReturnShapes);
      }],
````
- **EN**: This TableGen block defines `InferShapedTypeOpAdaptorBase` as a `class` record for `InferTypeOpInterface`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `InferShapedTypeOpAdaptorBase` 定义为 `class` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 256-269
````tablegen
      /*extraOpDefinition=*/[{
        ::llvm::LogicalResult
        $cppClass::inferReturnTypeComponents(::mlir::MLIRContext *context,
                          std::optional<::mlir::Location> location,
                          ::mlir::ValueShapeRange operands, ::mlir::DictionaryAttr attributes,
                          ::mlir::PropertyRef properties, ::mlir::RegionRange regions,
                          ::llvm::SmallVectorImpl<::mlir::ShapedTypeComponents> &inferredReturnShapes) {
          $cppClass::Adaptor adaptor(operands, attributes, properties, regions);
          return $cppClass::inferReturnTypeComponents(context,
            location, adaptor, inferredReturnShapes);
        }
      }]
    >
  ]>;
````
- **EN**: This block groups callable interfaces such as `inferReturnTypeComponents`, `adaptor`, indicating how `InferTypeOpInterface` is queried or updated.
- **CN**: 该代码块聚合了 `inferReturnTypeComponents`, `adaptor` 等可调用接口，展示了如何查询或更新 `InferTypeOpInterface`。

### Lines 271-272
````tablegen
def InferShapedTypeOpAdaptor : InferShapedTypeOpAdaptorBase<[
  "inferReturnTypeComponents"]>;
````
- **EN**: This TableGen block defines `InferShapedTypeOpAdaptor` as a `def` record for `InferTypeOpInterface`.
- **CN**: 该 TableGen 代码块将 `InferShapedTypeOpAdaptor` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。

### Lines 273-277
````tablegen
def InferShapedTypeOpAdaptorWithReify : InferShapedTypeOpAdaptorBase<[
  "inferReturnTypeComponents", "reifyReturnTypeShapes"]>;

// Convenience class grouping together type and shaped type op interfaces for
// ops that have tensor return types.
````
- **EN**: This TableGen block defines `InferShapedTypeOpAdaptorWithReify` as a `def` record for `InferTypeOpInterface`.
- **CN**: 该 TableGen 代码块将 `InferShapedTypeOpAdaptorWithReify` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。

### Lines 278-290
````tablegen
class InferTensorTypeBase<list<string> overridenMethods = []> : TraitList<
  [
    // Op implements infer type op interface.
    DeclareOpInterfaceMethods<InferTypeOpInterface>,
    // The op will have methods implementing the ShapedType type inference
    // interface.
    DeclareOpInterfaceMethods<InferShapedTypeOpInterface, overridenMethods>,
    // The op produces tensors and will use the ShapedType type infer interface
    // along with knowledge that it is producing Tensors to infer the type.
    NativeOpTrait<
      /*name=*/"InferTensorType",
      /*traits=*/[],
      /*extraOpDeclaration=*/[{}],
````
- **EN**: This TableGen block defines `InferTensorTypeBase` as a `class` record for `InferTypeOpInterface`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `InferTensorTypeBase` 定义为 `class` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 291-308
````tablegen
      /*extraOpDefinition=*/[{
        ::llvm::LogicalResult
        $cppClass::inferReturnTypes(::mlir::MLIRContext *context,
                          std::optional<::mlir::Location> location,
                          ::mlir::ValueRange operands, ::mlir::DictionaryAttr attributes,
                          ::mlir::PropertyRef properties, ::mlir::RegionRange regions,
                          ::llvm::SmallVectorImpl<::mlir::Type> &inferredReturnTypes) {
          ::llvm::SmallVector<::mlir::ShapedTypeComponents, 2> retComponents;
          if (failed($cppClass::inferReturnTypeComponents(context, location,
                                    operands, attributes, properties, regions,
                                    retComponents)))
            return failure();
          return ::mlir::detail::inferReturnTensorTypes(retComponents,
                                    inferredReturnTypes);
        }
      }]
    >
  ]>;
````
- **EN**: This block groups callable interfaces such as `inferReturnTypes`, `failed`, `inferReturnTypeComponents`, `failure`, indicating how `InferTypeOpInterface` is queried or updated.
- **CN**: 该代码块聚合了 `inferReturnTypes`, `failed`, `inferReturnTypeComponents`, `failure` 等可调用接口，展示了如何查询或更新 `InferTypeOpInterface`。

### Lines 310-310
````tablegen
def InferTensorType : InferTensorTypeBase<["inferReturnTypeComponents"]>;
````
- **EN**: This TableGen block defines `InferTensorType` as a `def` record for `InferTypeOpInterface`.
- **CN**: 该 TableGen 代码块将 `InferTensorType` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。

### Lines 311-315
````tablegen
def InferTensorTypeWithReify: InferTensorTypeBase<[
    "inferReturnTypeComponents", "reifyReturnTypeShapes"]>;

// Convenience class grouping together type and shaped type op interfaces for
// ops that have tensor return types.
````
- **EN**: This TableGen block defines `InferTensorTypeWithReify` as a `def` record for `InferTypeOpInterface`.
- **CN**: 该 TableGen 代码块将 `InferTensorTypeWithReify` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。

### Lines 316-328
````tablegen
class InferTensorTypeAdaptorBase<list<string> overridenMethods = []> : TraitList<
  [
    // Op implements infer type op interface.
    DeclareOpInterfaceMethods<InferTypeOpInterface>,
    // The op will have methods implementing the ShapedType type inference
    // interface.
    InferShapedTypeOpAdaptorBase<overridenMethods>,
    // The op produces tensors and will use the ShapedType type infer interface
    // along with knowledge that it is producing Tensors to infer the type.
    NativeOpTrait<
      /*name=*/"InferTensorType",
      /*traits=*/[],
      /*extraOpDeclaration=*/[{}],
````
- **EN**: This TableGen block defines `InferTensorTypeAdaptorBase` as a `class` record for `InferTypeOpInterface`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `InferTensorTypeAdaptorBase` 定义为 `class` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 329-346
````tablegen
      /*extraOpDefinition=*/[{
        LogicalResult
        $cppClass::inferReturnTypes(::mlir::MLIRContext *context,
                          std::optional<::mlir::Location> location,
                          ::mlir::ValueRange operands, ::mlir::DictionaryAttr attributes,
                          ::mlir::PropertyRef properties, ::mlir::RegionRange regions,
                          ::llvm::SmallVectorImpl<::mlir::Type> &inferredReturnTypes) {
          SmallVector<ShapedTypeComponents, 2> retComponents;
          if (failed($cppClass::inferReturnTypeComponents(context, location,
                                    operands, attributes, properties, regions,
                                    retComponents)))
            return failure();
          return ::mlir::detail::inferReturnTensorTypes(retComponents,
                                    inferredReturnTypes);
        }
      }]
    >
  ]>;
````
- **EN**: This block groups callable interfaces such as `inferReturnTypes`, `failed`, `inferReturnTypeComponents`, `failure`, indicating how `InferTypeOpInterface` is queried or updated.
- **CN**: 该代码块聚合了 `inferReturnTypes`, `failed`, `inferReturnTypeComponents`, `failure` 等可调用接口，展示了如何查询或更新 `InferTypeOpInterface`。

### Lines 348-348
````tablegen
def InferTensorTypeAdaptor : InferTensorTypeAdaptorBase<["inferReturnTypeComponents"]>;
````
- **EN**: This TableGen block defines `InferTensorTypeAdaptor` as a `def` record for `InferTypeOpInterface`.
- **CN**: 该 TableGen 代码块将 `InferTensorTypeAdaptor` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。

### Lines 349-350
````tablegen
def InferTensorTypeAdaptorWithReify: InferTensorTypeAdaptorBase<[
    "inferReturnTypeComponents", "reifyReturnTypeShapes"]>;
````
- **EN**: This TableGen block defines `InferTensorTypeAdaptorWithReify` as a `def` record for `InferTypeOpInterface`.
- **CN**: 该 TableGen 代码块将 `InferTensorTypeAdaptorWithReify` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。

### Lines 352-436
````tablegen
def ReifyRankedShapedTypeOpInterface :
    OpInterface<"ReifyRankedShapedTypeOpInterface"> {
  let description = [{
    Interface to compute the shape of the result of an operation when
    the result is a ranked shape type, i.e. `RankedTensorType` or
    `MemRefType`.
  }];
  let cppNamespace = "::mlir";

  let methods = [
    InterfaceMethod<
      /*desc=*/[{
        Reify the shapes of all the result of an operation (typically in terms 
        of the shape of its operands).

        `reifiedReturnShapes` is populated with one vector per op result. Each
        of those vectors contains an OpFoldResult for each dimension of the
        shaped type. The given builder may be used to insert ops that compute
        result shapes.

        If the shape of a particular result cannot be computed it in terms of
        its operands it must be left empty. If any dimension of the result cannot
        be computed it must be set to OpFoldResult().
      }],
      /*retTy=*/"::llvm::LogicalResult",
      /*methodName=*/"reifyResultShapes",
      /*args=*/(ins "::mlir::OpBuilder &":$builder,
        "::mlir::ReifiedRankedShapedTypeDims &":$reifiedReturnShapes),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{ return ::mlir::failure(); }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Reify the shape of a single result of an operation (typically in terms 
        of the shape of its operands).

        Returns the shape of a single result of the operation as a
        `SmallVector<OpFoldResult>`, one per dimension of the shaped type. The
        given builder may be used to insert ops that compute result shapes.

        If any dimension of the result cannot be computed it must be set to
        OpFoldResult().
      }],
      /*retTy=*/"::llvm::FailureOr<::llvm::SmallVector<::mlir::OpFoldResult>>",
      /*methodName=*/"reifyShapeOfResult",
      /*args=*/(ins "::mlir::OpBuilder &":$builder,
        "int":$resultIndex),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        ReifiedRankedShapedTypeDims reifiedShapes;
        if (failed(cast<ReifyRankedShapedTypeOpInterface>($_op.getOperation()).reifyResultShapes(builder, reifiedShapes)))
          return failure();
        if (resultIndex < 0 || resultIndex >= static_cast<int>(reifiedShapes.size()))
          return $_op.emitOpError("invalid result index");
        return reifiedShapes[resultIndex];
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Reify the shape of a dimension of a given result of an operation
        (typically in terms of the shape of its operands).

        Returns the shape of a specific dimension of a result of the operation as
        an OpFoldResult. The given builder may be used to insert ops that compute
        the shapes.

        If the dimension of the result cannot be computed the method must return
        `failure()`.
      }],
      /*retTy=*/"::llvm::FailureOr<::mlir::OpFoldResult>",
      /*methodName=*/"reifyDimOfResult",
      /*args=*/(ins "::mlir::OpBuilder &":$builder,
        "int":$resultIndex, "int":$dim),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        auto shapes = cast<ReifyRankedShapedTypeOpInterface>($_op.getOperation()).reifyShapeOfResult(builder, resultIndex);
        if (failed(shapes))
          return failure();
        if (dim < 0 || dim >= static_cast<int>((*shapes).size()))
          return $_op.emitOpError("invalid dimension");
        return (*shapes)[dim];
      }]
    >
  ];
}
````
- **EN**: This TableGen block defines `ReifyRankedShapedTypeOpInterface` as a `def` record for `InferTypeOpInterface`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ReifyRankedShapedTypeOpInterface` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 437-438
````tablegen
// Op has the same operand and result type.
// TODO: Change from hard coded to utilizing type inference trait.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 440-442
````tablegen
def SameOperandsAndResultType : NativeOpTrait<"SameOperandsAndResultType">;

// Op has the same ranks for all operands and results types, if known.
````
- **EN**: This TableGen block defines `SameOperandsAndResultType` as a `def` record for `InferTypeOpInterface`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SameOperandsAndResultType` 定义为 `def` 记录，用于描述 `InferTypeOpInterface` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 443-445
````tablegen
def SameOperandsAndResultRank : NativeOpTrait<"SameOperandsAndResultRank">;

#endif // MLIR_INFERTYPEOPINTERFACE
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Trait and interface composition
  **CN**: trait 与接口组合
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/IR/OpBase.td
- InferTypeOpInterface builds on OpInterface<"InferTypeOpInterface">
- InferShapedTypeOpInterface builds on OpInterface<"InferShapedTypeOpInterface">
- InferTypeOpAdaptor builds on InferTypeOpAdaptorBase;
- InferTypeOpAdaptorWithIsCompatible builds on InferTypeOpAdaptorBase<
- InferShapedTypeOpAdaptor builds on InferShapedTypeOpAdaptorBase<[
- InferShapedTypeOpAdaptorWithReify builds on InferShapedTypeOpAdaptorBase<[
- InferTensorType builds on InferTensorTypeBase<["inferReturnTypeComponents"]>;
- InferTensorTypeWithReify builds on InferTensorTypeBase<[
- InferTensorTypeAdaptor builds on InferTensorTypeAdaptorBase<["inferReturnTypeComponents"]>;
- InferTensorTypeAdaptorWithReify builds on InferTensorTypeAdaptorBase<[
- ReifyRankedShapedTypeOpInterface builds on OpInterface<"ReifyRankedShapedTypeOpInterface">
- SameOperandsAndResultType builds on NativeOpTrait<"SameOperandsAndResultType">;
- SameOperandsAndResultRank builds on NativeOpTrait<"SameOperandsAndResultRank">;
