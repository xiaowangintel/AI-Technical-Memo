# XeGPUTypes.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/XeGPU/IR/XeGPUTypes.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR XeGPUTypes component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 XeGPUTypes 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
````tablegen
//===- XeGPUTypes.td - XeGPU dialect types definition -------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_XEGPU_IR_XEGPUTYPES_TD
#define MLIR_DIALECT_XEGPU_IR_XEGPUTYPES_TD

include "mlir/Dialect/XeGPU/IR/XeGPUAttrs.td"
include "mlir/Dialect/XeGPU/IR/XeGPUDialect.td"
include "mlir/IR/BuiltinTypes.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 16-17
````tablegen
def XeGPU_IntType : AnyTypeOf<[I1, I<4>, I8, I16, I32, I64, SI1, SI8, SI16,
                               SI32, SI64, UI1, UI8, UI16, UI32, UI64]>;
````
- **EN**: This TableGen block defines `XeGPU_IntType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_IntType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 18-19
````tablegen
def XeGPU_FloatType : AnyTypeOf<[F4E2M1FN, F8E4M3FN, F8E5M2, F8E8M0FNU, F16,
                                 F32, F64, BF16, TF32]>;
````
- **EN**: This TableGen block defines `XeGPU_FloatType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_FloatType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 20-20
````tablegen
def XeGPU_ScalarType: AnyTypeOf<[XeGPU_IntType, XeGPU_FloatType]>;
````
- **EN**: This TableGen block defines `XeGPU_ScalarType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_ScalarType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 21-21
````tablegen
def XeGPU_PointerType : AnyTypeOf<[UI64, UI32, I64, I32]>;
````
- **EN**: This TableGen block defines `XeGPU_PointerType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_PointerType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 22-23
````tablegen
def XeGPU_BaseAddrType
    : AnyTypeOf<[Non0RankedMemRefOf<[XeGPU_ScalarType]>, XeGPU_PointerType]>;
````
- **EN**: This TableGen block defines `XeGPU_BaseAddrType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_BaseAddrType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 24-24
````tablegen
def XeGPU_DpasOprType: FixedVectorOfRankAndType<[1, 2, 3], [XeGPU_ScalarType]>;
````
- **EN**: This TableGen block defines `XeGPU_DpasOprType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_DpasOprType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 25-25
````tablegen
def XeGPU_DpasResType: FixedVectorOfRankAndType<[1, 2], [XeGPU_ScalarType]>;
````
- **EN**: This TableGen block defines `XeGPU_DpasResType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_DpasResType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 26-26
````tablegen
def XeGPU_OffsetType: FixedVectorOfNonZeroRankOf<[Index]>;
````
- **EN**: This TableGen block defines `XeGPU_OffsetType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_OffsetType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 27-27
````tablegen
def XeGPU_MaskType: FixedVectorOfNonZeroRankOf<[I1]>;
````
- **EN**: This TableGen block defines `XeGPU_MaskType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_MaskType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 28-28
````tablegen
def XeGPU_ValueType: VectorOfRankAndType<[1,2,3,4,5,6,7,8], [XeGPU_ScalarType]>;
````
- **EN**: This TableGen block defines `XeGPU_ValueType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_ValueType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 29-29
````tablegen
def XeGPU_ValueOrScalarType : AnyTypeOf<[XeGPU_ValueType, XeGPU_ScalarType]>;
````
- **EN**: This TableGen block defines `XeGPU_ValueOrScalarType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_ValueOrScalarType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 30-31
````tablegen
def XeGPU_VectorOrScalarType
    : AnyTypeOf<[VectorOfRankAndType<[1,2,3,4,5,6,7,8], [XeGPU_ScalarType, Index]>, XeGPU_ScalarType]>;
````
- **EN**: This TableGen block defines `XeGPU_VectorOrScalarType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_VectorOrScalarType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 32-35
````tablegen
def XeGPU_GatherScatterBaseAddrType
    : AnyTypeOf<[MemRefRankOf<[XeGPU_ScalarType], [1]>, XeGPU_PointerType]>;

// common base class for types in XeGPU dialect
````
- **EN**: This TableGen block defines `XeGPU_GatherScatterBaseAddrType` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `XeGPU_GatherScatterBaseAddrType` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 36-40
````tablegen
class XeGPUTypeDef<string name, string typeMnemonic, list<Trait> traits = [],
                   string baseCppClass = "::mlir::Type">
    : TypeDef<XeGPU_Dialect, name, traits, baseCppClass> {
  let mnemonic = typeMnemonic;
}
````
- **EN**: This TableGen block defines `XeGPUTypeDef` as a `class` record for `XeGPUTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `XeGPUTypeDef` 定义为 `class` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 42-42
````tablegen
def isSharedPred : CPred<"XeGPUDialect::isSharedMemory(llvm::cast<mlir::MemRefType>($_self))">;
````
- **EN**: This TableGen block defines `isSharedPred` as a `def` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `isSharedPred` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 43-46
````tablegen
class StaticShared1DMemRefOf<list<Type> allowedTypes> :
  ConfinedType<MemRefRankOf<allowedTypes, [1]>, [HasStaticShapePred, isSharedPred],
     "reside in share memory and statically 1d shaped " # MemRefOf<allowedTypes>.summary # " ",
     "mlir::MemRefType">;
````
- **EN**: This TableGen block defines `StaticShared1DMemRefOf` as a `class` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `StaticShared1DMemRefOf` 定义为 `class` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 48-51
````tablegen
class StaticShared2DMemRefOf<list<Type> allowedTypes>:
  ConfinedType<MemRefRankOf<allowedTypes, [2]>, [HasStaticShapePred, isSharedPred],
     "reside in share memory and statically 2d shaped " # MemRefOf<allowedTypes>.summary # " ",
     "mlir::MemRefType">;
````
- **EN**: This TableGen block defines `StaticShared2DMemRefOf` as a `class` record for `XeGPUTypes`.
- **CN**: 该 TableGen 代码块将 `StaticShared2DMemRefOf` 定义为 `class` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。

### Lines 53-126
````tablegen
def XeGPU_TensorDesc: XeGPUTypeDef<"TensorDesc", "tensor_desc",
        [ShapedTypeInterface], "::mlir::TensorType"> {
  let summary = "TensorDesc describing regions of interested data.";
  let description = [{
    TensorDesc is a type designed to describe regions of interest in data, as well as some features
    unique to Intel hardware. Unlike the built-in tensor type in MLIR, it essentially contains only
    metadata and does not hold the data itself. It is primarily designed to support 2D block load/store
    and DPAS (matrix multiplication instruction) on Intel GPUs. It encodes the following information:

    * shape:  the sizes/shape of the interested data block, e.g., 8x16 means 8 rows
              and each row contains 16 contiguous data elements.

    * element_type: the data type of the data element, e.g., f16, f32.

    Similar to the built-in tensor, it also provides optional attributes for encoding
    additional information via BlockTensorDescAttr, or supporting Workgroup & Subgroup
    level programmings via the Layout attribute. Please check their definition for details.

    Syntax:

    ```
    TensorDesc-type ::= `tensor_desc` `<` dim-list element-type (attr-list)? `>`
    element-type ::= float-type | integer-type
    dim-list := (static-dim-list `x`)?
    static-dim-list ::= decimal-literal `x` decimal-literal
    attr-list = (, encoding-attr)? (, layout-attr)?
    enconding-attr = (, memory_space = value)? (, arr_len = value)? (, boundary_check = value)?
    layout-attr = DistributeLayoutAttr
    ```

    Examples:

    ```mlir
    // A block TensorDesc with 8x16 i32 elements
    xegpu.tensor_desc<8x16xi32>

    // A block TensorDesc with 8x16 f32 elements
    xegpu.tensor_desc<8x16xf32>

    // A TensorDesc with 8x16 f32 elements for a memory region in shared memory space.
    xegpu.tensor_desc<8x16xf32, #xegpu.tdesc_attr<memory_space = slm>>

    // A 1D TensorDesc with a layout for subgroup level programming, each lane access two continuous elements
    xegpu.tensor_desc<32xf32, #xegpu.layout<lane_layout = [16], lane_data = [2]>>

    // A 1D TensorDesc with a layout for subgroup level programming, each lane access two elements with stride = 16
    xegpu.tensor_desc<32xf32, #xegpu.layout<lane_layout = [16], lane_data = [1]>>

    // A TensorDesc with a layout for subgroup level programming
    xegpu.tensor_desc<8x16xf32, #xegpu.layout<lane_layout = [1, 16], lane_data = [1, 1]>>

    // A TensorDesc with a layout for workgroup level programming
    xegpu.tensor_desc<32x64xf32, #xegpu.layout<sg_layout = [2, 4], sg_data = [16, 16], lane_layout = [1, 16], lane_data = [1, 1]>>

    // A TensorDesc with a layout for workgroup level programming without lane_layout and lane_data
    xegpu.tensor_desc<32x64xf32, #xegpu.layout<sg_layout = [2, 4], sg_data = [16, 16]>>

    ```
  }];

  let parameters = (ins ArrayRefParameter<"int64_t">: $shape,
                        "mlir::Type": $elementType,
                        OptionalParameter<"mlir::Attribute">: $encoding,
                        OptionalParameter<"mlir::Attribute">: $layout);

  let builders = [
    TypeBuilderWithInferredContext<(ins
      "llvm::ArrayRef<int64_t>": $shape,
      "mlir::Type": $elementType,
      CArg<"int", "1">: $array_length,
      CArg<"bool", "true">: $boundary_check,
      CArg<"xegpu::MemorySpace", "xegpu::MemorySpace::Global">:$memory_space,
      CArg<"mlir::Attribute", "mlir::Attribute()">:$layout)>
  ];
````
- **EN**: This TableGen block defines `XeGPU_TensorDesc` as a `def` record for `XeGPUTypes`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `XeGPU_TensorDesc` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 127-174
````tablegen
  let extraClassDeclaration = [{
    using mlir::ShapedType::Trait<TensorDescType>::getElementTypeBitWidth;
    using mlir::ShapedType::Trait<TensorDescType>::getRank;
    using mlir::ShapedType::Trait<TensorDescType>::getNumElements;
    using mlir::ShapedType::Trait<TensorDescType>::isDynamicDim;
    using mlir::ShapedType::Trait<TensorDescType>::hasStaticShape;
    using mlir::ShapedType::Trait<TensorDescType>::getNumDynamicDims;
    using mlir::ShapedType::Trait<TensorDescType>::getDimSize;
    using mlir::ShapedType::Trait<TensorDescType>::getDynamicDimIndex;

    TensorDescType clone(::mlir::Type elementType) {
      return llvm::cast<TensorDescType>(cloneWith(getShape(), elementType));
    }

    BlockTensorDescAttr getBlockAttr() const {
      return llvm::dyn_cast_if_present<BlockTensorDescAttr>(getEncoding());
    }

    DistributeLayoutAttr getLayoutAttr() const {
      return llvm::dyn_cast_if_present<DistributeLayoutAttr>(getLayout());
    }

    xegpu::MemorySpace getMemorySpace() const {
      return getBlockAttr().getMemorySpace().getValue();
    }

    // get the ArrayLength for blocked TensorDesc
    int getArrayLength() {
      return getBlockAttr().getArrayLength().getInt();
    }

    bool getBoundaryCheck() {
      return getBlockAttr().getBoundaryCheck().getValue();
    }

    /// Helper to drop all layout information from the TensorDesc type.
    TensorDescType dropLayouts() {
      if (!getLayoutAttr())
        return *this;

      return get(getContext(), getShape(), getElementType(), getEncoding(),
        xegpu::LayoutAttr());
    }
  }];

  let hasCustomAssemblyFormat = true;
  let genVerifyDecl = 1;
}
````
- **EN**: This block groups callable interfaces such as `clone`, `cloneWith`, `getShape`, `getBlockAttr`, indicating how `XeGPUTypes` is queried or updated.
- **CN**: 该代码块聚合了 `clone`, `cloneWith`, `getShape`, `getBlockAttr` 等可调用接口，展示了如何查询或更新 `XeGPUTypes`。

### Lines 177-185
````tablegen
def XeGPU_Nbarrier: XeGPUTypeDef<"Nbarrier", "nbarrier", [], "mlir::Type"> {
  let summary = "!xegpu.nbarrier a custom XeGPU type representing a barrier.";

  let extraClassDeclaration = [{
    static NbarrierType get(mlir::MLIRContext *context) {
      return Base::get(context);
    };
  }];
}
````
- **EN**: This TableGen block defines `XeGPU_Nbarrier` as a `def` record for `XeGPUTypes`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `XeGPU_Nbarrier` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。 其中涉及 语义文档。

### Lines 187-260
````tablegen
def XeGPU_MemDesc: XeGPUTypeDef<"MemDesc", "mem_desc", [ShapedTypeInterface], "mlir::Type"> {
  let summary = "MemDesc describing the data in SLM";
  let description = [{
    MemDesc represents a block of data stored in shared local memory.
    By default, unless a layout attribute is provided, the data is stored
    contiguously in row-major order within the region.

    Examples:
    ```mlir
    // A multi-dimensional array stored in column-major order.
    !xegpu.mem_desc<128x128xf16, #xegpu.mem_layout<stride = [1, 128]>>

    // A multi-dimensional array stored in a blocked layout. Elements within the same block
    // are stored contiguously in memory. Blocks are stored in row-major order.
    !xegpu.mem_desc<128x128xf16, #xegpu.mem_layout<block = [8, 8]>>

    // A multi-dimensional array stored in column-major order with blocked layout.
    !xegpu.mem_desc<128x128xf16, #xegpu.mem_layout<stride = [1, 128], block = [8, 8]>>
    ```
  }];
  let parameters = (ins ArrayRefParameter<"int64_t">: $shape,
                        "mlir::Type": $elementType,
                        OptionalParameter<"MemLayoutAttr">: $mem_layout);

  let extraClassDeclaration = [{
    bool hasRank() const { return true; }

    MemDescType cloneWith(std::optional<llvm::ArrayRef<int64_t>> shape, Type elementType) const {
      return MemDescType::get(getContext(), shape.value_or(getShape()), elementType, getMemLayout());
    }

    ArrayAttr getStrideAttr() {
      auto layout = getMemLayout();
      if (layout && layout.hasAttr("stride")) {
        return layout.getStrideAttr();
      }
      // derive and return default strides
      SmallVector<int64_t> defaultStrides;
      llvm::append_range(defaultStrides, getShape().drop_front());
      llvm::append_values(defaultStrides, 1);
      Builder builder(getContext());
      return builder.getI64ArrayAttr(defaultStrides);
    }

    ArrayAttr getBlockAttr() {
      auto layout = getMemLayout();
      if (layout && layout.hasAttr("block")) {
        return layout.getBlockAttr();
      }
      Builder builder(getContext());
      return builder.getI64ArrayAttr({});
    }

    /// Heuristic to determine if the MemDesc uses column-major layout,
    /// based on the rank and the value of the first stride dimension.
    bool isColMajor() {
      auto dim0 = dyn_cast<IntegerAttr>(getStrideAttr()[0]);
      return getRank() == 2 && dim0.getInt() == 1;
    }

    // Get the Blocking shape for a MemDescType, Which is represented
    // as an attribute in MemDescType. By default it is the shape
    // of the mdescTy
    SmallVector<int64_t> getBlockShape() {
      SmallVector<int64_t> size(getShape());
      ArrayAttr blockAttr = getBlockAttr();
      if (!blockAttr.empty()) {
        size.clear();
        for (auto attr : blockAttr.getValue()) {
          size.push_back(cast<IntegerAttr>(attr).getInt());
        }
      }
      return size;
    }
````
- **EN**: This TableGen block defines `XeGPU_MemDesc` as a `def` record for `XeGPUTypes`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `XeGPU_MemDesc` 定义为 `def` 记录，用于描述 `XeGPUTypes` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 261-289
````tablegen
    // Get strides as vector of integer.
    // If it contains block attribute, the strides are blocked strides.
    //
    // The blocking is applied to the base matrix shape derived from the
    // memory descriptor's stride information. If the matrix described by
    // the memory descriptor is not contiguous, it is assumed that the base
    // matrix is contiguous and follows the same memory layout.
    //
    // It first computes the original matrix shape using the stride info,
    // then computes the number of blocks in each dimension of original shape,
    // then compute the outer block shape and stride,
    // then combines the inner and outer block shape and stride
    // e.g. for `mem_desc<32x256xf16, @block=[16, 8], @strides=[1, 32]>`
    // its memory layout tuple is ([2,32,16,8],[128,256,1,16])
    // for  `mem_desc<256x32xf16, @block=[8, 16]>` with default @stride[32, 1]
    // its memory layout tuple is ([32,2,8,16],[256,128,16,1])
    SmallVector<int64_t> getStrideShape();

    /// Generates instructions to compute the linearize offset
    //  if the memory descriptor is blocked, it returns linearize offset based on the blocked layout
    //  the strides of memory descriptor is always considered regardless of blocked or not
    Value getLinearOffsets(OpBuilder &builder,
               Location loc, ArrayRef<OpFoldResult> offsets);


  }];

  let hasCustomAssemblyFormat = true;
}
````
- **EN**: This block groups callable interfaces such as `getStrideShape`, `getLinearOffsets`, indicating how `XeGPUTypes` is queried or updated.
- **CN**: 该代码块聚合了 `getStrideShape`, `getLinearOffsets` 等可调用接口，展示了如何查询或更新 `XeGPUTypes`。

### Lines 291-291
````tablegen
#endif // MLIR_DIALECT_XEGPU_IR_XEGPUTYPES_TD
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Custom assembly syntax specification
  **CN**: 自定义汇编语法规格
- **EN**: Trait and interface composition
  **CN**: trait 与接口组合
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/Dialect/XeGPU/IR/XeGPUAttrs.td
- mlir/Dialect/XeGPU/IR/XeGPUDialect.td
- mlir/IR/BuiltinTypes.td
- XeGPU_IntType builds on AnyTypeOf<[I1, I<4>, I8, I16, I32, I64, SI1, SI8, SI16,
- XeGPU_FloatType builds on AnyTypeOf<[F4E2M1FN, F8E4M3FN, F8E5M2, F8E8M0FNU, F16,
- XeGPU_ScalarType builds on AnyTypeOf<[XeGPU_IntType, XeGPU_FloatType]>;
- XeGPU_PointerType builds on AnyTypeOf<[UI64, UI32, I64, I32]>;
- XeGPU_BaseAddrType builds on AnyTypeOf<[Non0RankedMemRefOf<[XeGPU_ScalarType]>, XeGPU_PointerType]>;
- XeGPU_DpasOprType builds on FixedVectorOfRankAndType<[1, 2, 3], [XeGPU_ScalarType]>;
- XeGPU_DpasResType builds on FixedVectorOfRankAndType<[1, 2], [XeGPU_ScalarType]>;
- XeGPU_OffsetType builds on FixedVectorOfNonZeroRankOf<[Index]>;
- XeGPU_MaskType builds on FixedVectorOfNonZeroRankOf<[I1]>;
- XeGPU_ValueType builds on VectorOfRankAndType<[1,2,3,4,5,6,7,8], [XeGPU_ScalarType]>;
- XeGPU_ValueOrScalarType builds on AnyTypeOf<[XeGPU_ValueType, XeGPU_ScalarType]>;
- XeGPU_VectorOrScalarType builds on AnyTypeOf<[VectorOfRankAndType<[1,2,3,4,5,6,7,8], [XeGPU_ScalarType, Index]>, XeGPU_ScalarType]>;
- XeGPU_GatherScatterBaseAddrType builds on AnyTypeOf<[MemRefRankOf<[XeGPU_ScalarType], [1]>, XeGPU_PointerType]>;
- isSharedPred builds on CPred<"XeGPUDialect::isSharedMemory(llvm::cast<mlir::MemRefType>($_self))">;
- XeGPU_TensorDesc builds on XeGPUTypeDef<"TensorDesc", "tensor_desc",
- XeGPU_Nbarrier builds on XeGPUTypeDef<"Nbarrier", "nbarrier", [], "mlir::Type">
- XeGPU_MemDesc builds on XeGPUTypeDef<"MemDesc", "mem_desc", [ShapedTypeInterface], "mlir::Type">
