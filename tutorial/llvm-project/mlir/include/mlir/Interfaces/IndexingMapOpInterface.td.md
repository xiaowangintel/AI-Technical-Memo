# IndexingMapOpInterface.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Interfaces/IndexingMapOpInterface.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR IndexingMapOpInterface component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This is the definition file for the IndexingMapOpInterface.
- **用途（CN）**: 为 MLIR 的 IndexingMapOpInterface 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
````tablegen
//===- IndexingMapOpInterface.td - Interface Declaration -*- tablegen -*---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is the definition file for the IndexingMapOpInterface.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_INTERFACES_INDEXING_MAP_OP_INTERFACE
#define MLIR_INTERFACES_INDEXING_MAP_OP_INTERFACE

include "mlir/IR/OpBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 18-144
````tablegen
def IndexingMapOpInterface : OpInterface<"IndexingMapOpInterface"> {
  let description = [{
    Interface for operations that connect an iteration domain to operands via
    affine maps. Provides methods to access indexing maps between iteration
    domain and operand index spaces.
  }];
  let cppNamespace = "::mlir";
  let methods = [
    InterfaceMethod<
      /*desc=*/[{
        Return the indexing maps attribute within the current operation.
      }],
      /*retTy=*/"ArrayAttr",
      /*methodName=*/"getIndexingMaps"
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return the indexing maps within the current operation.
      }],
      /*retTy=*/"SmallVector<AffineMap>",
      /*methodName=*/"getIndexingMapsArray",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        auto range = $_op.getIndexingMaps()
          .template getAsValueRange<AffineMapAttr>();
        return {range.begin(), range.end()};
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return the input or output indexing map for `opOperand`.
      }],
      /*retTy=*/"AffineMap",
      /*methodName=*/"getMatchingIndexingMap",
      /*args=*/(ins "OpOperand*":$opOperand),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        assert(opOperand->getOwner() == this->getOperation());
        auto indexingMaps =
          $_op.getIndexingMaps().template getAsValueRange<AffineMapAttr>();
        return *(indexingMaps.begin() + opOperand->getOperandNumber());
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Hook to provide a custom AffineMap used to compute all the operand
        subshapes given loop bounds. This is used to answer the question: "given
        an iteration space over the codomain, what are the subshapes of the
        operands involved in the computation".
        The default behavior is to just concatenate all the indexing maps.
        A custom AffineMap allows providing a map that can be used to
        compute subshapes even in cases where the concatenation of indexing maps
        (i.e. the data traversal order) is not a simple permutation of the loop
        traversal order. It is then possible to define ops with skewed data
        traversal order for which we can still easily compute hyperrectangular
        loop bounds and subviews.
      }],
      /*retTy=*/"AffineMap",
      /*methodName=*/"getLoopsToShapesMap",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        auto maps =  $_op.getIndexingMapsArray();
        return concatAffineMaps(maps, $_op.getContext());
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Hook to provide a custom AffineMap used to construct the
        hyperrectangular loop iteration space given all the operand subshapes.
        This is used to answer the question:
        "Given a list of operand ranges, what is the subportion of the iteration
        space involved in the computation".
        This is the inverse problem of `getLoopsToShapesMap`.
        Return the empty AffineMap when such an AffineMap cannot be constructed.
        The default behavior is based on a very simple inference procedure that
        only works with permutation affine maps.
        A more advanced Tensor-Comprehension like inference is possible but has
        proven to be ambiguous in unfavorable case.
        A safer and more robust alternative is to allow each op to define
        its own AffineMap.
      }],
      /*retTy=*/"AffineMap",
      /*methodName=*/"getShapesToLoopsMap",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return inversePermutation($_op.getLoopsToShapesMap());
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Returns the static shape of the underlying operand (note this is
        op-specific behavior).
        Returns ShapedType::kDynamic for non-statically-known loop ranges.
      }],
      /*retTy=*/"SmallVector<int64_t>",
      /*methodName=*/"getStaticOperandShape",
      /*args=*/(ins "OpOperand*":$opOperand),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        SmallVector<int64_t> res;
        llvm::append_range(res, $_op.getShape(opOperand));
        return res;
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Returns loop ranges by composing `getShapesToLoopsMap()` with the
        flattened list of operand shapes.
        Returns ShapedType::kDynamic for non-statically-known loop ranges.
      }],
      /*retTy=*/"SmallVector<int64_t>",
      /*methodName=*/"getStaticLoopRanges",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        SmallVector<int64_t> allShapesSizes;
        for (OpOperand &opOperand : this->getOperation()->getOpOperands())
          llvm::append_range(allShapesSizes, $_op.getShape(&opOperand));
        AffineMap invertedMap = $_op.getShapesToLoopsMap();
        assert(invertedMap && "expected a valid op");
        return invertedMap.compose(allShapesSizes);
      }]
    >
  ];
````
- **EN**: This TableGen block defines `IndexingMapOpInterface` as a `def` record for `IndexingMapOpInterface`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `IndexingMapOpInterface` 定义为 `def` 记录，用于描述 `IndexingMapOpInterface` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 145-151
````tablegen
  let extraClassDeclaration = [{
    // Verifier implementation for IndexingMapOpInterface.
    // This must be called manually as part of other verifiers so that the
    // verification order, and meaningful error messages, are not preempted.
    LogicalResult verifyImpl();
  }];
}
````
- **EN**: This block groups callable interfaces such as `verifyImpl`, indicating how `IndexingMapOpInterface` is queried or updated.
- **CN**: 该代码块聚合了 `verifyImpl` 等可调用接口，展示了如何查询或更新 `IndexingMapOpInterface`。

### Lines 152-152
````tablegen
#endif // MLIR_INTERFACES_INDEXING_MAP_OP_INTERFACE
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
- IndexingMapOpInterface builds on OpInterface<"IndexingMapOpInterface">
