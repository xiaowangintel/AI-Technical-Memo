# ViewLikeInterface.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Interfaces/ViewLikeInterface.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR ViewLikeInterface component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: Defines the interface for view-like operations.
- **用途（CN）**: 为 MLIR 的 ViewLikeInterface 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
````tablegen
//===- ViewLikeInterface.td - ViewLike interface -----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Defines the interface for view-like operations.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_INTERFACES_VIEWLIKEINTERFACE
#define MLIR_INTERFACES_VIEWLIKEINTERFACE

include "mlir/IR/OpBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 18-41
````tablegen
def ViewLikeOpInterface : OpInterface<"ViewLikeOpInterface"> {
  let description = [{
    A view-like operation "views" a buffer in a potentially different way. It
    takes in a (view of) buffer (and potentially some other operands) and returns
    another view of buffer.
  }];
  let cppNamespace = "::mlir";

  let methods = [
    InterfaceMethod<
      "Returns the source buffer from which the view is created.",
      "::mlir::Value", "getViewSource">,
    InterfaceMethod<
      /*desc=*/[{ Returns the buffer which the view created. }],
      /*retTy=*/"::mlir::Value",
      /*methodName=*/"getViewDest",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op->getResult(0);
      }]
    >
  ];
}
````
- **EN**: This TableGen block defines `ViewLikeOpInterface` as a `def` record for `ViewLikeInterface`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ViewLikeOpInterface` 定义为 `def` 记录，用于描述 `ViewLikeInterface` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 43-210
````tablegen
def OffsetSizeAndStrideOpInterface : OpInterface<"OffsetSizeAndStrideOpInterface"> {
  let description = [{
    Common interface for ops that allow specifying mixed dynamic and static
    offsets, sizes and strides variadic operands.
    Ops that implement this interface need to expose the following methods:
      1. `getArrayAttrMaxRanks` to specify the length of static integer
          attributes.
      2. `offsets`, `sizes` and `strides` variadic operands.
      3. `static_offsets`, resp. `static_sizes` and `static_strides` integer
          array attributes.
      4. `getOffsetSizeAndStrideStartOperandIndex` method that specifies the
         starting index of the OffsetSizeAndStrideOpInterface operands

    The invariants of this interface are:
      1. `static_offsets`, `static_sizes` and `static_strides` have length
          exactly `getArrayAttrMaxRanks()`[0] (resp. [1], [2]).
      2. `offsets`, `sizes` and `strides` have each length at most
         `getArrayAttrMaxRanks()`[0] (resp. [1], [2]).
      3. if an entry of `static_offsets` (resp. `static_sizes`,
         `static_strides`) is equal to a special sentinel value, namely
         `ShapedType::kDynamic`, then the corresponding entry is a dynamic
         offset (resp. size, stride).
      4. a variadic `offset` (resp. `sizes`, `strides`) operand  must be present
         for each dynamic offset (resp. size, stride).
      5. `offsets`, `sizes` and `strides` operands are specified in this order
         at operand index starting at `getOffsetSizeAndStrideStartOperandIndex`.
      6. `offsets` and `sizes` operands are non-negative.

    This interface is useful to factor out common behavior and provide support
    for carrying or injecting static behavior through the use of the static
    attributes.
  }];

  let cppNamespace = "::mlir";

  let methods = [
    StaticInterfaceMethod<
      /*desc=*/[{
        Return the number of leading operands before the `offsets`, `sizes` and
        and `strides` operands.
      }],
      /*retTy=*/"unsigned",
      /*methodName=*/"getOffsetSizeAndStrideStartOperandIndex",
      /*args=*/(ins)
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return the expected rank of each of the`static_offsets`, `static_sizes`
        and `static_strides` attributes.
      }],
      /*retTy=*/"std::array<unsigned, 3>",
      /*methodName=*/"getArrayAttrMaxRanks",
      /*args=*/(ins)
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return the dynamic offset operands.
      }],
      /*retTy=*/"::mlir::OperandRange",
      /*methodName=*/"getOffsets",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op.getOffsets();
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return the dynamic size operands.
      }],
      /*retTy=*/"::mlir::OperandRange",
      /*methodName=*/"getSizes",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op.getSizes();
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return the dynamic stride operands.
      }],
      /*retTy=*/"::mlir::OperandRange",
      /*methodName=*/"getStrides",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op.getStrides();
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return the static offset attributes.
      }],
      /*retTy=*/"::llvm::ArrayRef<int64_t>",
      /*methodName=*/"getStaticOffsets",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op.getStaticOffsets();
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return the static size attributes.
      }],
      /*retTy=*/"::llvm::ArrayRef<int64_t>",
      /*methodName=*/"getStaticSizes",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op.getStaticSizes();
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return the dynamic stride attributes.
      }],
      /*retTy=*/"::llvm::ArrayRef<int64_t>",
      /*methodName=*/"getStaticStrides",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op.getStaticStrides();
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return a vector of all the static or dynamic offsets of the op.
      }],
      /*retTy=*/"::llvm::SmallVector<::mlir::OpFoldResult>",
      /*methodName=*/"getMixedOffsets",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        Builder b($_op->getContext());
        return ::mlir::getMixedValues($_op.getStaticOffsets(),
                                      $_op.getOffsets(), b);
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return a vector of all the static or dynamic sizes of the op.
      }],
      /*retTy=*/"::llvm::SmallVector<::mlir::OpFoldResult>",
      /*methodName=*/"getMixedSizes",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        Builder b($_op->getContext());
        return
            ::mlir::getMixedValues($_op.getStaticSizes(), $_op.getSizes(), b);
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return a vector of all the static or dynamic strides of the op.
      }],
      /*retTy=*/"::llvm::SmallVector<::mlir::OpFoldResult>",
      /*methodName=*/"getMixedStrides",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        Builder b($_op->getContext());
        return ::mlir::getMixedValues($_op.getStaticStrides(),
                                      $_op.getStrides(), b);
      }]
    >,
````
- **EN**: This TableGen block defines `OffsetSizeAndStrideOpInterface` as a `def` record for `ViewLikeInterface`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `OffsetSizeAndStrideOpInterface` 定义为 `def` 记录，用于描述 `ViewLikeInterface` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 211-279
````tablegen
    InterfaceMethod<
      /*desc=*/"Return true if the offset `idx` is dynamic.",
      /*retTy=*/"bool",
      /*methodName=*/"isDynamicOffset",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return ::mlir::ShapedType::isDynamic(getStaticOffsets()[idx]);
      }]
    >,
    InterfaceMethod<
      /*desc=*/"Return true if the size `idx` is dynamic.",
      /*retTy=*/"bool",
      /*methodName=*/"isDynamicSize",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return ::mlir::ShapedType::isDynamic(getStaticSizes()[idx]);
      }]
    >,
    InterfaceMethod<
      /*desc=*/"Return true if the stride `idx` is dynamic.",
      /*retTy=*/"bool",
      /*methodName=*/"isDynamicStride",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return ::mlir::ShapedType::isDynamic(getStaticStrides()[idx]);
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Assert the offset `idx` is a static constant and return its value.
      }],
      /*retTy=*/"int64_t",
      /*methodName=*/"getStaticOffset",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        assert(!$_op.isDynamicOffset(idx) && "expected static offset");
        return getStaticOffsets()[idx];
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Assert the size `idx` is a static constant and return its value.
      }],
      /*retTy=*/"int64_t",
      /*methodName=*/"getStaticSize",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        assert(!$_op.isDynamicSize(idx) && "expected static size");
        return getStaticSizes()[idx];
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Assert the stride `idx` is a static constant and return its value.
      }],
      /*retTy=*/"int64_t",
      /*methodName=*/"getStaticStride",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        assert(!$_op.isDynamicStride(idx) && "expected static stride");
        return getStaticStrides()[idx];
      }]
    >,
````
- **EN**: This block groups callable interfaces such as `isDynamic`, `getStaticOffsets`, `getStaticSizes`, `getStaticStrides`, indicating how `ViewLikeInterface` is queried or updated.
- **CN**: 该代码块聚合了 `isDynamic`, `getStaticOffsets`, `getStaticSizes`, `getStaticStrides` 等可调用接口，展示了如何查询或更新 `ViewLikeInterface`。

### Lines 281-408
````tablegen
    InterfaceMethod<
      /*desc=*/[{
        Assert the offset `idx` is dynamic and return the position of the
        corresponding operand.
      }],
      /*retTy=*/"unsigned",
      /*methodName=*/"getIndexOfDynamicOffset",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        assert($_op.isDynamicOffset(idx) && "expected dynamic offset");
        auto numDynamic = ::mlir::detail::getNumDynamicEntriesUpToIdx(
          getStaticOffsets(), idx);
        return $_op.getOffsetSizeAndStrideStartOperandIndex() + numDynamic;
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Assert the size `idx` is dynamic and return the position of the
        corresponding operand.
      }],
      /*retTy=*/"unsigned",
      /*methodName=*/"getIndexOfDynamicSize",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        assert($_op.isDynamicSize(idx) && "expected dynamic size");
        auto numDynamic = ::mlir::detail::getNumDynamicEntriesUpToIdx(
          getStaticSizes(), idx);
        return $_op.getOffsetSizeAndStrideStartOperandIndex() +
          getOffsets().size() + numDynamic;
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Assert the stride `idx` is dynamic and return the position of the
        corresponding operand.
      }],
      /*retTy=*/"unsigned",
      /*methodName=*/"getIndexOfDynamicStride",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        assert($_op.isDynamicStride(idx) && "expected dynamic stride");
        auto numDynamic = ::mlir::detail::getNumDynamicEntriesUpToIdx(
          getStaticStrides(), idx);
        return $_op.getOffsetSizeAndStrideStartOperandIndex() +
          getOffsets().size() + getSizes().size() + numDynamic;
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Assert the offset `idx` is dynamic and return its value.
      }],
      /*retTy=*/"::mlir::Value",
      /*methodName=*/"getDynamicOffset",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op.getOperand(getIndexOfDynamicOffset(idx));
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Assert the size `idx` is dynamic and return its value.
      }],
      /*retTy=*/"::mlir::Value",
      /*methodName=*/"getDynamicSize",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op.getOperand(getIndexOfDynamicSize(idx));
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Assert the stride `idx` is dynamic and return its value.
      }],
      /*retTy=*/"::mlir::Value",
      /*methodName=*/"getDynamicStride",
      /*args=*/(ins "unsigned":$idx),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return $_op.getOperand(getIndexOfDynamicStride(idx));
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{
        Return true if all `other`'s offsets, sizes and strides are the same.
        Takes a custom `cmp` comparison function on OpFoldResult to avoid taking
        a dialect dependence.
      }],
      /*retTy=*/"bool",
      /*methodName=*/"isSameAs",
      /*args=*/(ins "::mlir::OffsetSizeAndStrideOpInterface":$other,
                    "::llvm::function_ref<bool(::mlir::OpFoldResult, ::mlir::OpFoldResult)>":$cmp),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return ::mlir::detail::sameOffsetsSizesAndStrides(
          ::mlir::cast<::mlir::OffsetSizeAndStrideOpInterface>(
            $_op.getOperation()), other, cmp);
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{ Return true if all strides are guaranteed to be 1. }],
      /*retTy=*/"bool",
      /*methodName=*/"hasUnitStride",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return ::llvm::all_of(getMixedStrides(), [](::mlir::OpFoldResult ofr) {
          return ::mlir::getConstantIntValue(ofr) == static_cast<int64_t>(1);
        });
      }]
    >,
    InterfaceMethod<
      /*desc=*/[{ Return true if all offsets are guaranteed to be 0. }],
      /*retTy=*/"bool",
      /*methodName=*/"hasZeroOffset",
      /*args=*/(ins),
      /*methodBody=*/"",
      /*defaultImplementation=*/[{
        return ::llvm::all_of(getMixedOffsets(), [](::mlir::OpFoldResult ofr) {
          return ::mlir::getConstantIntValue(ofr) == static_cast<int64_t>(0);
        });
      }]
    >,
  ];
````
- **EN**: This block groups callable interfaces such as `isDynamicOffset`, `getNumDynamicEntriesUpToIdx`, `getStaticOffsets`, `getOffsetSizeAndStrideStartOperandIndex`, indicating how `ViewLikeInterface` is queried or updated.
- **CN**: 该代码块聚合了 `isDynamicOffset`, `getNumDynamicEntriesUpToIdx`, `getStaticOffsets`, `getOffsetSizeAndStrideStartOperandIndex` 等可调用接口，展示了如何查询或更新 `ViewLikeInterface`。

### Lines 410-414
````tablegen
  let verify = [{
    return ::mlir::detail::verifyOffsetSizeAndStrideOp(
        ::mlir::cast<::mlir::OffsetSizeAndStrideOpInterface>($_op));
  }];
}
````
- **EN**: This block groups callable interfaces such as `verifyOffsetSizeAndStrideOp`, indicating how `ViewLikeInterface` is queried or updated.
- **CN**: 该代码块聚合了 `verifyOffsetSizeAndStrideOp` 等可调用接口，展示了如何查询或更新 `ViewLikeInterface`。

### Lines 416-425
````tablegen
// This trai indicates that pointer-like objects (such as memrefs) returned
// from this operation will never alias with each other. This provides a
// guarantee to optimization passes that accesses through different results
// of this operation can be safely reordered, as they will never reference
// overlapping memory locations.
//
// Operations with this trait take multiple pointer-like operands
// and return the same operands with additional non-aliasing guarantees.
// If the access to the results of this operation aliases at runtime, the
// behavior of such access is undefined.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 427-429
````tablegen
def DistinctObjectsTrait : NativeOpTrait<"DistinctObjectsTrait">;

#endif // MLIR_INTERFACES_VIEWLIKEINTERFACE
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
- ViewLikeOpInterface builds on OpInterface<"ViewLikeOpInterface">
- OffsetSizeAndStrideOpInterface builds on OpInterface<"OffsetSizeAndStrideOpInterface">
- DistinctObjectsTrait builds on NativeOpTrait<"DistinctObjectsTrait">;
