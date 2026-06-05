# InferTypeOpInterface.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Interfaces/InferTypeOpInterface.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR InferTypeOpInterface component. The leading comments describe it as: This file contains the definitions of the infer op interfaces defined in.
- **用途（CN）**: 声明 MLIR InferTypeOpInterface 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
````cpp
//===- InferTypeOpInterface.h - Infer Type Interfaces -----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the definitions of the infer op interfaces defined in
// `InferTypeOpInterface.td`.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_INTERFACES_INFERTYPEOPINTERFACE_H_
#define MLIR_INTERFACES_INFERTYPEOPINTERFACE_H_

#include "mlir/IR/Attributes.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Location.h"
#include "mlir/IR/OpDefinition.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/PointerUnion.h"
#include "llvm/ADT/SmallVector.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 26-72
````cpp
namespace mlir {

class ShapedTypeComponents;
using ReifiedRankedShapedTypeDims = SmallVector<SmallVector<OpFoldResult>>;

/// Reify the shape of the result of an operation (typically in terms of the
/// shape of its operands).
LogicalResult
reifyResultShapes(OpBuilder &b, Operation *op,
                  ReifiedRankedShapedTypeDims &reifiedReturnShapes);
FailureOr<SmallVector<OpFoldResult>>
reifyShapeOfResult(OpBuilder &b, Operation *op, int resultIndex);
FailureOr<OpFoldResult> reifyDimOfResult(OpBuilder &b, Operation *op,
                                         int resultIndex, int dim);

/// Adaptor class to abstract the differences between whether value is from
/// a ShapedType or ShapedTypeComponents or DenseIntElementsAttribute.
class ShapeAdaptor {
public:
  ShapeAdaptor(Type t) {
    if (auto st = dyn_cast<ShapedType>(t))
      val = st;
  }
  ShapeAdaptor(Attribute t) {
    if (auto da = dyn_cast<DenseIntElementsAttr>(t))
      val = da;
  }
  ShapeAdaptor(ShapedTypeComponents *components) : val(components) {}
  ShapeAdaptor(ShapedTypeComponents &components) : val(&components) {}

  /// Returns whether the shape has a rank.
  bool hasRank() const;

  /// Returns the element type.
  Type getElementType() const;

  /// Populates the dimensions from shape referenced.
  /// Requires: shape is ranked.
  void getDims(SmallVectorImpl<int64_t> &res) const;

  /// Populates the dimensions of the ShapeTypeComponents.
  /// Requires: shape is ranked.
  void getDims(ShapedTypeComponents &res) const;

  /// Returns the size of the index'th dimension.
  /// Requires: shape is ranked.
  int64_t getDimSize(int index) const;
````
- **EN**: This C++ declaration introduces `ShapedTypeComponents` and establishes part of the API surface for `InferTypeOpInterface`. Representative entry points here include `reifyResultShapes`, `reifyShapeOfResult`, `reifyDimOfResult`, `ShapeAdaptor`.
- **CN**: 该 C++ 声明引入了 `ShapedTypeComponents`，并构成 `InferTypeOpInterface` API 表面的一部分。 这一段可见的代表性接口包括 `reifyResultShapes`, `reifyShapeOfResult`, `reifyDimOfResult`, `ShapeAdaptor`。

### Lines 73-140
````cpp
  /// Returns whether the index'th dimension is dynamic.
  /// Requires: shape is ranked.
  bool isDynamicDim(int index) const {
    return ShapedType::isDynamic(getDimSize(index));
  }

  /// Returns whether the shape is fully static.
  bool hasStaticShape() const;

  /// Returns the rank of the shape.
  /// Requires: shape is ranked.
  int64_t getRank() const;

  /// Returns the number of elements in the shape.
  /// Requires: hasStaticShape
  int64_t getNumElements() const;

  /// Returns whether valid (non-null) shape.
  explicit operator bool() const { return !val.isNull(); }

  /// Dumps textual repesentation to stderr.
  void dump() const;

private:
  // Union storing either ShapedTypeComponents, ShapedType (stored as Type and
  // casted), or DenseIntElementsAttribute (stored as Atrtribute).
  PointerUnion<ShapedTypeComponents *, Type, Attribute> val = nullptr;
};

/// ShapedTypeComponents that represents the components of a ShapedType.
/// The components consist of
///  - A ranked or unranked shape with the dimension specification match those
///    of ShapeType's getShape() (e.g., dynamic dimension represented using
///    ShapedType::kDynamic)
///  - A element type, may be unset (nullptr)
///  - A attribute, may be unset (nullptr)
/// Used by ShapedType type inferences.
class ShapedTypeComponents {
  /// Internal storage type for shape.
  using ShapeStorageT = SmallVector<int64_t, 3>;

public:
  /// Default construction is an unranked shape.
  ShapedTypeComponents() : elementType(nullptr), attr(nullptr) {};
  ShapedTypeComponents(Type elementType)
      : elementType(elementType), attr(nullptr), ranked(false) {}
  ShapedTypeComponents(ShapedType shapedType) : attr(nullptr) {
    ranked = shapedType.hasRank();
    elementType = shapedType.getElementType();
    if (ranked)
      dims = llvm::to_vector<4>(shapedType.getShape());
  }
  ShapedTypeComponents(ShapeAdaptor adaptor) : attr(nullptr) {
    ranked = adaptor.hasRank();
    elementType = adaptor.getElementType();
    if (ranked)
      adaptor.getDims(*this);
  }
  template <typename Arg, typename = std::enable_if_t<
                              std::is_constructible<ShapeStorageT, Arg>::value>>
  ShapedTypeComponents(Arg &&arg, Type elementType = nullptr,
                       Attribute attr = nullptr)
      : dims(std::forward<Arg>(arg)), elementType(elementType), attr(attr),
        ranked(true) {}
  ShapedTypeComponents(ArrayRef<int64_t> vec, Type elementType = nullptr,
                       Attribute attr = nullptr)
      : dims(vec.begin(), vec.end()), elementType(elementType), attr(attr),
        ranked(true) {}
````
- **EN**: This C++ declaration introduces `ShapedTypeComponents` and establishes part of the API surface for `InferTypeOpInterface`. Representative entry points here include `isDynamicDim`, `isDynamic`, `getDimSize`, `hasStaticShape`.
- **CN**: 该 C++ 声明引入了 `ShapedTypeComponents`，并构成 `InferTypeOpInterface` API 表面的一部分。 这一段可见的代表性接口包括 `isDynamicDim`, `isDynamic`, `getDimSize`, `hasStaticShape`。

### Lines 142-190
````cpp
  /// Return the dimensions of the shape.
  /// Requires: shape is ranked.
  ArrayRef<int64_t> getDims() const {
    assert(ranked && "requires ranked shape");
    return dims;
  }

  /// Return whether the shape has a rank.
  bool hasRank() const { return ranked; };

  /// Return the element type component.
  Type getElementType() const { return elementType; };

  /// Return the raw attribute component.
  Attribute getAttribute() const { return attr; };

private:
  friend class ShapeAdaptor;

  ShapeStorageT dims;
  Type elementType;
  Attribute attr;
  bool ranked{false};
};

/// Range of values and shapes (corresponding effectively to Shapes dialect's
/// ValueShape type concept).
// Currently this exposes the Value (of operands) and Type of the Value. This is
// not ideal as then one can accidentally reference an out of date shape. This
// is done to both enable gradual switch and also as OpAdaptor doesn't currently
// allow returning anything other than Value.
class ValueShapeRange : public ValueRange::RangeBaseT {
public:
  using ValueShapeMapFn = function_ref<ShapeAdaptor(Value)>;

  ValueShapeRange(ValueRange values, ValueShapeMapFn operandShape = nullptr,
                  ValueShapeMapFn valueToShape = nullptr)
      : RangeBaseT(values), operandShape(operandShape),
        valueToShape(valueToShape) {}
  ValueShapeRange(const std::initializer_list<Value> &values)
      : ValueShapeRange(ValueRange(values)) {}

  ValueShapeRange(const ValueShapeRange &) = default;

  /// Sets the Value to ShapeAdaptor mapping function and returns this.
  ValueShapeRange &setValueToShapeMapping(ValueShapeMapFn fn) {
    valueToShape = fn;
    return *this;
  }
````
- **EN**: This C++ declaration introduces `ValueShapeRange` and establishes part of the API surface for `InferTypeOpInterface`. Representative entry points here include `getDims`, `hasRank`, `getElementType`, `getAttribute`.
- **CN**: 该 C++ 声明引入了 `ValueShapeRange`，并构成 `InferTypeOpInterface` API 表面的一部分。 这一段可见的代表性接口包括 `getDims`, `hasRank`, `getElementType`, `getAttribute`。

### Lines 192-238
````cpp
  ValueShapeRange &setOperandShapeMapping(ValueShapeMapFn fn) {
    operandShape = fn;
    return *this;
  }

  /// Returns the set Value to ShapeAdaptor mapping function.
  ValueShapeMapFn getValueToShapeMapping() const { return valueToShape; }
  ValueShapeMapFn getOperandShapeMapping() const { return operandShape; }

  // Accessors.

  /// Returns the types of the values within this range.
  /// Note: This returns only the types of Values in the ValueRange and not a
  /// more refined type.
  using type_iterator = ValueTypeIterator<iterator>;
  using type_range = ValueTypeRange<ValueRange>;
  type_range getTypes() const { return {begin(), end()}; }
  auto getType() const { return getTypes(); }

  /// Returns the Values in the ValueRange.
  /// To query the most up to date shape of a Value, query the shape
  /// using getShape below rather than using the type of the Value.
  ValueRange getValues() const { return ValueRange(begin(), end()); };

  /// Returns an argument as shape. If the argument is not constant or not a
  /// shape, then the function returns a nullptr.
  /// This will first query the valueToShape mapping (if set), before querying
  /// the ValueRange.
  ShapeAdaptor getValueAsShape(int index);

  /// Returns the shape of index'th operand.
  // TODO: Update so that operator[] references these instead to avoid
  // accidentally refering to less refined shape.
  ShapeAdaptor getShape(int index) const;

  /// Returns the shape of the given Value.
  ShapeAdaptor getShape(Value val) const;

private:
  // Mapping from Value to ShapedTypeComponents corresponding to shape of type
  // of Value.
  ValueShapeMapFn operandShape;

  // Mapping from Value to ShapedTypeComponents corresponding to constant Value
  // if interpreted as shape.
  ValueShapeMapFn valueToShape;
};
````
- **EN**: This block groups callable interfaces such as `setOperandShapeMapping`, `getValueToShapeMapping`, `getOperandShapeMapping`, `getTypes`, indicating how `InferTypeOpInterface` is queried or updated.
- **CN**: 该代码块聚合了 `setOperandShapeMapping`, `getValueToShapeMapping`, `getOperandShapeMapping`, `getTypes` 等可调用接口，展示了如何查询或更新 `InferTypeOpInterface`。

### Lines 240-263
````cpp
namespace detail {
// Helper function to infer return tensor returns types given element and
// shape inference function.
LogicalResult
inferReturnTensorTypes(ArrayRef<ShapedTypeComponents> retComponents,
                       SmallVectorImpl<Type> &inferredReturnTypes);

/// Verifies that the inferred result types match the actual result types for
/// the op. Precondition: op implements InferTypeOpInterface.
LogicalResult verifyInferredResultTypes(Operation *op);

/// Report a fatal error indicating that the result types could not be
/// inferred.
void reportFatalInferReturnTypesError(OperationState &state);
} // namespace detail

namespace OpTrait {
template <typename ConcreteType>
class InferTensorType;
} // namespace OpTrait
} // namespace mlir

/// Include the generated interface declarations.
#include "mlir/Interfaces/InferTypeOpInterface.h.inc"
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 266-288
````cpp
namespace mlir {
namespace OpTrait {

template <typename ConcreteType>
class InferTypeOpAdaptor : public TraitBase<ConcreteType, InferTypeOpAdaptor> {
};

template <typename ConcreteType>
class InferShapedTypeOpAdaptor
    : public TraitBase<ConcreteType, InferShapedTypeOpAdaptor> {};

/// Tensor type inference trait that constructs a tensor from the inferred
/// shape and elemental types.
/// Requires: Op implements InferShapedTypeOpInterface and InferTypeOpInterface.
///   Less strict is possible (e.g., implements inferReturnTypeComponents and
///   these always populates all element types and shapes or fails, but this
///   trait is currently only used where the interfaces are, so keep it
///   restricted for now).
template <typename ConcreteType>
class InferTensorType : public TraitBase<ConcreteType, InferTensorType> {};

} // namespace OpTrait
} // namespace mlir
````
- **EN**: This C++ declaration introduces `InferTypeOpAdaptor` and establishes part of the API surface for `InferTypeOpInterface`.
- **CN**: 该 C++ 声明引入了 `InferTypeOpAdaptor`，并构成 `InferTypeOpInterface` API 表面的一部分。

### Lines 290-290
````cpp
#endif // MLIR_INTERFACES_INFERTYPEOPINTERFACE_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Iterator-based traversal APIs
  **CN**: 基于迭代器的遍历接口
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/Attributes.h
- mlir/IR/Builders.h
- mlir/IR/BuiltinTypes.h
- mlir/IR/Location.h
- mlir/IR/OpDefinition.h
- mlir/Support/LLVM.h
- llvm/ADT/PointerUnion.h
- llvm/ADT/SmallVector.h
- mlir/Interfaces/InferTypeOpInterface.h.inc
- ValueShapeRange inherits from public ValueRange::RangeBaseT
- InferTypeOpAdaptor inherits from public TraitBase<ConcreteType, InferTypeOpAdaptor>
- InferShapedTypeOpAdaptor inherits from public TraitBase<ConcreteType, InferShapedTypeOpAdaptor>
- InferTensorType inherits from public TraitBase<ConcreteType, InferTensorType>
- ValueShapeRange builds on public ValueRange::RangeBaseT
- InferTypeOpAdaptor builds on public TraitBase<ConcreteType, InferTypeOpAdaptor>
- InferShapedTypeOpAdaptor builds on public TraitBase<ConcreteType, InferShapedTypeOpAdaptor>
- InferTensorType builds on public TraitBase<ConcreteType, InferTensorType>
